---
title: tricore architecture - Programming Model
date: 2025/5/21
categories: 
- Tricore
---

<center>
Programming Model
</center>

<!--more-->

***

## 1 数据类型
指令集支持以下数据类型的操作：
- 布尔值（Boolean）
- 位字符串（Bit String）
- 字节（Byte）
- 有符号分数（Signed Fraction）
- 地址（Address）
- 有符号和无符号整数（Signed and Unsigned Integers）
- **IEEE-754 单精度浮点数（Single-precision Floating-point Number）** 

注意：仅支持单精度浮点数。


大多数指令针对特定的数据类型进行操作，而某些指令可用于处理多个数据类型。

### 1.1 布尔值（Boolean）
布尔值仅有 TRUE（真） 或 FALSE（假）：
- TRUE：生成时的值为 1，测试时为 非零值。
- FALSE：值为 0。

布尔值通常作为比较指令和逻辑指令的结果，并用于逻辑运算和条件跳转指令的源操作数。

### 1.2 位字符串（Bit String）
位字符串是 一组紧密排列的位字段。
位字符串由逻辑指令、移位指令和位字段指令生成和使用。

### 1.3 字节（Byte）
字节是 8 位值，可用于表示字符或极短整数。不假设特定的编码格式。

### 1.4 有符号分数（Signed Fraction）
该架构支持 16 位、32 位和 64 位 的 有符号分数数据，用于 DSP 算术运算。数据值采用最高位符号位：
- 0 表示 正数（+）
- 1 表示 负数（-）
后续位表示 隐含的二进制小数点和分数，其数值范围为 [-1,1)。

### 1.5 地址（Address）
地址是 32 位无符号值，用于存储 内存地址。


### 1.6 有符号和无符号整数（Signed and Unsigned Integers）
有符号和无符号整数通常为 32位。当从内存加载到寄存器时，较短的有符号或无符号整数会被符号扩展（Sign-Extended）或 零扩展（Zero-Extended）到 32位。

**多精度（Multi-precision）**
支持带进位的加法和减法，用于多精度整数运算。整数在移位和掩码操作中被视为位字符串。多精度移位可以通过 单精度移位（Single-precision Shifts）和位字段提取（Bit Field Extracts）组合实现。
备注：多精度计算用于处理 超大整数 和 高精度浮点数。


### 1.7 IEEE-754 单精度浮点数（IEEE-754 Single-Precision Floating-Point Number）

根据核心架构的具体实现，IEEE-754 浮点数可以通过协处理器硬件指令（Coprocessor Hardware Instructions）或软件库调用（Software Calls to a Library）进行支持。


## 2 数据格式（Data Formats）
所有通用寄存器（GPRs，General Purpose Registers） 均为 32 位宽，大多数指令操作 32 位字（Word） 数据。

当 字节（Byte）或半字（Half-word）数据元素从内存加载时，它们会被 自动符号扩展（Sign-Extended）或零扩展（Zero-Extended） 以填充寄存器。

扩展类型 由 加载指令（Load Instruction） 隐式决定：
- LD.B：加载 字节 并进行 符号扩展。
- LD.BU：加载 字节 并进行 零扩展。

支持的数据格式
- 位（Bit）
- 字节（Byte）：有符号（Signed）、无符号（Unsigned）
- 半字（Half-word）：有符号（Signed）、无符号（Unsigned）、分数（Fraction）
- 字（Word）：有符号（Signed）、无符号（Unsigned）、分数（Fraction）、浮点数（Floating-point）
- 48 位（48-bit）：有符号（Signed）、无符号（Unsigned）、分数（Fraction）
- 双字（Double-word）：有符号（Signed）、无符号（Unsigned）、分数（Fraction）



### 2.1 对齐要求（Alignment Requirements）
TriCore 架构对 地址（Address） 和 数据（Data） 的对齐要求有所不同：
- 地址变量：存入或加载到 地址寄存器（Address Register） 时，必须 按字（Word）对齐。
- 数据：无论数据大小如何,可以在 半字（Half-Word）边界 上对齐（除非有特殊说明）。这使得 DSP 应用 可以在 半字边界 上加载或存储 两个或四个 16 位数据元素，以支持 打包算术运算（Packed Arithmetic Operations）。

开发人员需要注意以下限制：
- LDMST、CMPSWAP.W、SWAPMSK.W 和 SWAP.W 指令 需要 按字（Word）对齐。
- 字节操作（LD.B、ST.B、LD.BU、ST.T） 可以 按字节（Byte）对齐。
- 所有外设空间访问 必须 自然对齐（Naturally Aligned，即根据访问大小对齐），但 双字（Double-Word）访问 可以 按字（Word）对齐。

**非外设空间对齐规则：**
| Access Type                     | Access Size          | Alignment of Address in Memory |
|---------------------------------|----------------------|---------------------------------|
| Load, Store Data Register       | Byte                 | Byte (1)                        |
|                                 | Half-Word            | 2 bytes (2)                     |
|                                 | Word                 | 2 bytes (2)                     |
|                                 | Double-Word          | 2 bytes (2)                     |
| Load, Store Address Register    | Word                 | 4 bytes (4)                     |
|                                 | Double-Word          | 4 bytes (4)                     |
| SWAP.W, LDMST                   | Word                 | 4 bytes (4)                     |
| CMPSWAP.W, SWAPMSK.W            | Word                 | 4 bytes (4)                     |
| ST.T                            | Byte                 | Byte (1)                        |
| Context Load/Store/Restore/Save | 16 x 32-bit registers| 64 bytes (64)                   |

<br/>

**外设空间对齐规则：**
| Access Type                     | Access Size          | Alignment of Address in Memory |
|---------------------------------|----------------------|---------------------------------|
| Load, Store Data Register       | Byte                 | Byte (1)                        |
|                                 | Half-Word            | 2 bytes (2)                     |
|                                 | Word                 | 4 bytes (4)                     |
|                                 | Double-Word          | 8 bytes (8)                     |
| Load, Store Address Register    | Word                 | 4 bytes (4)                     |
|                                 | Double-Word          | 8 bytes (8)                     |
| SWAP.W, LDMST, ST.T             | Word                 | 4 bytes (4)                     |
| CMPSWAP.W, SWAPMSK.W            | Word                 | 4 bytes (4)                     |
| Context Load/Store/Restore/Save | 16 x 32-bit registers| Not Permitted                   |

### 2.2 Byte Ordering
数据内存和 CPU 寄存器以小端格式存储，最低有效字节在低地址。

## 3 Memory Model

TriCore 处理器架构采用 32 位地址宽度，可访问 最多 4GB 内存。其地址空间被划分为 16 个区域或段（[0<sub>H</sub> - F<sub>H]</sub>），每个段大小为 256MB。地址的 高 4 位 用于选择特定的段，每个段的 **前 16KB 可以使用 绝对寻址（Absolute Addressing） 访问**。

**数据访问与地址计算**
许多数据访问使用 基地址寄存器（Base Address Register） 加上 位移量（Displacement） 计算得到地址。**位移量不能跨越段边界，否则会触发 MEM 陷阱（MEM Trap）**。这一限制确保可以 直接从基地址确定访问的段。

**物理内存属性（Physical Memory Attributes）**
段 [0<sub>H</sub> - 7<sub>H</sub>] 的物理内存属性取决于具体实现：如果 MMU（内存管理单元） 存在且启用，这些段被视为 虚拟地址，必须进行 地址转换。
如果 MMU 不存在，访问特性取决于具体实现。

**物理内存地址（Physical Memory Addresses）**
段 F<sub>H</sub> 保证用于 外设空间（Peripheral Space）：所有访问都是非推测性（Non-Speculative）。User-0 模式无法访问。
非推测性：
- 严格顺序性：所有操作按程序代码的顺序执行，不提前执行可能不需要的指令（如分支预测后的指令）。
- 无预测性：不基于历史行为或统计模型推测未来的操作路径（如跳过未确定的分支）。

核心特殊功能寄存器 CSFRs（Core Special Function Registers） 映射到 64KB 的内存空间。该 64KB 空间的基地址取决于具体实现。



## 4 信号量与原子操作
以下指令以 原子方式（Atomic Fashion） 读取和/或写入内存：
- LDMST（加载、修改、存储）
- SWAP.W（寄存器与内存交换）
- ST.T（存储位）
- CMPSWAP.W（条件交换）
- SWAPMSK.W（掩码交换）

LDMST 指令使用掩码寄存器（Mask Register），将 源寄存器（Source Register） 的选定位写入 内存字（Memory Word）。但 LDMST 不返回值，因此 不能用于二进制信号量（Binary Semaphore）的“测试并设置（Test and Set）”操作。

如果启用了内存保护（Memory Protection），则 LDMST、CMPSWAP.W、SWAPMSK.W、SWAP.W 或 ST.T 指令的有效地址 必须位于 同时具有读写权限的范围内。

CMPSWAP.W 指令：条件交换，即 仅在特定条件满足时，才将 源寄存器的值 交换到 内存字。
SWAPMSK.W 指令：通过掩码（Mask）交换 源寄存器与内存字的内容。


**原子指令的执行会强制完成 所有在其之前的内存访问**。确保任何缓冲状态（Buffered State） 在 原子操作执行前 被 正确写入内存。


## 5 寻址模式（Addressing Modes）
寻址模式允许 加载（Load） 和 存储（Store） 指令访问 简单数据元素，如：
- 记录（Records）
- 随机访问和顺序访问数组（Randomly and Sequentially Accessed Arrays）
- 堆栈（Stacks）
- 循环缓冲区（Circular Buffers）

这些数据元素的宽度可以是 8 位、16 位、32 位或 64 位。TriCore 架构支持 七种寻址模式。

| **寻址模式（Addressing Mode）** | **地址寄存器使用（Address Register Use）** |
|--------------------------------|--------------------------------|
| **绝对寻址（Absolute）** | 无（None） |
| **基地址 + 短偏移（Base + Short Offset）** | 地址寄存器（Address Register） |
| **基地址 + 长偏移（Base + Long Offset）** | 地址寄存器（Address Register） |
| **前增量（Pre-increment）** | 地址寄存器（Address Register） |
| **后增量（Post-increment）** | 地址寄存器（Address Register） |
| **循环寻址（Circular）** | 地址寄存器对（Address Register Pair） |
| **位反转寻址（Bit-reverse）** | 地址寄存器对（Address Register Pair） |


这些寻址模式支持高效编译 C/C++ 代码，便捷访问外设寄存器，优化 DSP 数据结构（如 滤波器的循环缓冲区 和 FFT 的位反转索引）。



指令格式（Instruction Formats）
指令格式提供尽可能多的地址位，用于 绝对寻址（Absolute Addressing）。尽可能大的偏移范围，用于 基地址 + 偏移（Base + Offset Addressing）。

在某些情况下，地址寄存器（Address Register） 既可以作为 加载目标（Target of a Load），又可以作为 寻址模式更新的一部分（Update Associated with a Particular Addressing Mode）。

### 5.1 绝对寻址（Absolute Addressing）
绝对寻址 适用于 引用 I/O 外设寄存器 和 全局数据。

绝对寻址使用指令指定的 18 位常量作为内存地址。完整的 32 位地址由 18 位常量的最高 4 位移动到 32 位地址的最高位生成。其余位填充为零，确保地址格式正确。

例如：
```
LD.W D[a], off18 (ABS)(Absolute Addressing Mode)

对应执行内容：
EA = {off18[17:14], 14b'0, off18[13:0]};
D[a] = M(EA, word);
```

### 5.2 基地址 + 偏移寻址（Base + Offset Addressing）
基地址 + 偏移寻址 适用于：
- 引用记录元素（Record Elements）
- 访问局部变量（Local Variables）（使用栈指针（SP，Stack Pointer）作为基地址）
- 访问静态数据（Static Data）（使用地址寄存器（Address Register）指向静态数据区域）

完整的有效地址（Effective Address） 由 地址寄存器（Address Register）和符号扩展的 10 位偏移量（Sign-Extended 10-bit Offset） 相加得到。

部分内存操作 提供 基地址 + 长偏移（Base + Long Offset） 寻址模式。偏移量为 16 位符号扩展值（Sign-Extended 16-bit Value）。允许使用两条指令序列 访问 内存中的任何位置。

例如：
```
LD.W D[a], A[b], off10 (BO)(Base + Short Offset Addressing Mode)
对应执行内容：
EA = A[b] + sign_ext(off10);
D[a] = M(EA, word);


LD.W D[a], A[b], off16 (BOL)(Base + Long Offset Addressing Mode)
对应执行内容：
EA = A[b] + sign_ext(off16);
D[a] = M(EA, word);

```

### 5.3 预增量和预减量寻址（Pre-Increment and Pre-Decrement Addressing）
预增量（Pre-Increment） 和 预减量（Pre-Decrement） 寻址模式可用于：向上增长的栈（Upward-Growing Stack）进行数据推入（Push）。向下增长的栈（Downward-Growing Stack）进行 数据推入（Push）（预减量寻址通过负偏移量（Negative Offset） 实现）等。

预增量寻址模式 使用 地址寄存器 + 偏移量 作为有效地址（Effective Address），写回地址寄存器的值（Value Written Back into the Address Register）

例如：
```
LD.W D[a], A[b], off10 (BO)(Pre-increment Addressing Mode)
实际执行内容：
EA = A[b] + sign_ext(off10);
D[a] = M(EA, word);
A[b] = EA;
```


### 5.4 后增量和后减量寻址（Post-Increment and Post-Decrement Addressing）
后增量（Post-Increment） 和 后减量（Post-Decrement） 寻址模式可用于：
- 顺序访问数组（Sequential Access of Arrays）：后增量 用于 向前访问（Forward Access），后减量 用于 向后访问（Backward Access）。
- 栈操作（Stack Operations）：后增量 用于 从向上增长的栈弹出数据（Pop from an Upward-Growing Stack）。后减量 用于 从向下增长的栈弹出数据（Pop from a Downward-Growing Stack）。

后增量寻址模式 使用 地址寄存器的当前值 作为 有效地址。然后通过添加符号扩展的 10 位偏移量到原始值，更新地址寄存器。

例如：
```
LD.W D[a], A[b], off10 (BO)(Post-increment Addressing Mode)
实际执行内容：
EA = A[b];
D[a] = M(EA, word);
A[b] = EA + sign_ext(off10);

```

### 5.5 循环寻址（Circular Addressing）
循环寻址主要用于访问循环缓冲区（Circular Buffers），特别是在 滤波计算（Filter Calculations） 中。

循环寻址模式使用地址寄存器对（Address Register Pair） 来存储所需状态：

偶数寄存器（Aeven） 始终存储 基地址（B）。奇数寄存器（Aodd）的高 16 位 存储 缓冲区大小（L），低 16 位 存储 缓冲区索引（I）。

有效地址（Effective Address） 计算方式：Effective Address = B + I 
缓冲区的内存范围： Memory Range = [B, B + L - 1]

使用以下算法对索引进行后加（post-incremented）：
```


tmp = I + sign_ext(offset10);
if (tmp < 0)
  I = tmp + L;
else if (tmp >= L)
  I = tmp - L;
else
  I = tmp;
```
在 TriCore 处理器 的 循环寻址（Circular Addressing） 模式中，指令字（Instruction Word） 指定 10 位偏移量（Offset），单位为 字节。偏移量可以是正数或负数，用于调整索引位置。
只要偏移量的绝对值小于缓冲区大小，就能保证 正确的环绕行为（Wrap Around）。

环绕行为示例：
假设一个 循环缓冲区 包含 25 个 16 位数据，当前索引（I）= 48：
- 使用 偏移量 2（每个值 2 字节）。新索引值 = 0（48 + 2 = 50，环绕到 0）。
- 如果偏移量为 4：新索引值 = 2（48 + 4 = 52，环绕到 2）。

如果当前索引为 4，偏移量为 -8：新索引值 = 46（4 - 8 + 50 = 46）。

**循环缓冲区对齐规则：**
- 缓冲区起始地址必须对齐到 64 位边界。
- 缓冲区长度必须是数据大小的整数倍：
- 如果使用 LD.W（加载 32 位数据），缓冲区长度必须是 4 字节的整数倍。
- 如果使用 LD.D（加载 64 位数据），缓冲区长度必须是 8 字节的整数倍。

如果 不满足对齐要求，触发对齐陷阱（ALN Trap）。如果索引（I）≥ 缓冲区长度（L），也会触发 ALN Trap。

**循环寻址模式不能用于外设空间（Peripheral Space）**。如果尝试访问外设空间，会触发 MEM 陷阱（MEM Trap）。

指令示例：
```
LD.W D[a], P[b], off10 (BO)(Circular Addressing Mode)
实际执行动作： （P[b]为 A[b]、A[b+1]寄存器对构成的64位地址寄存器）
index = zero_ext(A[b+1][15:0]);
length = zero_ext(A[b+1][31:16]);
EA0 = A[b] + index;
EA2 = A[b] + (index + 2% length);
D[a] = {M(EA2, halfword), M(EA0, halfword)};
new_index = index + sign_ext(off10);
new_index = new_index < 0 ? new_index + length : new_index % length;
A[b+1] = {length[15:0], new_index[15:0]};
```

### 5.6 位反转寻址（Bit-Reverse Addressing）
位反转寻址 主要用于 访问 FFT（快速傅里叶变换）算法中的数组。在 FFT 的常见实现 中，最终计算结果通常存储为 位反转顺序（Bit-Reversed Order）

位反转寻址使用 地址寄存器对（Address Register Pair） 来存储所需状态：
- 偶数寄存器（Aeven）：存储 数组的基地址（B）。
- 奇数寄存器（Aodd）：低 16 位 存储 数组索引（I），高 16 位 存储 修正值（M），用于 每次访问后更新索引 I。

有效地址（Effective Address） 计算方式：Effective Address = B + I

索引 I 的更新方式： I_new = reverse[reverse(I) + reverse(M)]，reverse(I) 函数 交换 位 n 和位 (15 - n)，其中 n = 0, ..., 7。

位反转寻址示例：
假设一个 1024 点的 FFT 使用 16 位数据：缓冲区大小（Buffer Size）= 2048 字节。
使用位反转索引遍历数组，得到的字节索引序列如下：0, 1024, 512, 1536, ...。该序列可以通过初始化 I = 0 和 M = 0x0400H 获得。

M的要求值为缓冲区大小/2，其中缓冲区大小以字节为单位给出。

指令示例：
```


LD.W D[a], P[b] (BO)(Bit-reverse Addressing Mode)
实际执行内容：
index = zero_ext(A[b+1][15:0]);
incr = zero_ext(A[b+1][31:16]);
EA = A[b] + index;
D[a] = M(EA, word);
new_index = reverse16(reverse16(index) + reverse16(incr));
A[b+1] = {incr[15:0], new_index[15:0]};
```


### 5.7 合成寻址模式（Synthesized Addressing Modes）
某些硬件不直接支持的寻址模式可以通过短指令序列（Short Instruction Sequences）合成，以实现灵活的数据访问。

**索引寻址（Indexed Addressing）**：
索引寻址模式 可以使用 ADDSC.A（Add Scaled Index to Address） 指令合成。ADDSC.A 指令 通过 将缩放后的数据寄存器值 加到 地址寄存器 来计算地址。

缩放因子（Scale Factor） 可以是 1、2、4 或 8，用于索引：
- 字节数组（Bytes）
- 半字数组（Half-Words）
- 字数组（Words）
- 双字数组（Double-Words）

指令示例：
```


ADDSC.A A[c], A[b], D[a], n (RR)

实际执行动作：
A[c] = A[b] + (D[a] << n);

```

**位索引寻址（Bit Indexed Addressing）**
位索引寻址适用于访问索引位数组（Indexed Bit Arrays），在TriCore处理器中，使用 ADDSC.AT 指令进行索引计算。

位索引寻址的计算方式
ADDSC.AT 指令通过将索引值右移 3 位（除以8）并加到地址寄存器来计算地址，结果地址的低两位被清零，确保访问包含目标位的字（Word）。

之后，加载包含目标位的字（Word）。使用 EXTR.U 指令 提取 目标位。

位提取示例:
```


ADDSC.AT A0, A1, D2, 1  // 计算位索引地址
LD.W D3, [A0]           // 加载包含目标位的字
EXTR.U D4, D3, #5, #1   // 提取第 5 位
```

位存储示例：:
ADDSC.AT 指令 计算 目标位地址。
LDMST（Load/Modify/Store）指令 用于 存储单个位或位字段。
```


ADDSC.AT A0, A1, D2, 1  // 计算目标位地址
IMASK E8, D4, #5, #1    // 生成掩码和数据
LDMST [A0], E8          // 存储目标位
```


**PC 相对寻址（PC-Relative Addressing）**
PC 相对寻址是 分支（Branch）和 调用（Call） 的常规模式。然而，TriCore 架构不支持直接的 PC 相对数据寻址，原因如下：
- 指令存储（Instruction Memory） 和 数据存储（Data Memory） 在芯片上是分开的。
- 直接访问程序存储器中的数据 代价较高，因此不支持 PC 相对寻址。

当需要 PC 相对寻址数据 时：将附近代码标签的地址存入地址寄存器（Address Register）。使用基地址 + 偏移（Base + Offset）模式 访问数据。一旦基地址寄存器被加载，它可以用于访问 附近的其他 PC 相对数据项。

代码地址可以通过多种方式加载到地址寄存器。
在静态链接（Statically Linked）的情况下，代码标签的绝对地址已知，可以使用 LEA（Load Effective Address）指令 加载。
如果代码是 动态加载 的，或者 由位置无关的代码片段（Position-Independent Code）组成，而没有重定位链接器（Relocating Linker） 的支持：使用 JL（Jump and Link）指令 加载代码地址：JL 指令执行跳转并链接到下一条指令，将 该指令的地址存入返回地址（RA）寄存器 A[11]。（在执行 JL 之前，需要 将当前函数的返回地址保存，以确保正确的返回路径。）