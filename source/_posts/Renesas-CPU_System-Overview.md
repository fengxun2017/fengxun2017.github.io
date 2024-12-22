---
title: Renesas-RH850  CPU System - Overview
date: 2024/12/2
categories: 
- Renesas
---

<center>

Renesas学习笔记
CPU System: Overview

</center>

<!-- more -->

***

CPU系统由以下模块组成：

#### CPU0 (PE0), CPU1 (PE1), CPU2 (PE2), CPU3 (PE3)：
每个`PE`包含一个主CPU，还包括一个检查核，以确保安全。

#### ICUMHA：
ICUMHA （Intelligent Cryptographic Unit/Master）是一种硬件安全模块。ICUMHA可以通过Option Byte设置激活。

#### Local RAM：
每个PE都有一个高速可访问的本地RAM。


#### Cluster RAM：
集群RAM是所有pe共享的大容量片上RAM。该RAM的某些区域起 retention RAM 的作用。

#### Code Flash:
包括用于程序存储的大量代码闪存。所有`PE`和`ICUMHA`共享代码flash，它们通过本地/全局flash总线相互连接。

#### Emulation RAM:
这是模拟代码 flash 的 RAM。这些程序可以通过外部工具的控制来替换而无需重写代码 flash。
RH850/U2A-EVA 和 RH850/U2A6 支持

#### Data Flash:
这是可由 CPU 重写的闪存。

#### Instrumentation RAM：
仪表 RAM 是作为 RH850 第 2 代调试功能并入的数据保存 RAM 区域，用于工具输出。仅在 RH850/U2A-EVA 中受支持。

####  P-Bus and H-Bus and I-Bus：
外围IP连接在P总线、H总线和I总线上。
对于U2A-EVA、U2A16、U2A8：
- P总线由10个组组成，P总线组0到9。 
- H总线由4组组成，H总线组0到3。 
- I总线由3组组成，I总线组0到2。

对于U2A6： 
- P总线由10组组成，P总线组0到9。
- H总线由2组组成，H总线组1和3。
- I总线由3组组成，I总线组0到2。

#### INTC1, INTC2:
INTC1 是每个 PE 独享的中断控制器。
INTC2 是所有 PE 可以共享的公共中断控制器，它可以通过寄存器设置中断请求的目标 PE。

#### DMA:
DMA 包括两个DMA传输模块，sDMAC和DTS。DTS包含用于安全保证的检查逻辑。

