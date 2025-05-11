
# 1 V7-M 架构扩展：

## 1.1 DSP 扩展（DSP extension）
该可选扩展为 Armv7-M Thumb 指令集 添加了 Arm 数字信号处理（DSP）指令。这些指令包括饱和运算（saturating）和无符号单指令多数据（SIMD）指令。 如果 Armv7-M 处理器包含 DSP 扩展，那么该实现被称为 Armv7E-M，并且在指令详情章节中，会明确标注这些额外的指令属于 Armv7E-M 指令。

## 1.2 浮点扩展（Floating-point Extension）
该可选扩展为 Armv7-M Thumb 指令集 添加了 浮点运算指令，并且有两个不同的版本：
- FPv4-SP: 这是 VFPv4-D16 扩展的单精度实现，该扩展定义于 Armv7-A 和 Armv7-R 架构配置文件。
- FPv5: 该扩展额外支持 双精度计算 并提供了更多的指令。

**浮点扩展的完整分类（Floating-point Extension full characterizations）如下**：

| 扩展版本  | 仅支持单精度 | 支持单精度和双精度 |
|----------|------------------|----------------------|
| **FPv4-SP** | FPv4-SP-D16-M | 不适用 |
| **FPv5** | FPv5-SP-D16-M | FPv5-D16-M |


# 2 Application Level Programmers’ Model

## 2.1  About the application level programmers’ model

操作系统同时支持 特权模式 和 非特权模式，但应用程序通常运行在非特权模式下。

应用程序在 非特权模式（Unprivileged Mode） 运行时：
- 允许操作系统为其分配 系统资源，这些资源可以是 专用（Private） 或 共享（Shared） 资源。
- 提供一定程度的 进程和任务隔离保护，从而帮助操作系统免受 故障应用程序 的影响。

处于 非特权模式 运行意味着处理器运行在 线程模式（Thread Mode）。

### 2.1.1 Interaction with the system level architecture

