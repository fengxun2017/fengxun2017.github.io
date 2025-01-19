---
title: Renesas-RH850  CPU System -  Inter-CPU Overview
date: 2025/1/8
categories: 
- Renesas
---

<center>

Renesas学习笔记
CPU System :  Inter-CPU Overview

</center>

<!-- more -->

***

本节包括屏障同步（BARR）、处理器间中断（IPIR）和时间保护计时器（TPTM）。本节的第一部分描述了该产品的特定属性，如单元数量、寄存器基地址等。其余部分展示了功能和寄存器。

##### 1 Features

###### 1.1 单元数量

| 产品名称        | BARR 单元数量 | 每个CPU的 IPIR通道数量 | IPIR 单元数量 | TPTM 单元数量 | 每个 CPU 的 TPTM 通道数量 |
| -------------- | ------------ | ------------ | ------------ | ------------ | ------------------ | 
| RH850/U2A-EVA  | 1            | 4         | 1            | 1         | 2 interval timers <br>1 free-run timer <br> 2 Up timers                   | 
| RH850/U2A16    | 1            | 4         | 1            | 1         | 2 interval timers <br>1 free-run timer <br> 2 Up timers                         | 
| RH850/U2A8     | 1            | 4         | 1            | 1         | 2 interval timers <br>1 free-run timer <br> 2 Up timers                         | 
| RH850/U2A6     | 1            | 4         | 1            | 1         | 2 interval timers <br>1 free-run timer <br> 2 Up timers                         | 

###### 1.2 寄存器基地址
BARR、IPIR 和 TPTM 的基地址列于下表中。BARR、IPIR 和 TPTM 寄存器地址作为这些基地址的偏移量给出。

| 基地址名称    | 基地址         | 总线组          |
| ----------- | ------------- | ------------- |
| <BARR_base> | FFFB 8000H    | I-总线组 1     |
| <IPIR_base> | FFFB 9000H    | I-总线组 0     |
| <TPTM_base> | FFFB B000H    | I-总线组 2     |

###### 1.3 时钟供给
时钟供给列于下表中。

| 单元名称 | 单元时钟名称 | 供给时钟名称  |
| ------- | ----------- | ---------- |
| BARR    | cpu_clk     | CLK_CPU    |
| IPIR    | cpu_clk     | CLK_CPU    |
| TPTM    | cpu_clk     | CLK_CPU    |

###### 1.4 中断请求
中断请求列于下表中。

| 中断符号名称 | 单元 | 中断信号描述         | 中断编号  | sDMA 触发编号 | DTS 触发编号 |
| ---------- | ---- | ------------------ | ------ | -------- | --------- |
| INTIPIR0   | ipir_int_ch0 | IPIR CH0 中断 | EIINT0 | —       | —         |
| INTIPIR1   | ipir_int_ch1 | IPIR CH1 中断 | EIINT1 | —       | —         |
| INTIPIR2   | ipir_int_ch2 | IPIR CH2 中断 | EIINT2 | —       | —         |
| INTIPIR3   | ipir_int_ch3 | IPIR CH3 中断 | EIINT3 | —       | —         |
| FEINT 或 INTTPTM<sup>*1</sup> | TPTM_IRQ | TPTM 间隔中断 | FEINT 或 EIINT31<sup>*1</sup> | — | — |
| INTTPTMU00 | INTTPTMU00 | TPTM 计时器上溢中断（比较值 0，针对 PE0） | EIINT209 | — | Group3-26 |
| INTTPTMU01 | INTTPTMU01 | TPTM 计时器上溢中断（比较值 1，针对 PE0） | EIINT210 | — | Group3-27 |
| INTTPTMU02 | INTTPTMU02 | TPTM 计时器上溢中断（比较值 2，针对 PE0） | EIINT211 | — | —         |
| INTTPTMU03 | INTTPTMU03 | TPTM 计时器上溢中断（比较值 3，针对 PE0） | EIINT212 | — | —         |
| INTTPTMU10 | INTTPTMU10 | TPTM 计时器上溢中断（比较值 0，针对 PE1） | EIINT213 | — | Group3-28 |
| INTTPTMU11 | INTTPTMU11 | TPTM 计时器上溢中断（比较值 1，针对 PE1） | EIINT214 | — | Group3-29 |
| INTTPTMU12 | INTTPTMU12 | TPTM 计时器上溢中断（比较值 2，针对 PE1） | EIINT215 | — | —         |
| INTTPTMU13 | INTTPTMU13 | TPTM 计时器上溢中断（比较值 3，针对 PE1） | EIINT216 | — | —         |
| INTTPTMU20 | INTTPTMU20 | TPTM 计时器上溢中断（比较值 0，针对 PE2） | EIINT217 | — | Group3-30 |
| INTTPTMU21 | INTTPTMU21 | TPTM 计时器上溢中断（比较值 1，针对 PE2） | EIINT218 | — | Group3-31 |
| INTTPTMU22 | INTTPTMU22 | TPTM 计时器上溢中断（比较值 2，针对 PE2） | EIINT219 | — | —         |
| INTTPTMU23 | INTTPTMU23 | TPTM 计时器上溢中断（比较值 3，针对 PE2） | EIINT220 | — | —         |
| INTTPTMU30 | INTTPTMU30 | TPTM 计时器上溢中断（比较值 0，针对 PE3） | EIINT221 | — | Group3-32 |
| INTTPTMU31 | INTTPTMU31 | TPTM 计时器上溢中断（比较值 1，针对 PE3） | EIINT222 | — | Group3-33 |
| INTTPTMU32 | INTTPTMU32 | TPTM 计时器上溢中断（比较值 2，针对 PE3） | EIINT223 | — | —         |
| INTTPTMU33 | INTTPTMU33 | TPTM 计时器上溢中断（比较值 3，针对 PE3） | EIINT224 | — | —         |

注意 *1：此中断可以选择用作 FEINT 或 EIINT31。请参见第 6.3.15 节，TPTMSEL — TPTM 中断 FE EI 选择寄存器。


###### 1.5 重置来源
以下显示了重置来源。这些模块由以下重置来源初始化。

| 单元名称 | 寄存器名称    | Power On Reset          | System Reset 1  | System Reset 2  | Application Reset  | DeepSTOP Reset  | Module Reset |
| ------ | ----------- | -------------- | ------ | ------- | ------- | ------ |-------|
| BARR   | 所有寄存器      | √              | √       | √        |   √     |—|—|
| IPIR   | 所有寄存器      | √              | √       | √        |   √     |—|—|
| TPTM   | 所有寄存器      | √              | √       | √        |   √     |—|—|

###### 1.6 外部输入/输出信号
此模块没有外部输入/输出信号。


##### 2 处理器元标识符 (PEID)
PEID，即处理器元的 ID 号码，可以从 PEID 寄存器中读取。通过参考 PEID，可以知道哪个 CPU 核心执行特定的程序。以下显示了此产品的 PEID。


| CPU 核心      | PEID  |
| ------------- | ----- |
| CPU0 (PE0)    | 000B  |
| CPU1 (PE1)    | 001B  |
| CPU2 (PE2)    | 010B  |
| CPU3 (PE3)    | 011B  |
