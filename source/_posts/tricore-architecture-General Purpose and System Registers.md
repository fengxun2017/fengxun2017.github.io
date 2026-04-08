---
title: tricore architecture - General Purpose and System Registers
date: 2025/5/27
categories: 
- Tricore
---

<center>
General Purpose and System Registers
</center>

<!--more-->

***

核心寄存器（Core Register）分为两类：通用寄存器（GPRs）和核心特殊功能寄存器（Core Special Function Registers, CSFRs）。

GPRs包括16个通用数据寄存器和16个通用地址寄存器。
CSFRs用于控制核心操作并提供核心状态信息。

寄存器分类
- 通用寄存器（General Purpose Registers）
- 系统寄存器（System Registers）：PSW, PC, PCXI
- 栈管理寄存器（Stack Management Registers）：A[10] 和 ISP
- SYSCON 和 CPU_ID 寄存器
- 异常处理寄存器（Trap Registers）
- 上下文管理寄存器（Context Management Registers）
- 内存保护寄存器（Memory Protection Registers）
- 内存管理寄存器（Memory Management Registers）
- 调试寄存器（Debug Registers）
- 浮点寄存器（Floating Point Registers）
- 与核心相关的特殊功能寄存器（Special Function Registers associated with the core）


**ENDINIT 保护（ENDINIT Protection）**
该架构支持初始化状态（initialisation state），在进入运行状态（operational state）之前，可以对所有核心特殊功能寄存器（CSFRs）进行修改。
在初始化状态时，所有 CSFRs 可以 通过 MTCR 指令 修改。在运行状态下，只有部分 CSFRs 可被修改，其他功能保持不变。

仅可在初始化状态下修改的 CSFRs 被称为“ENDINIT 保护寄存器（ENDINIT protected）。

初始化状态与运行状态的转换由系统实现决定，这种机制增加了对关键 CSFRs的保护，确保它们只能在初始化状态下修改。

以下寄存器受 ENDINIT 保护：BTV, BIV, ISP, PMA0, PMA1, PMA2, PCON0, DCON0, SEGEN

架构还提供了面向安全的 ENDINIT 保护，以下寄存器属于 SAFETY_ENDINIT 保护范围：SMACON, SYSCON, COMPAT, TPS_EXTIM_ENTRY_LVAL, TPS_EXTIM_EXIT_LVAL


## 1 通用寄存器（General Purpose Registers，GPRs）
通用寄存器（GPRs）均分为两类：
- 16个数据寄存器（DGPRs）：D[0] 到 D[15]
- 16个地址寄存器（AGPRs）：A[0] 到 A[15]

数据寄存器和地址寄存器的分离有助于提高处理效率，使算术运算和内存操作可以并行执行。 多个指令支持数据寄存器与地址寄存器之间的信息交换，例如用于创建或计算表索引。

两个连续的偶-奇数据寄存器可以拼接形成八个扩展大小寄存器， 以支持 64 位数据：E[0], E[2], E[4], E[6], E[8], E[10], E[12], E[14]（用于支持 64 位数据）
地址寄存器也可以同样拼接：P[0], P[2], P[4], P[6], P[8], P[10], P[12], P[14]

**特殊寄存器:**
- 系统全局寄存器：A[0], A[1], A[8], A[9]（这些寄存器的内容在调用、异常或中断时不会被保存或恢复）
- 栈指针（Stack Pointer，SP）：A[10]（参见“栈管理寄存器”）
- 返回地址（Return Address，RA）：A[11]（用于存储调用和链接跳转的返回地址，以及中断和异常的返回程序计数器 PC）

指令对寄存器的使用:
- 32位指令可以无限制地使用GPRs。
- 许多16位指令会隐式使用 A[15] 作为地址寄存器，D[15] 作为数据寄存器，这种隐式使用有助于将指令编码压缩至16位。


浮点运算:没有单独的浮点寄存器，数据寄存器用于执行浮点运算。浮点数据可通过快速上下文切换自动保存和恢复。



## 2 程序状态信息寄存器（Program State Information Registers）

程序状态信息寄存器用于存储和恢复任务的上下文，包括：
- PC（程序计数器）：存储当前执行指令的地址，仅在核心停止时可写入。
- PSW（程序状态字）：记录任务的架构状态，包括保护机制、访问权限、调用深度等。
- PCXI（先前上下文信息）：用于任务切换时保存上一个任务的状态。


### 2.1 程序计数器（PC）

32 位，存储当前执行的指令地址。只能在核心暂停状态下修改，否则不会生效。


### 2.2 程序状态字寄存器（PSW）
程序状态字寄存器（PSW）是一个32位寄存器，它存储了任务特定的架构状态，这些状态信息不会包含在通用寄存器（GPRs）中。 PSW包括以下参数：
- USB：程序状态字（PSW）的最高8位被定义为用户状态位（User Status Bits）。这些位可在执行指令时被设置或清除，通常用于记录运算结果的状态。 此外，某些指令可以使用这些位来影响其执行逻辑。例如：
  - ADDX（扩展加法）和ADDC（带进位加法）指令使用第31位来记录加法运算中的进位信息。
  - ADDC 指令在执行时，会读取该进位位的先前值，并将其影响到新的计算结果。

- 保护寄存器集合（PRS）：选择当前活动的数据与代码内存保护寄存器集。内存保护寄存器值控制当前进程内的加载、存储和指令获取。

- 安全任务标识符（Safety Task Identifier）：用于区分安全任务和普通任务。该标识符可以决定在内部总线上使用哪种 MASTER TAG ID，从而影响读/写访问权限。
  
- I/O 特权级别（IO）：I/O特权级别控制
  - 00b：User-0模式，禁止访问外设空间（触发PSE/MPP陷阱），无权启用/禁用中断
  - 01b：User-1模式，允许访问非保护外设（串行I/O端口、定时器等），可禁用中断（默认行为可通过系统控制寄存器覆盖）
  - 10b：监管模式（Supervisor），允许访问所有外设和核心寄存器，可禁用中断
  - 11b：保留值


- 中断栈标志（IS）：中断栈控制
  - 0：用户栈，中断触发时从ISP寄存器加载栈指针
  - 1：共享全局栈，中断触发时直接使用当前栈指针


- 全局寄存器写入权限（GW）：该字段决定当前执行线程是否有权限修改全局地址寄存器。大多数任务和中断服务程序（ISR）将全局地址寄存器视为只读寄存器，用于指向全局字面量池和关键数据结构。但是，某些任务或 ISR 可以被指定为特定全局地址寄存器的“所有者”，并且允许修改该寄存器。系统设计者必须决定哪些全局地址变量在高频访问或关键代码中足够重要，以便将它们分配到全局地址寄存器。（A[0]通常被保留为短格式加载和存储的基址寄存器。A[1]也被保留给编译器使用。A[8] 和 A[9]编译器不会使用这两个寄存器，它们可以用于存储关键系统地址变量。）
  - 1 ：允许希望全局地址寄存器 A[0], A[1], A[8], A[9]
  - 0 ：禁止希望全局地址寄存器 A[0], A[1], A[8], A[9]


- 调用深度计数器（CDC）：调用深度计数使能
  - 0：临时禁用计数，执行下一条CALL指令后自动启用。（关闭后，第一次 CALL 不会增加 CDC，但不会永久禁用，只是暂时忽略当前调用。下一次 CALL 指令会自动恢复 CDC 计数，确保后续函数调用正确记录深度）
  - 1：启用计数，若CDC=1111111b则强制禁用计数


- 调用深度计数启用字段（CDE）：调用深度计数器，CALL指令递增，RETURN指令递减，计数溢出时触发CDO陷阱。
  特殊模式：
  - 1111110b：每次CALL触发陷阱（调用跟踪模式）
  - 1111111b：禁用调用深度计数


**访问特权级别控制（I/O Privilege）**
软件管理任务（SMT，Software Managed Tasks），通过实时内核（RTOS）或操作系统创建，由调度软件进行任务调度。中断服务程序（ISR，Interrupt Service Routine）由硬件直接触发，处理外部或内部中断。
SMT 有时被称为“用户任务”（User Tasks），默认在用户模式（User Mode）下执行。
任务模式分类：每个任务根据其功能被分配不同的运行模式
- User-0 模式：适用于不需要访问外围设备的任务。无法启用或禁用中断。
- User-1 模式：适用于访问常规、未受保护的外设的任务，例如：串行端口的读写，定时器的读取，大多数 I/O 状态寄存器。允许任务禁用中断（但其默认行为可由系统控制寄存器修改）。
- Supervisor 模式：允许访问所有系统寄存器和外设。任务可以启用或禁用中断。

**任务上下文（Task Context）**
每个任务都有一组状态元素，统称为任务上下文，任务上下文包括：
- CPU 通用寄存器（General Purpose Registers）
- 程序计数器（PC，Program Counter）
- 程序状态信息（PCXI 和 PSW）

这些信息是处理器定义任务状态并保持任务执行所需的关键数据。


### 2.3 先前上下文信息和指针寄存器（Previous Context Information and Pointer Register，PCXI）
先前上下文信息寄存器（PCXI）存储与先前执行上下文相关的链接信息，用于支持中断和自动上下文切换。该寄存器是任务状态信息的一部分，其中先前上下文指针（PCX） 保存前一个任务的上下文保存区（CSA）的地址。

PCXI 寄存器字段包含这些字段：
- PCPN（存储被中断任务的优先级级别）
- PIE（存储被中断任务的中断使能位（ICR.IE）的状态。）
- UL（标识保存的上下文类型：上层或下层）
  - 0：下层上下文（Lower Context）
  - 1：上层上下文（Upper Context）
- PCXS：存储PCX 段地址，该字段与 PCXO（先前上下文指针偏移） 结合使用。
- PCXO（先前上下文指针偏移字段）

PCXO 和 PCXS 共同形成 PCX 指针，用于指向先前任务的 CSA（上下文保存区）。

PCXI 寄存器用于维护任务的上下文信息，特别是在任务切换、异常处理和中断处理时。它确保处理器可以正确恢复先前任务的运行状态，避免错误的上下文切换导致系统故障。


## 3 堆栈管理寄存器（Stack Management Registers）
tricore架构的堆栈管理支持用户堆栈和中断堆栈，并使用地址寄存器 A[10]、中断堆栈指针（ISP）以及 PSW 寄存器中的一个位来管理堆栈。

A[10] 作为堆栈指针（Stack Pointer，SP）使用。当一个任务被创建时，实时操作系统（RTOS）通常会初始化该寄存器的内容，以便为任务分配私有堆栈区域。

ISP 的作用是防止中断服务程序（ISR） 访问任务的私有堆栈，避免影响任务的上下文。


中断时的堆栈切换:
- 如果被中断的任务使用的是私有堆栈（PSW.IS == 0）：任务的上层上下文被保存。A[10]（SP）被加载为当前 ISP 的值。
- 如果被中断的任务已经在使用中断堆栈（PSW.IS == 1）：A[10]（SP）不会重新加载，ISR 继续使用中断堆栈，而不会切换到新的栈指针。

通常在系统初始化时，只需要初始化 ISP 一次。

在某些应用场景下，ISP 可能会在执行过程中动态修改。架构并未禁止 ISR 或系统服务程序使用私有堆栈，是否使用取决于系统的设计。


## 4 系统控制寄存器（SYSCON）

系统配置寄存器 SYSCON 提供以下功能：

- 用于 时序保护系统（Temporal Protection System） 的使能位。
- 用于 内存保护系统（Memory Protection System） 的使能位：在将使能位设为 1 之前，必须先初始化保护寄存器组。

- 用于定义 中断处理程序（Interrupt Handlers） 中 PSW.S 位的初始状态的位。
- 用于定义 陷阱处理程序（Trap Handlers） 中 PSW.S 位的初始状态的位。
- User-1 IO 模式外设访问的使能位：允许 User-1 模式任务 像 Supervisor 模式 一样访问外设，使 User-1 模式 能够访问 所有外设寄存器。

- User-1 IO 模式中断使能/禁用能力的禁用位：禁止 在 User-1 IO 模式下执行 User-1 指令（包含中断使能/关闭指令）。
  
- 启动暂停状态（Boot Halt Status） 和 启动释放（Release Bit）:在复位后，CPU 可能会被立即置于暂停状态，此时 BHALT 位 被设为 1。CPU 将保持暂停状态，直到该位被写入 0。当 BHALT 从 1 写入 0 时，CPU 将开始执行，起始地址由 程序计数器（PC）寄存器 定义。如果尝试将该位写入 1，则该写入将被忽略。


- 空闲上下文列表耗尽（Free Context List Depletion Condition， FCDSF） 状态指示器：该粘滞位表示 自上次软件清除该位以来，是否发生了 FCD（空闲上下文列表耗尽）陷阱。

注意：**此寄存器受 SAFETY_ENDINIT 保护，但 FCDSF 位除外**。



## 5 中断寄存器（Interrupt Registers）
在 TriCore 架构 中，典型的 服务请求控制寄存器（Service Request Control Register） 包含以下控制位：
- 启用或禁用中断请求
- 分配中断的优先级
- 将中断请求指向特定的服务提供者

核心特殊功能寄存器（CSFR） 负责控制中断功能，详细信息见“中断系统”

## 6 内存保护寄存器（Memory Protection Registers）
每个架构实现的内存保护寄存器组数量不同，最多可支持 8 组。

每组包括 数据集（Data Set）和代码集（Code Set）。

每个寄存器组由多个 范围寄存器（Range Table Entries） 组成：

每个 范围表项 包含一个段保护寄存器对和一个模式寄存器的位字段，寄存器对用于定义内存范围的上下边界地址。

核心特殊功能寄存器（CSFR） 负责控制 内存保护寄存器，详细信息见“内存保护系统”。

## 7 陷阱寄存器（Trap Registers）
核心特殊功能寄存器（CSFR） 负责控制 陷阱寄存器，详细信息见“陷阱系统”。


## 8 核心调试控制器寄存器（Core Debug Controller Registers）
TriCore 处理器中的调试支持寄存器 详细信息见“核心调试控制器”

## 9 浮点寄存器（Floating Point Registers）
TriCore 处理器可选的浮点运算单元（FPU）寄存器 详细信息见“FPU_TRAP_CON”。

## 10 访问核心特殊功能寄存器（CSFRs）

核心特殊功能寄存器（CSFR） 通过以下指令访问：

- MFCR（Move From Core Register） → 读取 CSFR。
- MTCR（Move To Core Register） → 写入 CSFR。

软件通常不需要频繁更新 CSFR。因此
- 硬件实现不需要特殊结构来避免 CSFR 更新的竞争风险。
- 软件实现中，在 MTCR 更新 CSFR 后，需插入 ISYNC 指令，确保 CSFR 更新的影响正确被后续指令识别。

其他访问规则：
- MTCR 访问未定义寄存器时，不会产生任何影响。
- MFCR 访问未定义寄存器时，将返回未定义数据。