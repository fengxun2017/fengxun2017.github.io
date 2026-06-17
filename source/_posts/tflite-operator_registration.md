---
title: TFLite Micro 深度剖析（二）：算子注册——从类型编号到计算函数
date: 2026/5/22
categories: 
- tflite
---

<center>
本文是 TFLite Micro 内部实现深度剖析系列的第 2 篇。主要介绍 TFLite Micro 的模型算子注册相关内容
</center>
<!--more-->

***

**示例模型的完整参考数据见**[第1篇前置信息](https://fengxun2017.github.io/2026/05/18/tfite-load-model)。

</br>

### 1.1 为什么需要算子注册？

`.tflite` 文件里存的是模型的结构和权重数据（FlatBuffer 编码）。模型中的每个算子（Conv2D、MaxPool、Relu 等）只记录了**类型编号**和**参数**（如步长、padding），但没有记录**怎么计算**。

这就像一份建筑图纸——图纸标注了"这里要装一扇门"（类型=门，参数=宽80cm、高2m），但**怎么做门**、**用什么材料**，图纸不管。施工队需要自己准备"门的安装方法"。

TFLM 也是一样。`.tflite` 文件说"第 0 个算子是 Conv2D，步长 1x1，padding SAME"，但 Conv2D 的实际计算代码不在文件里——**需要你在代码中手动注册**。

```
.tflite 文件只存:               代码中需要注册:
┌─────────────────┐            ┌──────────────────────────────┐
│ operator_codes: │            │ resolver.AddConv2D(...)      │
│   [0] CONV_2D   │  --匹配--> │   → EvalInt8() 计算函数       │
│   [1] MAX_POOL  │            │                              │
│   ...           │            │ resolver.AddMaxPool2D(...)   │
│                 │            │   → EvalInt8() 计算函数       │
│ operators[0]:   │            │                              │
│   opcode=0      │            │ 16 个实例，7 种类型，每种需注册│
│   inputs=[0,1,2]│            └──────────────────────────────┘
└─────────────────┘
```

**核心问题**：如果代码没注册某个算子（比如模型用了 Quantize 但你忘了注册），运行时 `AllocateTensors()` 会报错——解释器找到 operator_codes 里的类型编号，但在注册表里找不到对应的计算函数。

**为什么不在文件里自带计算代码？** 因为同一个算子在不同硬件上有不同实现：

- **Reference 实现**：纯 C，任何平台都能跑，但慢
- **CMSIS-NN 实现**：用 Cortex-M 的 DSP/NEON 指令加速
- **可能还有**：Xtensa、GPU 等实现

注册机制让你选择**用哪个实现**：

```cpp
// CMSIS-NN 优化版
resolver.AddConv2D(tflite::Register_CONV_2D_INT8());

// Reference 版（不传参数）
resolver.AddConv2D(); 
```

两种写法都合法、编译通过、推理结果一致，但性能有很大差距。

### 1.2 TFLMRegistration——算子的"身份证"

理解了"为什么注册"，接下来看"注册什么"。

每个算子的注册信息用一个 `TFLMRegistration` 结构体表示，可以理解为一个"算子身份证"：

```cpp
// tflite/micro/micro_common.h
struct TFLMRegistration {
    void*        (*init)(TfLiteContext*, const char*, size_t);   // ① 初始化
    void         (*free)(TfLiteContext*, void*);                 // ② 释放
    TfLiteStatus (*prepare)(TfLiteContext*, TfLiteNode*);        // ③ 准备
    TfLiteStatus (*invoke)(TfLiteContext*, TfLiteNode*);         // ④ 执行
    void         (*reset)(TfLiteContext*, void*);                // ⑤ 重置
    int32_t      builtin_code;      // 算子类型编码
    const char*  custom_name;       // 自定义算子名
};
```

核心是 5 个**回调函数**，它们在推理的不同阶段被调用：

```
推理生命周期:

  AllocateTensors()                     Invoke()
   ──────────────┐                  ┌──────────────┐
                 │                  │              │
  ┌──────────────▼──────────────────▼───────────┐  │
  │ init()  →  prepare()  →  invoke() → 输出结果 │  │
  │  (1次)      (1次)       (每次推理)           │──┘
  └─────────────────────────────────────────────┘
```

| 回调 | 调用时机 | 做什么 | 能分配内存？ |
|------|----------|--------|-------------|
| `init` | AllocateTensors 期间，每个算子一次 | 分配 OpData 结构体（保存量化参数等） | 可以（持久内存） |
| `prepare` | AllocateTensors 期间，每个算子一次 | 验证形状、计算量化参数、请求 scratch buffer | 可以（持久+scratch） |
| `invoke` | 每次 Invoke() | 执行实际计算（这就是"算子"的核心） | 禁止 |
| `free` | 析构时 | 释放资源 | - |
| `reset` | Reset() 时 | 重置内部状态 | - |

以 Conv2D INT8 (CMSIS-NN) 为例，注册时填入的回调：

```
TFLMRegistration for Conv2D INT8 (CMSIS-NN):
  .init       = Init()          → 分配 OpData（卷积参数、scratch buffer 大小）
  .free       = nullptr         → 不需要
  .prepare    = Prepare()       → 验证输入形状、计算 im2col 参数、请求 scratch buffer
  .invoke     = EvalInt8()      → 调用 arm_convolve_wrapper_int8_s8()（CMSIS-NN）
  .reset      = nullptr
  .builtin_code = 0             → 将被 AddBuiltin 覆写为 CONV_2D=3
```

关键区别在 `invoke`：CMSIS-NN 版的 `invoke=EvalInt8()` 调用底层函数，内部通过 DSP intrinsics（SMLAD 等）使用硬件加速；Reference 版的 `invoke=Eval()` 内部通过 switch 分发到纯 C 循环实现。

### 1.3 MicroMutableOpResolver——注册容器

知道了"注册什么"（TFLMRegistration），再看"存在哪里"。

`MicroMutableOpResolver` 是注册容器，内部用**编译期确定大小的固定数组**存储所有注册信息：

```cpp
// tflite/micro/micro_mutable_op_resolver.h
template <unsigned int tOpCount>
class MicroMutableOpResolver : public MicroOpResolver {
 private:
  // 三组固定数组，编译期确定大小，零动态分配
  TFLMRegistration registrations_[tOpCount];           // 算子实现（回调函数表）
  unsigned int registrations_len_ = 0;                  // 当前注册数量

  BuiltinOperator builtin_codes_[tOpCount];             // 算子类型编码
  TfLiteBridgeBuiltinParseFunction builtin_parsers_[tOpCount]; // 参数解析函数
  unsigned int num_buitin_ops_ = 0;                     // 内置算子计数
};
```

**为什么用模板？** 嵌入式上不能 `malloc`。`tOpCount=20` 在编译期确定数组大小，直接分配在栈或静态区，零堆内存开销。

三组数组各有用途：

```
registrations_[20]     → 存算子的计算函数（init/prepare/invoke）
                          推理时，解释器遍历模型中的 operator，
                          用 opcode_index 查找对应的 registration，调用其 invoke

builtin_codes_[20]     → 存算子类型编码（CONV_2D=3, MAX_POOL_2D=17, ...）
                          用于去重检查和查找

builtin_parsers_[20]   → 存参数解析函数（ParseConv2D, ParseMaxPool, ...）
                          AllocateTensors 时，将 FlatBuffer 中的算子参数
                          转成 C 结构体（如 TfLiteConvParams）
```

**查找机制**——线性查找：

```cpp
// micro_mutable_op_resolver.h
const TFLMRegistration* FindOp(tflite::BuiltinOperator op) const override {
    if (op == BuiltinOperator_CUSTOM) return nullptr;

    for (unsigned int i = 0; i < registrations_len_; ++i) {
        if (registrations_[i].builtin_code == op) {
            return &registrations_[i];
        }
    }
    return nullptr;  // 未注册 → AllocateTensors() 报错
}
```

为什么不用哈希表？算子数量少（< 20），线性查找足够快，且无额外内存开销。

### 1.4 AddConv2D() 的完整链路

以 `AddConv2D` 为例，跟踪一次完整的注册过程。整个过程分两步：**生成身份证** → **存入数组**。

```cpp

static tflite::MicroMutableOpResolver<20> resolver;
status = resolver.AddConv2D(tflite::Register_CONV_2D_INT8());


// micro_mutable_op_resolver.h
TfLiteStatus AddConv2D(const TFLMRegistration& registration = Register_CONV_2D()) {
    return AddBuiltin(BuiltinOperator_CONV_2D, registration, ParseConv2D);
}


```

#### 第一步：Register_CONV_2D_INT8() 生成 TFLMRegistration

```cpp
// kernels/cmsis_nn/conv.cc
TFLMRegistration Register_CONV_2D_INT8() {
    return tflite::micro::RegisterOp(Init, Prepare, EvalInt8);
}
```

`RegisterOp()` 定义在 `kernels/kernel_util.cc`：

```cpp
TFLMRegistration RegisterOp(
    void* (*init)(TfLiteContext* context, const char* buffer, size_t length),
    TfLiteStatus (*prepare)(TfLiteContext* context, TfLiteNode* node),
    TfLiteStatus (*invoke)(TfLiteContext* context, TfLiteNode* node),
    void (*free)(TfLiteContext* context, void* buffer) = nullptr,
    void (*reset)(TfLiteContext* context, void* buffer) = nullptr) {
  return {/*init=*/init,
          /*free=*/free,
          /*prepare=*/prepare,
          /*invoke=*/invoke,
          /*reset=*/reset,
          /*builtin_code=*/0,        // 由 AddBuiltin 覆写
          /*custom_name=*/nullptr};
}
```

#### 第二步：AddBuiltin() 存入数组

```cpp
// micro_mutable_op_resolver.h
TfLiteStatus AddBuiltin(tflite::BuiltinOperator op,
                        const TFLMRegistration& registration,
                        TfLiteBridgeBuiltinParseFunction parser) {
    // ① 检查：不允许 CUSTOM 类型走这个路径
    if (op == BuiltinOperator_CUSTOM) {
        return kTfLiteError;
    }

    // ② 检查：同一个算子不能重复注册
    if (FindOp(op) != nullptr) {
        return kTfLiteError;
    }

    // ③ 检查：数组是否已满
    if (registrations_len_ >= tOpCount) {
        return kTfLiteError;
    }

    // ④ 存入算子实现
    registrations_[registrations_len_] = registration;
    registrations_[registrations_len_].builtin_code = op;  // 覆写 builtin_code
    registrations_len_++;

    // ⑤ 存入解析函数
    builtin_codes_[num_buitin_ops_] = op;
    builtin_parsers_[num_buitin_ops_] = parser;
    num_buitin_ops_++;

    return kTfLiteOk;
}
```

三重检查（非 CUSTOM、不重复、不溢出）后，分别存入 `registrations_` 和 `builtin_parsers_` 两个数组。

#### 参数解析函数：ParseConv2D

每个 AddBuiltin 除了存入 `TFLMRegistration`，还存入一个 `parser` 函数。它的作用是将 FlatBuffer 中的算子参数转成 C 结构体：

```
.tflite 文件中:
  Operator {
    opcode_index: 0           → operator_codes[0] = CONV_2D
    builtin_options: Conv2DOptions {
      stride_h: 1
      stride_w: 1
      padding: SAME
      activation: RELU
    }
  }
```

`ParseConv2D` 把上面的 FlatBuffer `Conv2DOptions` 转成 C 结构体 `TfLiteConvParams`：

```cpp
struct TfLiteConvParams {
    TfLitePadding padding;
    int stride_width;
    int stride_height;
    TfLiteFusedActivation activation;
    // ... dilation 等
};
```

**注册完成后的内存布局**（示例模型注册了 7 种算子）：

```
MicroMutableOpResolver<20> 内部:

registrations_[0]:  CONV_2D        → Init/Prepare/EvalInt8 (CMSIS-NN)  ← 5 个实例共用
registrations_[1]:  MAX_POOL_2D    → Init/Prepare/EvalInt8 (CMSIS-NN)  ← 3 个实例共用
registrations_[2]:  STRIDED_SLICE  → Init/Prepare/Eval                 ← 2 个实例共用
registrations_[3]:  LOGISTIC       → Init/Prepare/Eval                 ← 2 个实例共用
registrations_[4]:  QUANTIZE       → Init/Prepare/Eval                 ← 2 个实例共用
registrations_[5]:  CONCATENATION  → Init/Prepare/Eval                 ← 1 个实例
registrations_[6]:  DEQUANTIZE     → Init/Prepare/Eval                 ← 1 个实例
registrations_[7-19]: 空

builtin_codes_[0-6] = {3, 17, 45, 14, 114, 2, 6}  对应 BuiltinOperator 枚举值
builtin_parsers_[0-6] = {ParseConv2D, ParseMaxPool, ParseStridedSlice, ...}
```

前 2 个注册了 CMSIS-NN INT8 优化版（Conv2D、MaxPool），后 5 个用 Reference 版（Logistic、Quantize 等轻量算子）。

### 1.5 CMSIS-NN 硬件加速丢失的陷阱

注册算子时有一个容易踩的坑：**即使代码写了 INT8 版，如果没有启用编译开关，会静默丢失硬件加速，变成纯 C 参考实现**。

```cpp
// kernels/conv.h
#if defined(CMSIS_NN) || defined(XTENSA)
    // 硬件加速版：由 CMSIS-NN 或 Xtensa 提供
    TFLMRegistration Register_CONV_2D_INT8();
#else
    // 丢失加速！退回到通用版
    inline TFLMRegistration Register_CONV_2D_INT8() {
        return Register_CONV_2D();  // → Eval() 带 switch 分发的通用版
    }
#endif
```

如果没有启用 `CONFIG_TENSORFLOW_LITE_MICRO_CMSIS_NN_KERNELS`，编译时 `CMSIS_NN` 宏未定义，`Register_CONV_2D_INT8()` 会变成 `Register_CONV_2D()`。

```
硬件加速路径 (编译开关开启):
  Register_CONV_2D_INT8()
    → cmsis_nn/conv.cc
    → RegisterOp(Init, Prepare, EvalInt8)
    → EvalInt8() 直接调用 arm_convolve_wrapper_int8_s8()
    → 底层通过 C intrinsics（SMLAD/SXTB16 等）使用 DSP 指令

纯 C 参考路径 (编译开关未开启):
  Register_CONV_2D_INT8()
    → Register_CONV_2D()
    → RegisterOp(Init, Prepare, Eval)
    → Eval() 内部 switch(tensor_type)
    → case kTfLiteInt8: reference_ops::Conv()
    → 底层是纯 C 循环，无 SIMD/DSP 加速
```

注意 Generic 路径的 `Eval()` 也能正确处理 INT8——计算结果和 CMSIS-NN 版完全一致，只是底层实现不同。


### 1.6 硬件加速的底层机制

#### Cortex-M7 DSP 的核心指令

**`SMLAD`——双 16-bit 乘加，SIMD 的核心**

```asm
SMLAD Rd, Rn, Rm, Ra
; Rd = Ra + Rn[15:0]×Rm[15:0] + Rn[31:16]×Rm[31:16]
; 一个 32-bit 寄存器里打包两个 int8 对 → 2 次乘加 = 1 条指令 1 个周期
```

纯 C 实现需要 2 次乘法 + 1 次加法 = 至少 3 条指令。SMLAD 一条指令完成。

**`SMLAL`——64-bit 乘累加（量化 requantize）**

```asm
SMLAL RdLo, RdHi, Rn, Rm
; RdHi:RdLo += Rn × Rm     (32×32 → 64-bit 乘累加)
```

用于 per-channel 量化中的 `val × multiplier` 计算。纯 C 实现需要 64-bit 中间变量 + 多条移位/加法。CMSIS-NN 用 `SMLAL` 加上 `lsr #31` 和 `orr` 拼接完成高精度定点乘法，比纯 C 软件模拟更高效。


#### DSP 指令的寄存器使用

Cortex-M7 的 DSP 扩展（SMLAD/SXTB16 等）操作 **32-bit 通用寄存器（GPR：R0-R12）**，不涉及 FPU 的 D/Q 寄存器。SMLAD 把一个 32-bit GPR 拆成两个 16-bit 半字做双乘加。FPU 寄存器（D0-D15）由浮点运算使用，与 DSP 的 GPR 操作互不干扰。

这与 Helium/NEON 不同。Helium 和 NEON 有真正的**向量寄存器**（Q 寄存器，128-bit），这些 Q 寄存器与 FPU 共享同一组物理存储。 Cortex-M7 的 DSP 指令没有这个概念。

```
Cortex-M7 的寄存器关系:

  GPR (R0-R12)           FPU 寄存器
  ┌──────────────┐       ┌────────────────────────────┐
  │ DSP 指令使用  │       │ S0-S31 (32×32-bit)         │
  │ SMLAD, SXTB16│       │ D0-D15 (16×64-bit)         │
  │ SXTAB16, ... │       │ 仅供浮点运算使用            │
  └──────────────┘       └────────────────────────────┘
       ↓ 独立                    ↓ 内部共享
   整数 SIMD 操作          D0 = S0:S1, D1 = S2:S3 ...
   无向量寄存器            S/D 是同一物理存储的两个视图
```

```
Helium (Cortex-M55) 的寄存器关系:

  GPR (R0-R12)           向量/FPU 寄存器（共享物理存储）
  ┌──────────────┐       ┌────────────────────────────┐
  │ 通用整数运算  │       │ S0-S31 / D0-D15 / Q0-Q7   │
  └──────────────┘       │ Helium 向量指令 ←→ FPU 浮点 │
                         │ 三者是同一物理存储的三个视图  │
                         └────────────────────────────┘
```

#### DSP 加速因素分析

CMSIS-NN 相比纯 C Reference 的加速来自多个因素的叠加：

| 因素 | 纯 C Reference | CMSIS-NN DSP |
|------|---------------|-------------|
| 乘加并行度 | 每次 1 个 MAC | 每次 2 个 MAC（SMLAD） |
| Requantize | 64-bit 软件模拟 | SMLAL 64 位乘累加 |
| 循环展开 | 编译器普通优化 | 手写 4x 展开，减少循环开销 |
| 数据预处理 | 逐元素转换 | SXTB16/SXTAB16 批量展开 |
| Im2col + GEMM | 逐元素访问 | 批量矩阵乘，缓存友好 |

#### 更高性能路径：Helium 与 NEON

Cortex-M7 DSP 只是起点。ARM 三个 profile 各有更高性能的向量扩展方案：

| | Cortex-M7 DSP | Helium (M55) | NEON (AArch32, R52) | NEON (AArch64, A53) |
|--|--|--|--|--|
| 寄存器类型 | GPR (32-bit) | Q0-Q7 (128-bit) | Q0-Q15 (128-bit) | Q0-Q31 (128-bit) |
| 与 FPU 寄存器关系 | 独立（GPR vs D0-D15） | 共享物理存储 | 共享物理存储 | 共享物理存储 |
| INT8 MAC/周期 | 2 | 16 | 16 | 16 |
| INT8 加载/周期 | 1 | 16 | 16 | 16 |

Cortex-M7 是唯一"DSP 用 GPR、FPU 用 D 寄存器、两者独立"的方案。Helium 和 NEON 的向量指令操作 Q 寄存器，Q 与 FPU 的 S/D 共享同一组物理存储——执行向量指令时，这些寄存器被当作 Q 视图使用；执行浮点指令时，被当作 S/D 视图使用。

Helium 的关键指令（128-bit 向量，16 个 int8 同时操作）：

```c
// 加载 16 个 int8（vs M7 逐个 load）
int8x16_t wgt = vldrbq_s8(weight_ptr);

// 16 个 int8 乘加，累加到 int32
int32_t acc = vmladavq_s8(inp, wgt);    // 一条指令 = 16 次乘加

// int16 → 饱和截断为 int8（16 个同时）
int8x16_t out = vqmovnbq_s16(acc_vec);
```

NEON 的指令形式类似，但寄存器更多（AArch32 有 16 个 Q，AArch64 有 32 个 Q），循环展开时更多数据常驻寄存器，减少 load/store。

回到示例项目：STM32H747 (Cortex-M7) 使用 DSP 扩展已是该芯片的最优路径。若换用 Cortex-M55 (Helium)，凭借 128-bit 向量宽度（一次处理 16 个 int8），推理速度可显著提升。要跑更大模型（如 MobileNetV2），则需要 Cortex-A + NEON 的算力。

### 1.7 注册阶段完整流程图

```
编译期:
  MicroMutableOpResolver<20> resolver;
  ├─ registrations_[20]  固定数组，栈/静态分配
  ├─ builtin_codes_[20]
  └─ builtin_parsers_[20]

运行时:
  resolver.AddConv2D(Register_CONV_2D_INT8())
       │
       ├─ Register_CONV_2D_INT8()
       │     └─ RegisterOp(Init, Prepare, EvalInt8)
       │           └─ 返回 TFLMRegistration {init=Init, invoke=EvalInt8, ...}
       │
       └─ AddBuiltin(CONV_2D, registration, ParseConv2D)
             ├─ FindOp(CONV_2D) → nullptr (未注册过)
             ├─ registrations_[0] = {init, nullptr, prepare, EvalInt8, nullptr}
             ├─ registrations_[0].builtin_code = CONV_2D
             ├─ registrations_len_++
             ├─ builtin_codes_[0] = CONV_2D
             ├─ builtin_parsers_[0] = ParseConv2D
             └─ num_buitin_ops_++

  resolver.AddMaxPool2D(Register_MAX_POOL_2D_INT8())
       └─ registrations_[1] = {init, nullptr, prepare, EvalInt8, nullptr}
          registrations_[1].builtin_code = MAX_POOL_2D
          registrations_len_ = 2

  ... 共注册 7 种算子（16 个实例共用）
```

**注册阶段零堆分配**。所有数据都在编译期确定的固定数组中。
