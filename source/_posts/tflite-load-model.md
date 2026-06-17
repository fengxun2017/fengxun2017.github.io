---
title: TFLite Micro 深度剖析（一）：模型加载——FlatBuffer 零拷贝的秘密（基于AI协助总结）
date: 2026/5/18
categories: 
- tflite
---

<center>
本文是 TFLite Micro 内部实现深度剖析系列的第 1 篇，主要介绍 TFLite Micro的模型加载相关内容。本篇同时提供示例模型的完整结构与数据，作为后续各篇的前置信息。
</center>
<!--more-->

***


## 前置信息：示例模型参考

本系列使用一个实际的 INT8 量化 CNN 目标检测模型（C2_BN）贯穿全篇。本节给出该模型的**完整结构**和 **.tflite 文件中的具体数据**，后续各篇将直接引用。

### 软件版本与运行环境

| 组件 | 版本 | 说明 |
|------|------|------|
| TFLite Micro | `8d404de7` (2024-06-27) | Zephyr module fork |
| FlatBuffers | v23.5.26 | `FLATBUFFERS_VERSION = 23.5.26` |
| CMSIS-NN | commit `22080c68` | ARM-software/CMSIS-NN |
| Schema | v3c (`TFL3`) | `schema.fbs` → `schema_generated.h` |
| 目标 MCU | STM32H747 (Cortex-M7 480MHz) | AXI SRAM 512KB + SDRAM2 4MB |

### 模型架构

示例训练时包含 BatchNorm 层。转换为 .tflite 时，BN 参数被吸收进 Conv2D 的权重和偏置中，ReLU6 融合为 Conv2D 的输出 clamp 范围 [0, 6]。因此 .tflite 中看不到独立的 BN 和 ReLU6 算子——它们在转换阶段就被融合掉了。

BN 融合的推导过程：

```
原始计算链:  Conv → BN → ReLU6

BN 的数学定义:
  y = γ × (x - μ) / σ + β

其中 x = Σ(W·in) + B（Conv 的输出），代入展开:
  y = γ × (Σ(W·in) + B - μ) / σ + β

将乘法分配到求和内部:
  y = Σ(W × (γ/σ) × in) + B × (γ/σ) + (β - γμ/σ)
      ~~~~~~~~~~~~~~~~       ~~~~~~~~~~   ~~~~~~~~~~~~~
      = W_fused × in        = B_fused 的一部分

定义融合后的参数:
  W_fused = W × (γ/σ)              ← 权重逐通道缩放
  B_fused = B × (γ/σ) + (β - γμ/σ) ← 偏置吸收均值和方差

融合后:  Conv_fused(W_fused, B_fused) → ReLU6
```

融合后只需一次矩阵乘加，省去了 BN 层的额外计算和中间 tensor。

T 表示 tensor。

```
输入: T[0]  [1,120,160,1] int8  S=0.003922  Z=-128
          │
  ┌───────┴────────┐
  │ Op[0] CONV_2D  │  1→32通道, 3×3, padding=SAME
  │ +BN融合+ReLU6  │  ← BN 在转换时融入 Conv 权重，ReLU6 融合为 clamp
  └───────┬────────┘
          │ T[15] [1,120,160,32]  S=0.035991  Z=-128
  ┌───────┴────────┐
  │ Op[1] MAX_POOL │  2×2, stride=2
  └───────┬────────┘
          │ T[16] [1,60,80,32]    S=0.035991  Z=-128   ← MaxPool 前后 S/Z 不变
  ┌───────┴────────┐
  │ Op[2] CONV_2D  │  32→64通道, 3×3
  │ +BN融合+ReLU6  │
  └───────┬────────┘
          │ T[17] [1,60,80,64]    S=0.070534  Z=-128
  ┌───────┴────────┐
  │ Op[3] MAX_POOL │  2×2, stride=2
  └───────┬────────┘
          │ T[18] [1,30,40,64]    S=0.070534  Z=-128
  ┌───────┴────────┐
  │ Op[4] CONV_2D  │  64→128通道, 3×3
  │ +BN融合+ReLU6  │
  └───────┬────────┘
          │ T[19] [1,30,40,128]   S=0.041528  Z=-128
  ┌───────┴────────┐
  │ Op[5] MAX_POOL │  2×2, stride=2
  └───────┬────────┘
          │ T[20] [1,15,20,128]   S=0.041528  Z=-128
  ┌───────┴────────┐
  │ Op[6] CONV_2D  │  128→128通道, 3×3
  │ +BN融合+ReLU6  │
  └───────┬────────┘
          │ T[21] [1,15,20,128]   S=0.036833  Z=-128
  ┌───────┴────────┐
  │ Op[7] CONV_2D  │  128→5通道, 1×1 (检测头)
  └───────┬────────┘
          │ T[22] [1,15,20,5]     S=0.130216  Z=73
     ┌────┴────┐
     │ Op[8]   │  StridedSlice 分离置信度(4ch)和 bbox(1ch)
     └────┬────┘
      ┌───┴───┐
      ↓       ↓
 T[23][..,4]  T[26][..,1]  S=0.130216  Z=73
   │           │
 Op[9]       Op[12]        Logistic (Sigmoid，int8 查表实现)
   │           │
 T[24]       T[27]         S=0.003906  Z=-128
   │           │
 Op[10]      Op[13]        Quantize (重量化，对齐 scale 以支持 Concat)
   │           │
 T[25]       T[28]         S=0.003918  Z=-128
   │           │
   └─────┬─────┘
   Op[14] CONCATENATION    ← 要求两个分支 S/Z 一致才能拼接
         │
   T[29] [1,15,20,5]  S=0.003918  Z=-128
         │
   Op[15] DEQUANTIZE   ← int8 → float32，最终输出
         │
   T[30] [1,15,20,5]  float32
```

### Tensor 数据表

`.tflite` 文件的 `subgraphs[0].tensors[]` 数组定义了 **31 个 tensor**（T[0]~T[30]）。本文用 `T[n]` 作为 `tensors[n]` 的简写。

**算子常量**（buffer 指向非空数据，运行时零拷贝读取）：

| Tensor | 内容 | 形状 | 类型 | 量化方式 | scale 范围 | Z |
|--------|------|------|:----:|----------|-----------|---|
| T[1] | StridedSlice begin | [4] | int32 | — | — | — |
| T[2] | StridedSlice end | [4] | int32 | — | — | — |
| T[3] | StridedSlice end | [4] | int32 | — | — | — |
| T[4] | StridedSlice strides | [4] | int32 | — | — | — |
| T[5] | Conv5 bias | [5] | int32 | per-channel | 0.000064 ~ 0.000149 | 0 |
| T[6] | Conv5 filter | [5,1,1,128] | int8 | per-channel 对称 | 0.001745 ~ 0.004041 | 0 |
| T[7] | Conv4 bias | [128] | int32 | per-channel | 0.000014 ~ 0.000072 | 0 |
| T[8] | Conv4 filter | [128,3,3,128] | int8 | per-channel 对称 | 0.000332 ~ 0.001743 | 0 |
| T[9] | Conv3 bias | [128] | int32 | per-channel | 0.000031 ~ 0.000087 | 0 |
| T[10] | Conv3 filter | [128,3,3,64] | int8 | per-channel 对称 | 0.000442 ~ 0.001237 | 0 |
| T[11] | Conv2 bias | [64] | int32 | per-channel | 0.000024 ~ 0.000079 | 0 |
| T[12] | Conv2 filter | [64,3,3,32] | int8 | per-channel 对称 | 0.000660 ~ 0.002191 | 0 |
| T[13] | Conv1 bias | [32] | int32 | per-channel | 0.000022 ~ 0.000183 | 0 |
| T[14] | Conv1 filter | [32,3,3,1] | int8 | per-channel 对称 | 0.00572 ~ 0.04677 | 0 |

**激活值**（buffer 指向空数据，运行时由 Arena 分配）：

| Tensor | 形状 | scale | Z | 含义 |
|--------|------|-------|---|------|
| T[0] | [1,120,160,1] | 0.003922 | -128 | 输入灰度图（对应 float [0,1]） |
| T[15] | [1,120,160,32] | 0.035991 | -128 | Conv1+BN+ReLU6 |
| T[16] | [1,60,80,32] | 0.035991 | -128 | MaxPool1 |
| T[17] | [1,60,80,64] | 0.070534 | -128 | Conv2+BN+ReLU6 |
| T[18] | [1,30,40,64] | 0.070534 | -128 | MaxPool2 |
| T[19] | [1,30,40,128] | 0.041528 | -128 | Conv3+BN+ReLU6 |
| T[20] | [1,15,20,128] | 0.041528 | -128 | MaxPool3 |
| T[21] | [1,15,20,128] | 0.036833 | -128 | Conv4+BN+ReLU6 |
| T[22] | [1,15,20,5] | 0.130216 | **73** | Conv5 检测头（Z≠-128） |
| T[23] | [1,15,20,4] | 0.130216 | 73 | StridedSlice 置信度 |
| T[24] | [1,15,20,4] | 0.003906 | -128 | Logistic 置信度 |
| T[25] | [1,15,20,4] | 0.003918 | -128 | Quantize 置信度 |
| T[26] | [1,15,20,1] | 0.130216 | 73 | StridedSlice bbox |
| T[27] | [1,15,20,1] | 0.003906 | -128 | Logistic bbox |
| T[28] | [1,15,20,1] | 0.003918 | -128 | Quantize bbox |
| T[29] | [1,15,20,5] | 0.003918 | -128 | Concat 拼合输出 |
| T[30] | [1,15,20,5] | — | — | Dequantize 最终输出（float32） |

### 算子类型汇总

模型使用 16 个算子实例，但只定义了 **7 种**算子类型。多个同类算子共用同一个 `operator_codes` 条目，通过 `opcode_index` 引用：

| opcode_index | BuiltinOperator | 枚举值 | 实例数 | 算子编号 |
|:---:|---|:---:|:---:|---|
| 0 | CONV_2D | 3 | 5 | Op[0,2,4,6,7] |
| 1 | MAX_POOL_2D | 17 | 3 | Op[1,3,5] |
| 2 | STRIDED_SLICE | 45 | 2 | Op[8,11] |
| 3 | LOGISTIC | 14 | 2 | Op[9,12] |
| 4 | QUANTIZE | 114 | 2 | Op[10,13] |
| 5 | CONCATENATION | 2 | 1 | Op[14] |
| 6 | DEQUANTIZE | 6 | 1 | Op[15] |

---

### 1.1 FlatBuffer 与 .tflite 文件

FlatBuffer 是 Google 开发的一种**零拷贝序列化格式**。传统序列化（JSON、XML、Protocol Buffers）需要先解析成内存结构才能使用，FlatBuffer 不需要——程序拿到二进制数据后，直接通过指针偏移就能访问任意字段，无需 malloc、无需反序列化。

TFLite 模型文件（`.tflite`）就是用 FlatBuffer 编码的二进制文件。TFLM 选择它有三个原因：

1. **零解析**：不需要先解析再使用，直接拿到指针
2. **零 malloc**：不需要动态分配内存来存储解析结果
3. **紧凑**：没有额外开销，适合嵌入式设备

**Schema 驱动**：`.tflite` 的数据结构由 `schema.fbs` 定义，经 `flatc` 编译器生成 `schema_generated.h`（25000+ 行 C++ 代码）。应用代码调用的 `GetModel()` 就定义在其中（详见 1.5 节）。

```
schema.fbs  ──(flatc)──→  schema_generated.h
                               ↓
                          GetModel(), Model::version(), ...
```

#### 1.1.1 .tflite 文件的逻辑结构

示例模型（257,304 字节，schema v3c）的逻辑层次：

```
Model (根表)
├── version = 3                    → schema 版本号
├── operator_codes[]               → 算子类型列表（7 种，去重后）
│   ├── [0] CONV_2D
│   ├── [1] MAX_POOL_2D
│   └── [2..6] STRIDED_SLICE, LOGISTIC, QUANTIZE, CONCATENATION, DEQUANTIZE
├── subgraphs[]                    → 子图数组
│   └── [0] 主子图
│       ├── tensors[]              → 31 个张量描述
│       ├── operators[]            → 16 个算子实例（按执行顺序）
│       ├── inputs[] = [0]         → 模型输入 tensor 索引
│       └── outputs[] = [30]       → 模型输出 tensor 索引
├── buffers[]                      → 35 个数据块
│   ├── [0] 空 (sentinel)
│   ├── [1] 空 (T[0] 输入)
│   ├── [2..15] 常量数据 (权重、偏置、StridedSlice 参数)
│   ├── [16..31] 空 (激活值)
│   └── [32..34] metadata 字符串
└── metadata[]                     → 元数据
```

所有对象之间通过**数组索引**引用，而非内存指针。例如 `operators[0].inputs=[0,14,13]` 指向 `tensors[]` 数组，`tensors[14].buffer=15` 指向 `buffers[]` 数组。

### 1.2 Schema 详解

#### 1.2.1 六个核心 table

Schema 中定义了 6 个核心表，构成模型的完整描述：

**Model（根表）**

```
table Model {
  version: uint                  → schema 版本号（当前 = 3）
  operator_codes: [OperatorCode] → 算子类型表（去重后的）
  subgraphs: [SubGraph]          → 子图数组（0号是主图）
  description: string            → 描述
  buffers: [Buffer]              → 数据块数组（权重、偏置）
  metadata_buffer: [int]         → 元数据缓冲区索引（已弃用）
  metadata: [Metadata]           → 元数据
  signature_defs: [SignatureDef] → 签名定义
}
```

**SubGraph（子图）**

```
table SubGraph {
  tensors: [Tensor]       → 该子图所有 tensor 的描述
  inputs: [int]           → 输入 tensor 索引（如 [0]）
  outputs: [int]          → 输出 tensor 索引（如 [30]）
  operators: [Operator]   → 算子列表（按执行顺序排列）
}
```

**Operator（算子实例）**

```
table Operator {
  opcode_index: uint            → 指向 operator_codes[] 的索引
  inputs: [int]                 → 输入 tensor 索引列表（-1 表示可选/无）
  outputs: [int]                → 输出 tensor 索引列表
  builtin_options: BuiltinOptions → 算子参数（联合体，不同算子不同类型）
}
```

**Tensor（张量描述）**

```
table Tensor {
  shape: [int]                        → 形状 [batch, height, width, channels]
  type: TensorType                    → 类型 (FLOAT32=0, INT8=9, ...)
  buffer: uint                        → 指向 buffers[] 的索引
  quantization: QuantizationParameters → 量化参数
}
```

**OperatorCode（算子类型）**

```
table OperatorCode {
  deprecated_builtin_code: int    → 已弃用，向后兼容
  custom_code: string             → 自定义算子名
  version: int                    → 算子版本号
  builtin_code: BuiltinOperator   → 算子枚举 (CONV_2D=3, MAX_POOL_2D=17, ...)
}
```

**Buffer（数据块）**

```
table Buffer {
  data: [ubyte] (force_align: 16);  → 权重、偏置等常量数据（向量自带长度前缀）
  offset: ulong;                    → 大模型(>2GB)使用，数据在 FlatBuffer 外部
  size: ulong;                      → 大模型专用
}
```

示例模型只用 `data` 字段，二进制中分两种：

- **非空**：`data` 指向 `[length:4B][byte₀]...[byteₙ]` 向量，如 buffers[15] 存放 Conv1 filter 288 字节
- **空**：vtable 中 `data` 偏移为 0，`data()` 返回 `nullptr`，如 buffers[16]（Conv1 output 无预存数据）

多个空 Buffer 共享同一个最小 vtable（FlatBuffer 去重优化）。

#### 1.2.2 Tensor.buffer 字段与数据来源

`Tensor.buffer` 是 `buffers[]` 数组的索引，决定该 tensor 的数据存在哪里。运行时通过 `GetFlatbufferTensorBuffer()` 检查对应 buffer 是否包含实际数据：

```cpp
void* GetFlatbufferTensorBuffer(tensor, buffers) {
    auto* buffer = (*buffers)[tensor.buffer()];
    if (buffer && buffer->data() && buffer->data()->size() > 0)
        return const_cast<void*>(buffer->data()->data());
    return nullptr;
}
```

注意：判断依据是 **buffer 中是否有实际数据**（`size() > 0`），而非 buffer 索引本身。

两种情况：

| buffer 数据状态 | 初始化后 data 指针 | 推理时处理 | 参与内存规划 |
|:---:|---|---|:---:|
| 非空（有权重字节） | 直接指向 .tflite 文件内数据 | 零拷贝，推理时只读 | 否 |
| 空（size=0 或不存在） | `nullptr` | `CommitPlan()` 在 Arena head 区分配地址（第 4 篇） | 是 |

示例模型中，转换器为每个 tensor 分配了唯一的 buffer 索引（1~31），没有 tensor 使用默认的 buffer=0：

```
T[0]  输入         buffer=1  → buffers[1]  空    → data=nullptr → Arena 分配
T[14] Conv1 filter buffer=15 → buffers[15] 288B  → data 直接指向 .tflite 文件
T[13] Conv1 bias   buffer=14 → buffers[14] 128B  → data 直接指向 .tflite 文件
T[15] Conv1 output buffer=16 → buffers[16] 空    → data=nullptr → Arena 分配
```

权重 tensor 在初始化阶段就拿到了有效指针（零拷贝），不需要参与后续的内存规划。只有 buffer 为空的 tensor 才需要在 Arena 中分配空间。

#### 1.2.3 算子参数——BuiltinOptions 联合体

`Operator.builtin_options` 是联合体（union），不同算子指向不同参数表：

```
Conv2DOptions {
  padding: Padding                 → SAME / VALID
  stride_w, stride_h: int          → 步幅
  fused_activation_function         → NONE / RELU / RELU6
  dilation_w/h_factor: int          → 膨胀系数
}

Pool2DOptions {
  padding, stride_w, stride_h
  filter_width, filter_height
  fused_activation_function
}

StridedSliceOptions {
  begin_mask, end_mask, ellipsis_mask, new_axis_mask, shrink_axis_mask: int
  ...
}
```

#### 1.2.4 量化参数

```
QuantizationParameters {
  scale: [float]          → 量化比例因子（per-channel 时每个通道一个）
  zero_point: [long]      → 零点偏移
  quantized_dimension: int → 按哪个维度量化（通常=3，通道维度）
}

反量化公式: float_value = scale × (int8_value - zero_point)
量化公式:   int8_value = round(float_value / scale + zero_point)
```

#### 1.2.5 索引引用追踪——以 Conv1 为例

`operators[0]` 是第一个 Conv2D 算子，通过索引串联所有相关对象：

```
operators[0] = { opcode_index=0, inputs=[0,14,13], outputs=[15] }
                 │                 │                   │
                 │                 │                   └─ 输出 → T[15] {Conv1 output, buffer=16→空}
                 │                 │
                 │                 ├─ inputs[0] → T[0]  {输入图像, buffer=1→空}
                 │                 ├─ inputs[1] → T[14] {Conv1 filter, buffer=15→288B}
                 │                 └─ inputs[2] → T[13] {Conv1 bias, buffer=14→128B}
                 │
                 └─ opcode_index=0 → operator_codes[0] = CONV_2D
```

注意 inputs 的顺序：`[input, filter, bias]`。这是 Conv2D 算子的固定约定，定义在 schema 的算子签名中。整个模型中没有任何内存地址/指针——所有引用都是数组下标。

### 1.3 FlatBuffer 二进制格式——导航规则

掌握了逻辑结构和 schema 之后，本节揭示这些内容在二进制中是如何编码的。理解这一层才能看懂源码中的 `GetRoot` / `GetField` / `GetPointer`。

#### 1.3.1 导航的三个基本操作

FlatBuffer 的所有数据访问只有 3 种操作：

**操作 1：soffset 向后跳（减法）——找 vtable**

每个表的前 4 字节都是 `soffset_t`（有符号 32 位），指向该表的 vtable：

```cpp
// flatbuffers/table.h
const uint8_t *GetVTable() const {
    return data_ - ReadScalar<soffset_t>(data_);   // 减法，向后跳
}
```

**操作 2：uoffset 向前跳（加法）——找根表、读引用字段**

用于定位根表和所有引用类型字段（string、vector、子表）：

```cpp
// 找根表 — flatbuffers/buffer.h
buf + *(uoffset_t*)buf                              // 加法，向前跳

// 读引用字段 — flatbuffers/table.h
auto p = data_ + field_offset;
p + *(uoffset_t*)p                                   // 加法，向前跳
```

**操作 3：直接读取（不跳）——读标量字段**

`uint`、`int`、`float`、`bool`、`enum` 等标量在表中内联存储，直接读取：

```cpp
// flatbuffers/table.h
ReadScalar<T>(data_ + field_offset)                  // 直接读值
```

#### 1.3.2 vtable——字段寻址表

每个 FlatBuffer 表都有一个 **vtable**（虚表），它告诉运行时每个字段在表数据中的偏移位置。

vtable 布局：

```
[vtable_size: uint16][object_size: uint16][field 0 偏移: uint16][field 1 偏移: uint16]...
```

- `vtable_size`：vtable 自身大小（字节）
- `object_size`：表数据大小（字节）
- 后面每 2 字节是一个字段的偏移，**顺序 = schema 声明顺序**
- 偏移 = 0 表示该字段不存在（用于向前兼容；也用于标量字段值为默认值时省略存储）

#### 1.3.3 谁决定用哪种操作

| 决定者 | 规则 | 依据 |
|:------:|:-----|:-----|
| **格式规范**（源码硬编码） | 每个表前 4 字节 = soffset，固定减法找 vtable | `GetVTable()` 写死的 |
| **schema 字段类型**（flatc 生成代码选择） | 标量 → `GetField` 直接读；引用 → `GetPointer` 加法跳 | flatc 根据 `.fbs` 生成代码 |

同一块 4 字节数据，schema 说它是 `uint` 就当标量直接读，说它是 `[Buffer]` 就当 uoffset 跳转。**二进制本身不标记类型——解读方式完全由 schema 决定**。

#### 1.3.4 核心规则总结

| # | 规则 | 说明 |
|:-:|------|:-----|
| 1 | soffset 永远用减法 | 表前 4 字节，向回找 vtable |
| 2 | uoffset 永远用加法 | 根表定位、引用字段跳转 |
| 3 | 标量直接读不跳 | uint/int/float 等在表中内联存储 |
| 4 | vtable 索引 = schema 声明顺序 | field 0 对应第一个声明的字段 |
| 5 | vtable 偏移 = 0 表示字段不存在 | 向前兼容 + 默认值省略 |
| 6 | 二进制不标记类型 | 同样的 4 字节，schema 决定读法 |

### 1.4 字节级验证：用真实模型走一遍

用示例模型（257,304 字节）的真实二进制数据逐步验证导航规则。
buf 为 .tflite 模型实际二进制数据数组。

**Step 1：找根表（uoffset 加法）**

```
buf[0..3]:  20 00 00 00     uoffset = 32
            │
            │  buf + 32
            ▼
buf[32]:    Model 表数据起始
```

**Step 2：找 vtable（soffset 减法）**

```
buf[32..35]: 14 00 00 00    soffset = 20
             │
             │  buf[32] - 20
             ▼
buf[12]:     vtable 起始
```

**Step 3：读取 vtable**

```
buf[12..31] = vtable (20 字节)

偏移   字节      含义
[12]   14 00     vtable_size = 20
[14]   20 00     object_size = 32
[16]   1c 00     field 0 (version)          偏移 = 28
[18]   18 00     field 1 (operator_codes)   偏移 = 24
[20]   14 00     field 2 (subgraphs)        偏移 = 20
[22]   10 00     field 3 (description)      偏移 = 16
[24]   0c 00     field 4 (buffers)          偏移 = 12
[26]   00 00     field 5 (metadata_buffer)  偏移 = 0 → 不存在
[28]   08 00     field 6 (metadata)         偏移 = 8
[30]   04 00     field 7 (signature_defs)   偏移 = 4
```

vtable 中的 field 索引 = schema 声明顺序：

```
table Model {                                    // schema.fbs
  version:uint;                  ← field 0      // 标量
  operator_codes:[OperatorCode]; ← field 1      // 引用
  subgraphs:[SubGraph];          ← field 2      // 引用
  description:string;            ← field 3      // 引用
  buffers:[Buffer];              ← field 4      // 引用
  metadata_buffer:[int];         ← field 5      // 引用 (不存在)
  metadata:[Metadata];           ← field 6      // 引用
  signature_defs:[SignatureDef]; ← field 7      // 引用
}
```

**Step 4：读取各字段**

标量字段——直接读：

```
Field 0: version (uint, 标量)
  位置: buf[32 + 28] = buf[60]
  数据: 03 00 00 00
  → version = 3
```

引用字段——uoffset 加法跳转：

```
Field 4: buffers ([Buffer], 引用)
  位置: buf[32 + 12] = buf[44]
  数据: 94 00 00 00 → uoffset = 148
  跳转: buf[44 + 148] = buf[192]
  → buffers 向量在 buf[192]

Field 6: metadata ([Metadata], 引用)
  位置: buf[32 + 8] = buf[40]
  数据: 1c 00 00 00 → uoffset = 28
  跳转: buf[40 + 28] = buf[68]
  → metadata 向量在 buf[68]

Field 7: signature_defs ([SignatureDef], 引用)
  位置: buf[32 + 4] = buf[36]
  数据: 1c 00 00 00 → uoffset = 28
  跳转: buf[36 + 28] = buf[64]
  → signature_defs 向量在 buf[64]
```

**Step 5：读取向量**

到达向量后，前 4 字节是长度，后面是元素：

```
signature_defs 向量 (buf[64]):
  [64..67]: 00 00 00 00 → length = 0（空）

metadata 向量 (buf[68]):
  [68..71]: 03 00 00 00 → length = 3
  [72..75]: 54 00 00 00 → metadata[0] uoffset=84 → buf[156]
  [76..79]: 2c 00 00 00 → metadata[1] uoffset=44 → buf[120]
  [80..83]: 04 00 00 00 → metadata[2] uoffset=4  → buf[84]

buffers 向量 (buf[192]):
  [192..195]: 23 00 00 00 → length = 35
  [196..335]: 35 × 4字节 uoffset → 每个 Buffer 子表
```

**Step 6：子表（Metadata）——规则完全相同**

Schema 定义：

```
table Metadata {
  name: string;    ← field 0  (引用，uoffset)
  buffer: uint;    ← field 1  (标量，直接读)
}
```

```
metadata[2] 在 buf[84]:
  [84..87]: 30 4a fc ff → soffset = -243152 (有符号)
            │
            │  buf[84] - (-243152) = buf[243236]
            ▼
            vtable（3个 Metadata 表共享此 vtable）

  vtable 映射:
    field 0 (name:string)   → 偏移 = 8
    field 1 (buffer:uint)   → 偏移 = 4

  读取:
    field 1 (buffer, 标量): buf[84+4] = buf[88] = 22 00 00 00 → 34
    field 0 (name, 引用):   buf[84+8] = buf[92] = 04 00 00 00
                            uoffset=4 → buf[92+4] = buf[96]
                            → "CONVERSION_METADATA"
```

三个 Metadata 条目：

| 条目 | name | buffer 索引 |
|:----:|:----:|:----------:|
| [0] | min_runtime_version | 32 |
| [1] | min_runtime_version | 33 |
| [2] | CONVERSION_METADATA | 34 |

注意：Metadata 的 vtable 中 field 0 (name) 偏移 = 8、field 1 (buffer) 偏移 = 4。物理存储顺序与 schema 声明顺序不同，但 vtable 正确映射了 field index → 物理偏移。FlatBuffer 编译器可以自由安排字段在表数据中的物理位置（通常为了对齐优化）。

#### 1.4.1 完整导航流程图

```
buf[0]: uoffset=32
    │
    │ uoffset 加法
    ▼
buf[32]: Model 表  (soffset=20 → vtable 在 buf[32-20]=buf[12]，低地址方向)
    │
    │  vtable (buf[12]) 告诉每个字段的偏移位置:
    │
    ├── buf[60] field 0 (version)     → 标量，直接读 → 3
    │
    ├── buf[56] field 1 (op_codes)    → uoffset=257084 ──加法──→ buf[257140]
    │
    ├── buf[52] field 2 (subgraphs)   → uoffset=242224 ──加法──→ buf[242276]
    │
    ├── buf[48] field 3 (description) → uoffset=242208 ──加法──→ buf[242256]
    │
    ├── buf[44] field 4 (buffers)     → uoffset=148 ──加法──→ buf[192]
    │                                                          │
    │                                                          ▼
    │                                                    len=35, 35个Buffer子表
    │
    ├── field 5 (metadata_buffer)     → 偏移=0, 不存在
    │
    ├── buf[40] field 6 (metadata)    → uoffset=28 ──加法──→ buf[68]
    │                                                          │
    │                                                          ▼
    │                                                    len=3, 3个Metadata子表
    │
    └── buf[36] field 7 (sig_defs)    → uoffset=28 ──加法──→ buf[64]
                                                             │
                                                             ▼
                                                       len=0, 空向量
```

### 1.5 代码中的模型加载实践

#### 1.5.1 GetModel()——零拷贝的本质

`GetModel()` 定义在 `schema_generated.h`（由 `flatc` 编译器自动生成）：

```cpp
inline const tflite::Model *GetModel(const void *buf) {
  return ::flatbuffers::GetRoot<tflite::Model>(buf);
}
```

`flatbuffers::GetRoot<T>()` 的实际实现：

```cpp
// flatbuffers/buffer.h
template<typename T> T *GetMutableRoot(void *buf) {
  if (!buf) return nullptr;     // 空指针保护
  EndianCheck();                 // 运行时 assert 校验字节序
  return reinterpret_cast<T *>(
      reinterpret_cast<uint8_t *>(buf) +
      EndianScalar(*reinterpret_cast<uoffset_t *>(buf)));  // buf[0..3] 是 offset
}
```

结合 1.4 节 Step 1 的验证：读取 `buf[0..3]` = 32（uoffset），然后 `buf + 32` 得到 Model 表指针。

**关键**：没有任何内存拷贝、没有任何解析、没有任何构造。`Model*` 直接指向原始 buffer。

#### 1.5.2 为什么需要对齐

在 ARM Cortex-M 上，如果 FlatBuffer 数据没有按 4/8 字节对齐，访问时会触发 UNALIGNED fault。因此实际使用中需要将模型数据拷贝到对齐的缓冲区：

```cpp
// 将模型数据复制到对齐的缓冲区（解决 flash 中不对齐的问题）
memcpy(aligned_model_buffer, model_data_ptr, model_data_len);
model = tflite::GetModel(aligned_model_buffer);
```

`aligned_model_buffer` 通常用 `__attribute__((aligned(64)))` 保证 64 字节对齐。

#### 1.5.3 版本校验

```cpp
if (model->version() != TFLITE_SCHEMA_VERSION) {
    // 版本不匹配，字段偏移可能不同
    return error;
}
```

FlatBuffer schema 有版本号。`.tflite` 文件由某版本的 `flatc` 编译器生成，`TFLITE_SCHEMA_VERSION` 是 TFLM 库编译时的版本。两者必须匹配。

#### 1.5.4 零拷贝的类型转换技巧

FlatBuffer 的 `Vector<int32_t>` 和 TFLite 运行时的 `TfLiteIntArray` 内存布局相同：

```cpp
// tflite/micro/flatbuffer_utils.cc
TfLiteIntArray* FlatBufferVectorToTfLiteTypeArray(
    const flatbuffers::Vector<int32_t>* flatbuffer_array) {
    // 小端机器上，TfLiteIntArray 和 Vector<int32_t> 内存布局一致
    // 直接 reinterpret_cast，零拷贝！
    return const_cast<TfLiteIntArray*>(
        reinterpret_cast<const TfLiteIntArray*>(flatbuffer_array));
}
```

```
FlatBuffer Vector<int32_t>:  [length:4bytes][data0][data1][...]
TfLiteIntArray:              [size:int][data[0]][data[1]][...]
```

两者在内存中完全一致，直接强转。

#### 1.5.5 模型加载阶段的完整流程图

```
aligned_model_buffer (64B 对齐缓冲区)
       │
       ▼
tflite::GetModel(buffer)
       │
       ▼ flatbuffers::GetRoot<Model>()
       │ 读取 buffer[0:4] 作为 offset
       │ 返回 (buffer + offset) 强转为 Model*
       │
       ▼
model->version() 检查
       │
       ▼
model 指针就绪，后续传给 MicroInterpreter 构造函数
```

**模型加载阶段零动态内存分配，零拷贝，零解析**。真正的解析发生在后续的 AllocateTensors() 阶段。
