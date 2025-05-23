---
title: Renesas-RH850  CPU System - TPTM
date: 2025/1/14
categories: 
- Renesas
---

<center>

Renesas学习笔记
CPU System :  TPTM

</center>

<!-- more -->

***

##### 1 TPTM 概述
时间保护定时器 (TPTM) 是一组专用于 CPU 的定时器，用于实现高性能和时间保护功能。每个 CPU 分别分配一个定时器集。一个定时器集由 2 个间隔定时器、1 个自由运行定时器和 2 个上升定时器组成。

每个 TPTM 的定时器集具有以下功能：
- 间隔定时器 × 2 通道（向下计数器）
- 自由运行定时器 × 1 通道（向上计数器）
- 上升定时器 × 2 通道（向上计数器）
- 启动、停止和重新启动间隔定时器、自由运行定时器和上升定时器的计数器。
- 可分别为间隔定时器、自由运行定时器和上升定时器配置分频计数器。
- 同一 PE 的间隔定时器的同步计数控制。
- 使用为每个 PE 准备的寄存器对同一 PE 的上升定时器进行同步计数控制。
- 准备了全局同步计数控制寄存器。一个 PE 可以控制包括其他 PE 所有的所有向上计数器。
- 间隔定时器的下溢中断。
- 上升定时器的比较值匹配中断。
- 寄存器访问的地址 EDC。
- 寄存器访问的数据 ECC。
- 支持防止未经授权访问的保护功能。
- 每个定时器集有 3 个控制信号以在调试模式下停止定时器。一个用于间隔定时器和自由运行定时器，另一个用于上升定时器 0，第三个用于上升定时器 1。

##### 2 TPTM 寄存器

###### 2.1 Self Region

自区域包含以下寄存器。自区域只能由特定 CPU 访问。
自寄存器是虚拟寄存器，实际上并不存在。从每个 PE 访问自寄存器被路由到访问源 PE 对应的实际寄存器。表 3.167 列出了访问源 PEs 和路由目标寄存器。有关每个寄存器位的功能，请参见路由目标寄存器的规范。当除 PEx 之外的主设备访问自寄存器时，寄存器返回 0，写访问被忽略，并通知错误响应。

基本上假设 TPTM 寄存器是通过自区域从 PEs 访问的。这使 PEs 可以使用相同的代码，因为不需要为每个 PE 指定不同的寄存器地址。

也可以通过指定实际寄存器的地址而不使用自寄存器直接访问寄存器。在这种情况下，假设目的是检查另一个 PE 的寄存器状态或通过调试工具引用单个寄存器。


- TPTMSIRUN — 间隔定时器的计数器启动寄存器
- TPTMSIRRUN — 间隔定时器的计数器重新启动寄存器
- TPTMSISTP — 间隔定时器的计数器停止寄存器
- TPTMSISTR — 间隔定时器的计数器状态寄存器
- TPTMSIIEN — 间隔定时器的中断启用寄存器
- TPTMSIUSTR — 间隔定时器的下溢状态寄存器
- TPTMSIDIV — 间隔定时器的分频寄存器
- TPTMSFRUN — 自由运行定时器的计数器启动寄存器
- TPTMSFRRUN — 自由运行定时器的计数器重新启动寄存器
- TPTMSFSTP — 自由运行定时器的计数器停止寄存器
- TPTMSFSTR — 自由运行定时器的计数器状态寄存器
- TPTMSFDIV — 自由运行定时器的分频寄存器
- TPTMSURUN — 上升定时器的计数器启动寄存器
- TPTMSURRUN — 上升定时器的计数器重新启动寄存器
- PTMSUSTP — 上升定时器的计数器停止寄存器
- TPTMSUSTR — 上升定时器的计数器状态寄存器
- TPTMSUIEN — 上升定时器的中断启用寄存器
- TPTMSUDIV — 上升定时器的分频寄存器
- TPTMSUTRG — 上升定时器的触发选择寄存器
- TPTMSICNT0 — 间隔定时器的通道 0 计数器寄存器
- TPTMSILD0 — 间隔定时器的通道 0 加载寄存器
- TPTMSICNT1 — 间隔定时器的通道 1 计数器寄存器
- TPTMSILD1 — 间隔定时器的通道 1 加载寄存器
- TPTMSFCNT — 自由运行定时器的计数器寄存器
- TPTMSUCNTm — 上升定时器的通道 m 计数器寄存器 (m = 0 到 1)
- TPTMSUCMPmi — 上升定时器 m 的比较值 i 寄存器 (m = 0 到 1, i = 0 到 3)



###### 2.2 TPTMnIRUN — 用于 PEn 的间隔定时器计数器启动寄存器
此寄存器控制每个通道的间隔定时器启动。

访问：此寄存器是一个只能写入的寄存器，可以按 32 位、16 位或 8 位单位写入。

地址：<TPTM_base> + 100H + 100H × n

复位后的值：0000 0000H


| 位位置    | 位名称     | 功能                                                                     |
| ------- | --------- | ---------------------------------------------------------------------- |
| 31 到 2  | 保留        | 写入时写复位后的值。                                                           |
| 1       | IRUN1     | 间隔定时器通道 1 的启动命令位                                                  |
|         |           | 0：无操作                                                                |
|         |           | 1：启动计数器                                                              |
|         |           | 写入 1 到该位时，TPTMnILD1 的值将加载到 TPTMnICNT1 中。此位始终读取为 0。                 |
| 0       | IRUN0     | 间隔定时器通道 0 的启动命令位                                                  |
|         |           | 0：无操作                                                                |
|         |           | 1：启动计数器                                                              |
|         |           | 写入 1 到该位时，TPTMnILD0 的值将加载到 TPTMnICNT0 中。此位始终读取为 0。                 |

###### 2.3 TPTMnIRRUN — 用于 PEn 的间隔定时器计数器重新启动寄存器
此寄存器控制每个通道的间隔定时器重新启动。

访问：此寄存器是一个只能写入的寄存器，可以按 32 位、16 位或 8 位单位写入。

地址：<TPTM_base> + 104H + 100H × n

复位后的值：0000 0000H


| 位位置    | 位名称     | 功能                                                                     |
| ------- | --------- | ---------------------------------------------------------------------- |
| 31 到 2  | 保留        | 写入时写复位后的值。                                                           |
| 1       | IRRUN1    | 间隔定时器通道 1 的重新启动命令位                                               |
|         |           | 0：无操作                                                                |
|         |           | 1：重新启动计数器                                                           |
|         |           | 写入 1 到该位时，计数器将从 TPTMnICNT1 的当前值重新启动。此位始终读取为 0。           |
| 0       | IRRUN0    | 间隔定时器通道 0 的重新启动命令位                                               |
|         |           | 0：无操作                                                                |
|         |           | 1：重新启动计数器                                                           |
|         |           | 写入 1 到该位时，计数器将从 TPTMnICNT0 的当前值重新启动。此位始终读取为 0。           |



###### 2.4 TPTMnISTP — 用于 PEn 的间隔定时器计数器停止寄存器
此寄存器控制每个通道的间隔定时器停止。

访问：此寄存器是一个只能写入的寄存器，可以按 32 位、16 位或 8 位单位写入。

地址：<TPTM_base> + 108H + 100H × n

复位后的值：0000 0000H

| 位位置    | 位名称     | 功能                                                                      |
| ------- | --------- | ----------------------------------------------------------------------- |
| 31 到 2  | 保留        | 写入时写复位后的值。                                                            |
| 1       | ISTP1     | 间隔定时器通道 1 的停止命令位                                                   |
|         |           | 0：无操作                                                                 |
|         |           | 1：停止计数器                                                               |
|         |           | 写入 1 到该位时，计数器将在 TPTMnICNT1 的值处停止。此位始终读取为 0。                     |
| 0       | ISTP0     | 间隔定时器通道 0 的停止命令位                                                   |
|         |           | 0：无操作                                                                 |
|         |           | 1：停止计数器                                                               |
|         |           | 写入 1 到该位时，计数器将在 TPTMnICNT0 的值处停止。此位始终读取为 0。                     |


###### 2.5 TPTMnISTR — 用于 PEn 的间隔定时器计数器状态寄存器
此寄存器指示每个通道的间隔定时器状态。

访问：此寄存器是一个只读寄存器，可以按 32 位、16 位或 8 位单位读取。

地址：<TPTM_base> + 10CH + 100H × n

复位后的值：0000 0000H


| 位位置    | 位名称     | 功能                                                                 |
| ------- | --------- | ------------------------------------------------------------------ |
| 31 到 2  | 保留        | 读取时返回复位后的值。                                                       |
| 1       | ISTR1     | 间隔定时器通道 1 的状态位                                                |
|         |           | 0：计数器已停止。                                                          |
|         |           | 1：计数器正在运行。                                                        |
| 0       | ISTR0     | 间隔定时器通道 0 的状态位                                                |
|         |           | 0：计数器已停止。                                                          |
|         |           | 1：计数器正在运行。                                                        |

###### 2.6 TPTMnIIEN — 用于 PEn 的间隔定时器中断启用寄存器
此寄存器控制通过 TPTM_IRQ[n] 启用/禁用中断请求。当 TPTMn 中的间隔定时器通道 m 发生下溢时，无论是否设置了 IIENm 位，TPTMnIUSTR.IUSTRm 位都会被设置。如果设置了 IIENm 位，则 TPTM_IRQ[n] 被断言。请注意，如果 TPTMnIUSTR.IUSTRm 持有 1，则在设置 IIENm 位后，TPTM_IRQ[n] 将立即被断言。

访问：此寄存器可以按 32 位、16 位或 8 位单位读取或写入。

地址：<TPTM_base> + 110H + 100H × n

复位后的值：0000 0000H

| 位位置    | 位名称     | 功能                                                                       |
| ------- | --------- | ------------------------------------------------------------------------ |
| 31 到 2  | 保留        | 读取时返回复位后的值。写入时写复位后的值。                                          |
| 1       | IIEN1     | 间隔定时器通道 1 的中断启用位<br>0：禁用 <br> 1：启用                                                 |
| 0       | IIEN0     | 间隔定时器通道 0 的中断启用位<br>0：禁用 <br> 1：启用|







###### 2.7 TPTMnIUSTR — 用于 PEn 的间隔定时器下溢状态寄存器
此寄存器指示 TPTMn 中间隔定时器的下溢发生状态。

访问：此寄存器可以按 32 位、16 位或 8 位单位读取或写入。

地址：<TPTM_base> + 114H + 100H × n

复位后的值：0000 0000H

| 位位置    | 位名称     | 功能                                                                    |
| ------- | --------- | --------------------------------------------------------------------- |
| 31 到 2  | 保留        | 读取时返回复位后的值。写入时写复位后的值。                                               |
| 1       | IUSTR1    | 间隔定时器通道 1 的下溢标志位                                               |
|         |           | 0：未发生下溢                                                              |
|         |           | 1：发生下溢                                                                |
|         |           | 写入 0 到该位时，IUSTR1 被清除。写入 1 到该位被忽略。                                |
| 0       | IUSTR0    | 间隔定时器通道 0 的下溢标志位                                               |
|         |           | 0：未发生下溢                                                              |
|         |           | 1：发生下溢                                                                |
|         |           | 写入 0 到该位时，IUSTR0 被清除。写入 1 到该位被忽略。                                |

注意:要清除单个位，建议使用 CLR1 指令。

###### 2.8 TPTMnIDIV — 用于 PEn 的间隔定时器分频寄存器
此寄存器指定每个通道的间隔定时器计数时钟的分频比。

访问：此寄存器可以按 32 位、16 位或 8 位单位读取或写入。

地址：<TPTM_base> + 118H + 100H × n

复位后的值：0000 0000H


| 位位置    | 位名称     | 功能                                                                     |
| ------- | --------- | ---------------------------------------------------------------------- |
| 31 到 8  | 保留        | 读取时返回复位后的值。写入时写复位后的值。                                                 |
| 7 到 0   | IDIV[7:0] | 所有通道间隔定时器的时钟分频设置                                              |
|         |           | 计数器时钟比率由以下表达式指定：                                                |
|         |           | count_clock = cpu_clk / (IDIV[7:0] + 1)                                    |

注意:
- 为避免意外的定时器行为，建议在 TPTMnICNTm 所有通道都处于停止状态时写入 TPTMnIDIV 寄存器。
- 禁止通过位操作指令访问此寄存器。

###### 2.8 TPTMnFRUN — 用于 PEn 的自由运行定时器计数器启动寄存器
此寄存器控制自由运行定时器的启动。

访问：此寄存器是一个只能写入的寄存器，可以按 32 位、16 位或 8 位单位写入。

地址：<TPTM_base> + 120H + 100H × n

复位后的值：0000 0000H

| 位位置    | 位名称     | 功能                                                                     |
| ------- | --------- | ---------------------------------------------------------------------- |
| 31 到 1  | 保留        | 写入时写复位后的值。                                                           |
| 0       | FRUN      | 自由运行定时器的启动命令位                                                      |
|         |           | 0：无操作                                                                |
|         |           | 1：启动计数器                                                              |
|         |           | 在计数器停止期间写入 1 到该位时，0000 0000H 的值将加载到 TPTMnFCNT 并启动计数器。           |
|         |           | 在计数器工作期间写入 1 到该位时，0000 0000H 的值将加载到 TPTMnFCNT 并重新启动计数器。         |
|         |           | 此位始终读取为 0。                                                           |





###### 2.9 TPTMnFRRUN — 用于 PEn 的自由运行定时器计数器重新启动寄存器
此寄存器控制自由运行定时器的重新启动。

访问：此寄存器是一个只能写入的寄存器，可以按 32 位、16 位或 8 位单位写入。

地址：<TPTM_base> + 124H + 100H × n

复位后的值：0000 0000H


| 位位置    | 位名称     | 功能                                                                     |
| ------- | --------- | ---------------------------------------------------------------------- |
| 31 到 1  | 保留        | 写入时写复位后的值。                                                           |
| 0       | FRRUN     | 自由运行定时器的重新启动命令位                                                    |
|         |           | 0：无操作                                                                |
|         |           | 1：重新启动计数器                                                           |
|         |           | 写入 1 到该位时，计数器将从 TPTMnFCNT 的当前值重新启动。此位始终读取为 0。               |


###### 2.10 TPTMnFSTP — 用于 PEn 的自由运行定时器计数器停止寄存器
此寄存器控制自由运行定时器的停止。

访问：此寄存器是一个只能写入的寄存器，可以按 32 位、16 位或 8 位单位写入。

地址：<TPTM_base> + 128H + 100H × n

复位后的值：0000 0000H

| 位位置    | 位名称     | 功能                                                                     |
| ------- | --------- | ---------------------------------------------------------------------- |
| 31 到 1  | 保留        | 写入时写复位后的值。                                                           |
| 0       | FSTP      | 自由运行定时器的停止命令位                                                      |
|         |           | 0：无操作                                                                |
|         |           | 1：停止计数器                                                              |
|         |           | 写入 1 到该位时，计数器将停止在 TPTMnFCNT 的值。此位始终读取为 0。                     |


###### 2.11 TPTMnFSTR — 用于 PEn 的自由运行定时器计数器状态寄存器
此寄存器指示自由运行定时器状态。

访问：此寄存器是一个只读寄存器，可以按 32 位、16 位或 8 位单位读取。

地址：<TPTM_base> + 12CH + 100H × n

复位后的值：0000 0000H

| 位位置    | 位名称     | 功能                                                                    |
| ------- | --------- | --------------------------------------------------------------------- |
| 31 到 1  | 保留        | 读取时返回复位后的值。                                                           |
| 0       | FSTR      | 自由运行定时器的状态位                                                      |
|         |           | 0：计数器停止                                                              |
|         |           | 1：计数器运行                                                              |








###### 2.12 TPTMnFDIV — 用于 PEn 的自由运行定时器分频寄存器
此寄存器指定自由运行定时器计数时钟的分频比。

访问：此寄存器可以按 32 位、16 位或 8 位单位读取或写入。

地址：<TPTM_base> + 130H + 100H × n

复位后的值：0000 0000H

| 位位置    | 位名称      | 功能                                                                     |
| ------- | ---------  | ---------------------------------------------------------------------- |
| 31 到 8  | 保留         | 读取时返回复位后的值。写入时写复位后的值。                                                 |
| 7 到 0   | FDIV[7:0]  | 自由运行定时器的时钟分频设置                                               |
|         |            | 计数器时钟比率由以下表达式指定：                                                |
|         |            | count_clock = cpu_clk / (FDIV[7:0] + 1)                                    |

注意
- 为避免意外的定时器行为，建议在 TPTMnFCNT 处于停止状态时写入 TPTMnFDIV 寄存器。
- 禁止通过位操作指令访问此寄存器。

###### 2.13 TPTMnURUN — 用于 PEn 的上升定时器计数器启动寄存器
此寄存器控制每个通道的上升定时器启动。

访问：此寄存器是一个只能写入的寄存器，可以按 32 位、16 位或 8 位单位写入。

地址：<TPTM_base> + 140H + 100H × n

复位后的值：0000 0000H

| 位位置    | 位名称     | 功能                                                                     |
| ------- | --------- | ---------------------------------------------------------------------- |
| 31 到 2  | 保留        | 写入时写复位后的值。                                                           |
| 1       | URUN1     | 上升定时器通道 1 的启动命令位                                                  |
|         |           | 0：无操作                                                                |
|         |           | 1：启动计数器                                                              |
|         |           | 写入 1 到该位时，0000 0000H 的值加载到 TPTMnUCNT1 中。此位始终读取为 0。                 |
| 0       | URUN0     | 上升定时器通道 0 的启动命令位                                                  |
|         |           | 0：无操作                                                                |
|         |           | 1：启动计数器                                                              |
|         |           | 写入 1 到该位时，0000 0000H 的值加载到 TPTMnUCNT0 中。此位始终读取为 0。                 |

###### 2.14 TPTMnURRUN — 用于 PEn 的上升定时器计数器重新启动寄存器
此寄存器控制每个通道的上升定时器重新启动。

访问：此寄存器是一个只能写入的寄存器，可以按 32 位、16 位或 8 位单位写入。

地址：<TPTM_base> + 144H + 100H × n

复位后的值：0000 0000H

| 位位置    | 位名称      | 功能                                                                     |
| ------- | ---------  | ---------------------------------------------------------------------- |
| 31 到 2  | 保留         | 写入时写复位后的值。                                                           |
| 1       | URRUN1     | 上升定时器通道 1 的重新启动命令位                                                    |
|         |            | 0：无操作                                                                |
|         |            | 1：重新启动计数器                                                           |
|         |            | 写入 1 到该位时，计数器将从 TPTMnUCNT1 的当前值重新启动。此位始终读取为 0。             |
| 0       | URRUN0     | 上升定时器通道 0 的重新启动命令位                                                    |
|         |            | 0：无操作                                                                |
|         |            | 1：重新启动计数器                                                           |
|         |            | 写入 1 到该位时，计数器将从 TPTMnUCNT0 的当前值重新启动。此位始终读取为 0。             |

###### 2.15 TPTMnUSTP — 用于 PEn 的上升定时器计数器停止寄存器
此寄存器控制每个通道的上升定时器停止。

访问：此寄存器是一个只能写入的寄存器，可以按 32 位、16 位或 8 位单位写入。

地址：<TPTM_base> + 148H + 100H × n

复位后的值：0000 0000H

| 位位置    | 位名称     | 功能                                                                     |
| ------- | --------- | ---------------------------------------------------------------------- |
| 31 到 2  | 保留        | 写入时写复位后的值。                                                           |
| 1       | USTP1     | 上升定时器通道 1 的停止命令位                                                  |
|         |           | 0：无操作                                                                |
|         |           | 1：停止计数器                                                              |
|         |           | 写入 1 到该位时，计数器将停止在 TPTMnUCNT1 的值。此位始终读取为 0。                     |
| 0       | USTP0     | 上升定时器通道 0 的停止命令位                                                  |
|         |           | 0：无操作                                                                |
|         |           | 1：停止计数器                                                              |
|         |           | 写入 1 到该位时，计数器将停止在 TPTMnUCNT0 的值。此位始终读取为 0。                     |




###### 2.16 TPTMnUSTR — 用于 PEn 的上升定时器计数器状态寄存器
此寄存器指示每个通道的上升定时器状态。

访问：此寄存器是一个只读寄存器，可以按 32 位、16 位或 8 位单位读取。

地址：<TPTM_base> + 14CH + 100H × n

复位后的值：0000 0000H

| 位位置    | 位名称     | 功能                                                                     |
| ------- | --------- | ---------------------------------------------------------------------- |
| 31 到 2  | 保留        | 读取时返回复位后的值。                                                           |
| 1       | USTR1     | 上升定时器通道 1 的状态位                                                      |
|         |           | 0：计数器停止（初始值）。                                                         |
|         |           | 1：计数器运行。                                                              |
| 0       | USTR0     | 上升定时器通道 0 的状态位                                                      |
|         |           | 0：计数器停止（初始值）。                                                         |
|         |           | 1：计数器运行。                                                              |

###### 2.17 TPTMnUIEN — 用于 PEn 的上升定时器中断启用寄存器
此寄存器控制 INTTPTMUni 中断请求的启用/禁用。当设置 TPTMnUIEN.UmIENi 时，如果 TPTMnUCNTm 和 TPTMnUCNTmi 匹配，则断言 INTTPTMUni。由于它们是脉冲中断，因此没有 INTTPTMUni 的状态寄存器（m = 0 到 1，i = 0 到 3）。

访问：此寄存器可以按 32 位、16 位或 8 位单位读取或写入。

地址：<TPTM_base> + 150H + 100H × n

复位后的值：0000 0000H


| 位位置    | 位名称     | 功能                                                                     |
| ------- | --------- | ---------------------------------------------------------------------- |
| 31 到 12 | 保留        | 读取时返回复位后的值。写入时写复位后的值。                                                 |
| 11      | U1IEN3    | 上升定时器通道 1 的比较值 3 的中断启用位                                           |
|         |           | 0：禁用（初始值）。                                                         |
|         |           | 1：启用。                                                                |
| 10      | U1IEN2    | 上升定时器通道 1 的比较值 2 的中断启用位                                           |
|         |           | 0：禁用（初始值）。                                                         |
|         |           | 1：启用。                                                                |
| 9       | U1IEN1    | 上升定时器通道 1 的比较值 1 的中断启用位                                           |
|         |           | 0：禁用（初始值）。                                                         |
|         |           | 1：启用。                                                                |
| 8       | U1IEN0    | 上升定时器通道 1 的比较值 0 的中断启用位                                           |
|         |           | 0：禁用（初始值）。                                                         |
|         |           | 1：启用。                                                                |
| 7 到 4  | 保留        | 读取时返回复位后的值。写入时写复位后的值。                                                 |
| 3       | U0IEN3    | 上升定时器通道 0 的比较值 3 的中断启用位                                           |
|         |           | 0：禁用（初始值）。                                                         |
|         |           | 1：启用。                                                                |
| 2       | U0IEN2    | 上升定时器通道 0 的比较值 2 的中断启用位                                           |
|         |           | 0：禁用（初始值）。                                                         |
|         |           | 1：启用。                                                                |
| 1       | U0IEN1    | 上升定时器通道 0 的比较值 1 的中断启用位                                           |
|         |           | 0：禁用（初始值）。                                                         |
|         |           | 1：启用。                                                                |
| 0       | U0IEN0    | 上升定时器通道 0 的比较值 0 的中断启用位                                           |
|         |           | 0：禁用（初始值）。                                                         |
|         |           | 1：启用。                                                                |

###### 2.18 TPTMnUDIV — 用于 PEn 的上升定时器分频寄存器
此寄存器指定每个通道的上升定时器计数时钟的分频比。

访问：此寄存器可以按 32 位、16 位或 8 位单位读取或写入。

地址：<TPTM_base> + 158H + 100H × n

复位后的值：0000 0000H

| 位位置    | 位名称      | 功能                                                                     |
| ------- | ---------  | ---------------------------------------------------------------------- |
| 31 到 8  | 保留         | 读取时返回复位后的值。写入时写复位后的值。                                                 |
| 7 到 0   | UDIV[7:0]  | 上升定时器的时钟分频设置                                                   |
|         |            | 计数器时钟比率由以下表达式指定：                                                |
|         |            | count_clock = cpu_clk / (UDIV[7:0] + 1)                                    |

注意
- 为避免意外的定时器行为，建议在 TPTMnUCNTm 所有通道都处于停止状态时写入 TPTMnUDIV 寄存器。
- 禁止通过位操作指令访问此寄存器。






###### 2.19 TPTMnUTRG — 用于 PEn 的上升定时器触发选择寄存器
此寄存器指定每个通道的上升定时器触发。

访问：此寄存器可以按 32 位、16 位或 8 位单位读取或写入。

地址：<TPTM_base> + 15CH + 100H × n

复位后的值：0000 0000H

| 位位置    | 位名称       | 功能                                                                     |
| ------- | ---------- | ---------------------------------------------------------------------- |
| 31 到 16 | 保留         | 读取时返回复位后的值。写入时写复位后的值。                                                 |
| 15      | GTRGEN1     | 上升定时器 1 的全局触发启用                                                   |
|         |             | 0：上升计数器 1 由 TPTMnURUN, TPTMnURRUN, TPTMnUSTP 控制                              |
|         |             | 1：上升计数器 1 由同步控制寄存器 (TPTMGgURUN, TPTMGgURRUN, TPTMGgUSTP) 控制            |
| 14 到 9  | 保留         | 读取时返回复位后的值。写入时写复位后的值。                                                 |
| 8       | GTRGCH1     | 此位用于指定控制上升计数器 1 的寄存器。                                           |
|         |             | 当 TPTMnUTRG.GTRGEN1 = 1 且 TPTMnUTRG.GTRGCH1 = g（g = 0 到 1）时，                      |
|         |             | 上升计数器 1 由 TPTMGgURUN, TPTMGgURRUN, TPTMGgUSTP 控制。                                 |
| 7       | GTRGEN0     | 上升定时器 0 的全局触发启用                                                   |
|         |             | 0：上升计数器 0 由 TPTMnURUN, TPTMnURRUN, TPTMnUSTP 控制                              |
|         |             | 1：上升计数器 0 由同步控制寄存器 (TPTMGgURUN, TPTMGgURRUN, TPTMGgUSTP) 控制            |
| 6 到 1  | 保留         | 读取时返回复位后的值。写入时写复位后的值。                                                 |
| 0       | GTRGCH0     | 此位用于指定控制上升计数器 0 的寄存器。                                           |
|         |             | 当 TPTMnUTRG.GTRGEN0 = 1 且 TPTMnUTRG.GTRGCH0 = g（g = 0 到 1）时，                      |
|         |             | 上升计数器 0 由 TPTMGgURUN, TPTMGgURRUN, TPTMGgUSTP 控制。                                 |


###### 2.20 TPTMnICNTm — 用于 PEn 的间隔定时器通道 m 计数器寄存器
此寄存器是间隔定时器通道 m 的计数器本身（m = 0 到 1）。

访问：此寄存器可以按 32 位单位读取或写入。

地址：<TPTM_base> + 180H + 100H × n + 8H × m

复位后的值：未定义

| 位位置    | 位名称        | 功能                                                                      |
| ------- | ----------- | ----------------------------------------------------------------------- |
| 31 到 0  | ICNT[31:0] | 用于 PEn 的间隔定时器通道 m 的计数器值。当写入 1B 到 TPTMnIRUN.IRUNm 时，TPTMnILDm 的值将加载到此寄存器中，并且此寄存器将从该值开始倒计数。当此寄存器发生下溢时，TPTMnILDm 的值将重新加载到此寄存器中，并且此寄存器将继续从该值倒计数。 |

**注意**
- 为避免意外的定时器行为，建议在停止状态下写入 TPTMnICNTm 寄存器。

###### 2.21 TPTMnILDm — 用于 PEn 的间隔定时器通道 m 加载寄存器
此寄存器指定间隔定时器通道 m 的加载值（m = 0 到 1）。

访问：此寄存器可以按 32 位单位读取或写入。

地址：<TPTM_base> + 184H + 100H × n + 8H × m

复位后的值：未定义

| 位位置    | 位名称       | 功能                                                                      |
| ------- | ---------- | ----------------------------------------------------------------------- |
| 31 到 0  | ILD[31:0]  | 用于 PEn 的间隔定时器通道 m 的加载数据值。当写入 1 到 TPTMnIRUNm 时，此寄存器的值将加载到 TPTMnICNTm 中。此外，当 TPTMnICNTm 发生下溢时，此寄存器的值将重新加载到 TPTMnICNTm 中。 |

**注意**
- 为避免意外的定时器行为，建议在相应通道的 TPTMnCNTm 处于停止状态时写入 TPTMnILDm 寄存器。





###### 2.22 TPTMnFCNT — 用于 PEn 的自由运行定时器计数器寄存器
此寄存器是自由运行定时器的计数器本身。

访问：此寄存器可以按 32 位单位读取或写入。

地址：<TPTM_base> + 1A0H + 100H × n

复位后的值：未定义

| 位位置    | 位名称      | 功能                                                                     |
| ------- | ---------- | ---------------------------------------------------------------------- |
| 31 到 0  | FCNT[31:0] | 用于 PEn 的自由运行定时器计数器值。当写入 1B 到 TPTMnFRUN.FRUN 时，0000 0000H 的值将加载到此寄存器中，并且此寄存器将从该值开始计数。当此寄存器发生溢出时，0000 0000H 的值将重新加载到此寄存器中，并且此寄存器将继续从该值计数。 |

**注意**
- 为避免意外的定时器行为，建议在停止状态下写入 TPTMnFCNT 寄存器。


###### 2.23 TPTMnUCNTm — 用于 PEn 的上升定时器计数器寄存器
此寄存器是上升定时器通道 m 的计数器本身（m = 0 到 1）。

访问：此寄存器可以按 32 位单位读取或写入。

地址：<TPTM_base> + 1C0H + 100H × n + 20H × m

复位后的值：未定义

| 位位置    | 位名称      | 功能                                                                     |
| ------- | ---------- | ---------------------------------------------------------------------- |
| 31 到 0  | UCNT[31:0] | 用于 PEn 的上升定时器计数器值。当写入 1B 到 TPTMnURUN.URUNm 时，0000 0000H 的值将加载到此寄存器中，并且此寄存器将从该值开始计数。此外，当此寄存器发生溢出时，0000 0000H 的值将重新加载到此寄存器中，并且此寄存器将继续从该值计数。 |

**注意**
- 为避免意外的定时器行为，建议在停止状态下写入 TPTMnUCNTm 寄存器。


###### 2.24 TPTMnUCMPmi — 用于 PEn 的上升定时器 m 的比较值 i 寄存器
此寄存器指定与上升定时器通道 m 的值相比的第 i 个值（m = 0 到 1，i = 0 到 3）。

访问：此寄存器可以按 32 位单位读取或写入。

地址：<TPTM_base> + 1C4H + 100H × n + 20H × m + 4H × i

复位后的值：0000 0000H

| 位位置    | 位名称      | 功能                                                                     |
| ------- | ---------- | ---------------------------------------------------------------------- |
| 31 到 0  | UCMP[31:0] | 与上升计数器的值相比的值。如果 TPTMnUIEN.UmIENi = 1 且 TPTMnUCNTm 等于 TPTMnUCMPmi，则断言 INTTPTMni。 |

**注意**
- 为避免意外的定时器行为，建议在停止状态下写入 TPTMnUCMPmi 寄存器。当上升定时器运行时更新 TPTMnUCMPmi 寄存器时，必须避免在写入 TPTMnUCMPmi 过程中 TPTMnUCNTm 超过 TPTMnUCMPmi，因为上升定时器的中断（INTTPTMUni）可能会丢失。

###### 2.25 TPTMGgURUN — 用于全局控制通道 g 的上升定时器计数器启动寄存器
此寄存器启动上升定时器。

访问：此寄存器是一个只能写入的寄存器，可以按 32 位、16 位或 8 位单位写入。

地址：<TPTM_base> + 940H + 100H × g（g = 0 到 1）

复位后的值：0000 0000H

| 位位置    | 位名称    | 功能                                                                                  |
| ------- | ------- | ----------------------------------------------------------------------------------- |
| 31 到 1  | 保留      | 写入时写复位后的值。                                                                         |
| 0       | URUN    | 上升定时器的启动命令位                                                                        |
|         |         | 0：无操作                                                                               |
|         |         | 1：如果 TPTMnUTRG.GTRGENm 为 1 且 TPTMnUTRG.GTRGCHm = g，则启动 PEn 的上升定时器 m。否则，无操作。 |
|         |         | 写入 1 到该位时，0000 0000H 的值加载到 TPTMnUCNTm 中。此位始终读取为 0。                                |





###### 2.26 TPTMGgURRUN — 全局控制通道g的计时器重新启动寄存器
此寄存器用于重新启动计时器。

访问：
此寄存器是只写寄存器，可以以32位、16位或8位单位写入。

地址：<TPTM_base> + 944H + 100H x g (g = 0到1)


| Bit 位置 | Bit 名称 | 功能 |
|----------|----------|------|
| 31到1    | 保留      | 写入时，写入复位后的值。 |
| 0        | URRUN    | 重新启动计时器命令位<br>0: 无操作<br>1: 如果TPTMnUTRG.GTRGENm为1且TPTMnUTRG.GTRGCHm = g，则重新启动PEn的计时器m。否则无操作。<br>写入1到该位时，计数器从TPTMnUCNTm的当前值重新启动。该位读取时始终为0。 |

###### 2.27 TPTMGgUSTP — 全局控制通道g的计时器停止寄存器
此寄存器用于停止计时器。

访问：此寄存器是只写寄存器，可以以32位、16位或8位单位写入。

地址：<TPTM_base> + 948H + 100H x g (g = 0到1)

| Bit 位置 | Bit 名称 | 功能 |
|----------|----------|------|
| 31到1    | 保留      | 写入时，写入复位后的值。 |
| 0        | USTP     | 停止计时器命令位<br>0: 无操作<br>1: 如果TPTMnUTRG.GTRGENm为1且TPTMnUTRG.GTRGCHm = g，则停止PEn的计时器n。否则无操作。<br>写入1到该位时，计数器停在TPTMnUCNTm的值。该位读取时始终为0。 |




##### 3 TPTM 功能

如果中断被用作 EI 级别而不是 FE 级别，请在 TPTM 启动前设置 TPTMSEL 寄存器。禁止在 TPTM 启动后更改 TPTMSEL 寄存器的设置值。

###### 3.1 间隔定时器操作
(1) 正常操作



图 3.56 显示了当计数器时钟比率为 CPU_CLK 的 1/2 周期且加载数据值为 0000 0004H 时，间隔定时器正常操作的时序图。

TPTMnICNTm 在设置 TPTMnIRUN.IRUNm 位的下一个周期开始倒计时。

如果在计数器运行时设置了 TPTMnIRUN.IRUNm 位，TPTMnICNTm 的值将被 TPTMnILDm 的值更新，并继续倒计时。

当 TPTMnICNTm 变为零时，TPTMnUSTR.USTRm 被设置，如果 TPTMnIIEN.IIENm 位被设置，TPTM_IRQ[n] 被断言。


(2) 重新启动期间的操作

TPTMnICNTm 在设置 TPTMnIRUN 后的下一个周期开始倒计时。

在倒计时期间设置 TPTMnISTP 时，TPTMnICNTm 在下一个周期停止倒计时。

在倒计时停止期间设置 TPTMnIRRUN 后，TPTMnICNTm 在下一个周期从当前计数器值重新开始倒计时。

当计数器时钟比率为 CPU_CLK 的 1/2 周期且加载数据值为 0000 0004H 时，重新启动时间隔定时器操作的时序图如图 3.58 所示。


（3）下溢期间的操作
如果间隔计数器仍在运行，即使 TPTMnCNTm 为 0000 0000H，也会发生计数器下溢，并且在下一个周期内设置 TPTMnIUSTR.IUSTRm 位。

当 TPTMnIIEN.IIENm 为 1B 时，中断被启用，TPTMnIUSTR.IUSTRm 被设置，并且在 TPTMnCNTm 变为 0000 0000H 后的下一个周期断言 TPTM_IRQ。

当 CPU 完成 TPTM 的中断处理后，需要清除 TPTMnIUSTR.IUSTRm。

TPTMn 中断的周期
TPTMn 中断的周期如下：
- TPTMn 中断生成周期 = TPTMnCNTm × 计数器时钟周期
  计数器时钟周期 = CPU_CLK × (TPTMnIDIV + 1)

当 TPTMnIIEN.IIENm 为 0 时，中断被禁用，TPTMnIUSTR.IUSTRm 被设置，但在 TPTMnCNTm 变为 0000 0000H 后的下一个周期内 TPTM_IRQ 不会被断言。在这种情况下，如果在清除 TPTMnIUSTR.IUSTRm 之前设置 TPTMnIIEN.IIENm，则断言 TPTM_IRQ。






###### 3.2 自由运行定时器操作
自由运行定时器的操作基本与间隔定时器相同，除了以下特点。对常见特征的解释省略。

计数器方向为向上计数。TPTMnFCNT 值在每个 TPTMnFDIV 周期内递增。

无加载值寄存器。通过设置 TPTMnFRUN.FRUN 位，TPTMnFCNT 始终从 0000 0000H 开始。

计数器溢出时没有中断断言。

通道数：1 个通道


###### 3.3 上升定时器操作
上升定时器的操作基本与间隔定时器相同，除了以下特点。对常见特征的解释省略。

计数器方向为向上计数。TPTMnUCNTm 值在每个 TPTMnUDIV 周期内递增。

无加载值寄存器。通过设置 TPTMnURUN.URUNm 位，TPTMnUCNT 始终从 0000 0000H 开始。

基于计数器值比较的边沿中断断言。当 TPTMnUCNT0 = TPTMnUCMP0i 或 TPTMnUCNT1 = TPTMnUCMP1i 时断言 INTTPTMUni。

通道数：2 个通道




















