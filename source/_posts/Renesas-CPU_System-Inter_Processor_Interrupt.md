---
title: Renesas-RH850  CPU System -  Inter-Processor Interrupt
date: 2025/1/11
categories: 
- Renesas
---

<center>

Renesas学习笔记
CPU System :  Inter-Processor Interrupt

</center>

<!-- more -->

***

##### 1 处理器间中断概述

处理器间中断寄存器 (IPIR) 是一种控制 PEs 之间快速中断请求的功能。使用 IPIR 可以比使用软件在 INTC2 中断通道上设置请求标志更快地处理 PEs 之间的中断。
IPIR 具有以下特点：
- 支持 4 通道的 PEs 之间中断功能
- 支持中断的电平检测。不支持边缘检测。
- 所有集群和所有 PEs 可访问
- 可以识别中断请求源 PE。
- 可以通过屏蔽中断请求防止意外的 PEs 之间中断。
- 可将 SET1、CLR1 和 NOT1 作为原子操作指令执行到 IPIR。
- 支持地址 EDC 功能
- 支持数据 ECC 功能
- 支持防止未经授权访问的保护功能
  - 也可以使用 LD 指令和 ST 指令进行读修改写操作，但不能作为原子操作。

##### 2 处理器间中断寄存器


###### 2.1 Self Region
Self Region 包含以下五种类型的寄存器：

- IPInENS — IPIRn 处理器间中断启用自寄存器
- IPInFLGS — IPIRn 处理器间中断标志自寄存器
- IPInFCLRS — IPIRn 处理器间中断清除自寄存器
- IPInREQS — IPIRn 处理器间中断请求自寄存器
- IPInRCLRS — IPIRn 处理器间中断请求清除自寄存器

自寄存器是虚拟寄存器，实际上并不存在。从每个 PE 访问自寄存器被路由到访问源 PE 对应的实际寄存器。表 3.149 列出了访问源 PEs 和路由目标寄存器。有关每个寄存器位的功能，请参见路由目标寄存器的规范。当除 PEx 之外的主设备访问自寄存器时，寄存器返回 0，写访问被忽略，并将通知错误响应。

基本上假设 PEs 使用 IPIR 功能时，通过自区域访问 IPIR 寄存器。这使 PEs 可以使用相同的代码，因为不需要为每个 PE 指定不同的寄存器地址。

也可以通过指定实际寄存器的地址而不使用自寄存器直接访问寄存器。在这种情况下，目的是检查另一个 PE 的寄存器状态或通过调试工具引用单个寄存器。

表 3.149 自区域寄存器路由列表
| 访问源 PE    | PE0       | PE1       | PE2       | PE3       |
| ------- | ---------- | -------- | -------- | -------- |
| IPInENS |            | IPInEN0  | IPInEN1  | IPInEN2  | IPInEN3  |
| IPInFLGS|            | IPInFLG0 | IPInFLG1 | IPInFLG2 | IPInFLG3 |
| IPInFCLRS|           | IPInFCLR0| IPInFCLR1| IPInFCLR2| IPInFCLR3|
| IPInREQS|            | IPInREQ0 | IPInREQ1 | IPInREQ2 | IPInREQ3 |
| IPInRCLRS|           | IPInRCLR0| IPInRCLR1| IPInRCLR2| IPInRCLR3|


###### 2.2 IPInENm — IPIRn 处理器间中断启用寄存器 m
这个寄存器设置允许发送PE向PEm发出PE间中断请求

此寄存器用于由接收 PE (PEm) 自身启用 PE (PEx) 的处理器间中断请求。

访问方式：此寄存器可以以 8 位为单位进行读取或写入。

地址：<IPIR_base> + 800H + 020H × n + 100H × m

复位后的值：00H

| 位位置  | 位名称     | 功能                                                                                      |
| ---- | --------- | ----------------------------------------------------------------------------------------- |
| 7 到 4 | 保留        | 读取时返回复位后的值。写入时写复位后的值。                                                              |
| 3 到 0 | EN[3:0]   | 处理器间中断启用。写入 1 到第 x 位以启用从 PEx 到 PEm 的处理器间中断请求。写入 0 到第 x 位以禁用从 PEx 到 PEm 的处理器间中断请求。<br>位 0：PE0 <br>位 1：PE1 <br>位 2：PE2 <br>位 3：PE3 |



###### 2.3 IPInFLGm — IPIRn 处理器间中断标志寄存器
此寄存器指示向 PEm 发出处理器间中断请求的发送 PE。

此寄存器用于在 PEm 接收到处理器间中断请求时区分请求的 PE。

访问方式：此寄存器是只读寄存器，可以以 8 位为单位读取。

地址：<IPIR_base> + 804H + 020H × n + 100H × m

复位后的值：00H

| 位位置  | 位名称     | 功能                                                                                      |
| ---- | --------- | ----------------------------------------------------------------------------------------- |
| 7 到 4 | 保留        | 读取时返回复位后的值。                                                                   |
| 3 到 0 | FLG[3:0]   | 处理器间中断请求标志。此寄存器指示来自其他 PEs 的处理器间中断请求状态。当 IPInENm[x] 位的值为 1 时，第 x 位会根据 IPInREQx[m] 的变化自动更新。如果 IPInENm[x] 位为 0，即使 IPInREQx[m] 设置，第 x 位也不会设置。当设置 IPInFCLRm[x] 位时，第 x 位也会被清除。 <br> 位 0：PE0 <br> 位 1：PE1 <br> 位 2：PE2 <br> 位 3：PE3|


###### 2.4 IPInFCLRm — IPIRn 处理器间中断清除寄存器 m
此寄存器清除对 PEm 的处理器间中断请求。

此寄存器用于在接收到来自另一个 PE 的处理器间中断请求后清除请求标志 (IPInFLGm) 和请求 (IPInREQx)。

访问方式：此寄存器是只写寄存器，可以以 8 位为单位写入。

地址：<IPIR_base> + 808H + 020H × n + 100H × m

复位后的值：00H

| 位位置  | 位名称    | 功能                                                                                  |
| ---- | -------- | ----------------------------------------------------------------------------------- |
| 7 到 4 | 保留       | 写入时写复位后的值。                                                                      |
| 3 到 0 | FCLR[3:0] | 处理器间中断请求标志清除。通过向此位写入 1 可以清除 IPInFLGm[x] 和 IPInREQx[m]。写入 0 被忽略。 <br> 位 0：PE0 <br>位 1：PE1 <br>位 2：PE2 <br>位 3：PE3|



###### 2.5 IPInREQm — IPIRn 处理器间中断请求寄存器 m
此寄存器控制 PEm 到其他 PEs 的处理器间中断请求。

访问：此寄存器可以按 8 位单位读取或写入。

地址：<IPIR_base> + 810H + 020H × n + 100H × m

复位后的值：00H
| 位位置  | 位名称     | 功能                                                                                       |
| ---- | --------- | ------------------------------------------------------------------------------------------ |
| 7 到 4 | 保留        | 读取时返回复位后的值。写入时写复位后的值。                                                               |
| 3 到 0 | REQ[3:0]   | 处理器间中断请求。当向 x 位写入 1 ，且IPInENx[m] 位值为 1：x 位值变为 1；将高电平输出到 PEx 的处理器间中断请求，IPInFLGx[m] 自动设为 1。<br>当 向x 位写入 1 时，但IPInENx[m] 位值为 0：x 位值变为 1；没有其他操作。<br>写入 0 被忽略。当读取时：寄存器值会被读出。 <br> 位 0：PE0 <br>位 1：PE1 <br>位 2：PE2 <br>位 3：PE3|


###### 2.6 IPInRCLRm — IPIRn 处理器间中断请求清除寄存器 m
此寄存器清除 PEm 对其他 PEs 的处理器间中断请求。

假定此寄存器用于由传输 PE 清除处理器间中断请求。

访问：此寄存器是一个只能写入的寄存器，可以按 8 位单位写入。

地址：<IPIR_base> + 814H + 020H × n + 100H × m

复位后的值：00H


| 位位置  | 位名称     | 功能                                                                                      |
| ---- | --------- | ----------------------------------------------------------------------------------------- |
| 7 到 4 | 保留        | 写入时写复位后的值。                                                                      |
| 3 到 0 | RCLR[3:0]  | 处理器间中断请求清除。向此位写入 1 可以清除 IPInREQm[x] 位。如果 IPInENx[m] 位的值为 1，则同时清除 IPInFLGx[m] 位。允许通过 IPInENm 寄存器发出处理器间中断请求的传输 PE 也可以使用此位取消处理器间中断请求，但在进行取消处理时，接收 PE 可能在某些情况下接受处理器间中断请求。写入 0 被忽略。此位在读取时始终返回 0。 <br> 位 0：PE0 <br>位 1：PE1 <br>位 2：PE2 <br>位 3：PE3|





##### 3 互处理器中断功能
###### 3.1 初始设置
在使用IPIR之前，必须通过IPInENm寄存器对授权的PE进行初始设置。下面是初始设置的一个例子。

(1) 接收PE的初始设置
图3.40展示了由PE1（接收PE）进行IPIR通道0的初始设置的例子。

首先，如果IPIR已经被使用，并且IPI0REQ0到3[1]位的值和IPI0FLG1寄存器的值已经从初始值更改，PE1将0FH写入IPI0FCLRS（= IPI0FCLR1）寄存器，以清除IPI0REQ0到3[1]位和IPI0FLG1寄存器的位。如果IPI0REQ0到3[1]位和IPI0FLG1寄存器的值是初始值，例如硬件复位后，则不需要执行此清除操作。

接下来，PE1将01H写入IPI0ENS（= IPI0EN1）寄存器，以接受从PE0到PE1的中断请求。与清除寄存器类似，完成使能寄存器的设置后，PE1可以接受来自PE0的中断请求。

(2) 由控制 PE 执行的初始设置
以下显示了来自接收 PE 以外的 PE 的初始设置示例。在本节中，为方便起见，执行初始设置的 PE 被称为控制 PE。图 3.41 显示了由 PE3（控制 PE）进行的 IPIR 通道 0 的初始设置示例。在此图中，省略了每个寄存器的 PE0 至 PE3 位。

首先，如果 IPIR 已经被使用，并且位 IPI0REQ0 至 3[1] 的值和 IPI0FLG1 寄存器的值已从初始值更改，则 PE3 写入 0FH 到 IPI0FCLR1 寄存器，以清除位 IPI0REQ0 至 3[1] 和 IPI0FLG1 寄存器。如果位 IPI0REQ0 至 3[1] 的值和 IPI0FLG1 寄存器的值为初始值，例如在硬件重置后，则不需要此清除操作。

接下来，PE3 写入 01H 到 IPI0EN1 寄存器，以接受来自 PE0 到 PE1 的中断请求。与清除寄存器类似，在设置启用寄存器后，PE1 可以接受来自 PE0 的中断请求。


###### 3.2 处理器间中断请求
图 3.42 显示了 PE0 使用 IPIR 通道 0 向 PE1 发送处理器间中断请求时的操作示例。在此图中，省略了每个寄存器的 PE0 至 PE3 位。在此示例中，初始设置预先设置为 IPI0EN1，以启用来自 PE0 的处理器间中断请求。有关初始设置的操作示例，请参见第 3.4.3.1 节，初始设置。以下描述了通过自寄存器访问寄存器的示例。

首先，作为发送 PE 的 PE0 读取 IPI0REQS (= IPI0REQ0) 寄存器，并检查 IPI0REQS[1] (= IPI0REQ0[1]) 位的值是否为 0B。如果 IPI0REQS[1] (= IPI0REQ0[1]) 位的值为 1B，这意味着来自 PE0 到 PE1 的前一个处理器间中断请求尚未被 PE1 接受，因此无法发出新的处理器间中断请求。如果确认 IPI0REQS[1] (= IPI0REQ0[1]) 位的值为 0B，从而启用了来自 PE0 到 PE1 的处理器间中断请求，PE0 将 IPI0REQS[1] (= IPI0REQ0[1]) 位设置为 1B，以发出来自 PE1 的处理器间中断请求。由于 IPI0EN1 寄存器启用了来自 PE0 到 PE1 的处理器间中断，当 PE0 将 IPI0REQS[1] (= IPI0REQ0[1]) 位设置为 1B 时，IPI0FLG1[1] 位会自动设置为 1B，并从 IPIR 向 PE1 输出处理器间中断请求信号。

图 3.43 显示了 PE1 使用 IPIR 通道 0 接收来自 PE0 的处理器间中断请求时的操作示例。当接收 PE 的 PE1 接收处理器间中断请求信号时，它读取 IPI0FLGS (= IPI0FLG1) 寄存器，并且由于 IPI0FLGS[0] (= IPI0FLG1[0]) 位的值为 1B，它识别到处理器间中断请求已从 PE0 发送。但如果请求方清除了请求，其值为 0B。在验证处理器间中断的来源后，PE1 将 IPI0FCLRS[0] (= IPI0FCLR1[0]) 位设置为 1B，处理转变为处理器间中断处理。当 PE1 将 IPI0FCLRS[0] (= IPI0FCLR1[0]) 位设置为 1B 时，IPI0REQS[1] (= IPI0REQ0[1]) 位和 IPI0FLGS (= IPI0FLG1) 位会自动清除。

图 3.44 显示了从接收 PE（PE1）进行 IPIR 初始设置，到发送 PE（PE0）发送处理器间中断请求，再到接收 PE（PE1）完成接收处理器间中断请求的操作流程。

请注意，**如果在源寄存器被接收 PE 使用清除寄存器清除之前，向相同接收 PE 生成新的处理器间中断请求，源寄存器中对应于新发送 PE 的位将设置为 1B，但处理器间中断请求信号保持高电平，第二个和后续的处理器间中断请求将不会输出。因此，接收 PE 在接收到处理器间中断请求时发生多个源时的处理必须由软件控制。**

发送 PE 可以通过检查请求寄存器的值来检测前一个处理器间中断请求是否已成功被接收 PE 接收。为了通过轮询循环监视请求寄存器，建议在轮询循环中执行休眠指令，以避免长时间占用总线系统。



3.4 请求屏蔽功能
通过IPInENm寄存器设置禁用的互PE中断请求将被忽略。图3.47展示了当禁用的PE请求互PE中断时的操作示例。在这个图中，每个寄存器的PE0到PE3位被省略。在这个示例中，初始设置提前设定为IPI0EN1，以禁止从PE2到PE1的互PE中断请求。关于初始设置的操作示例，请参见第3.4.3.1节，初始设置。

即使PE2将IPI0REQS[1]（= IPI0REQ2[1]）设置为1B，以请求PE1发出互PE中断，由于从PE2到PE1的互PE中断被禁止，因此IPInFLG1[2]位保持为0B，并且不会输出中断请求信号。

如果通过IPInENm寄存器设置禁用的发送PE意外地将1B写入IPI0REQm寄存器，可以使用IPInRCLRm寄存器清除IPI0REQm寄存器。详情请参见第3.4.3.5节，互PE中断请求清除功能。



###### 3.3 请求屏蔽功能
通过IPInENm寄存器设置禁用的互PE中断请求将被忽略。图3.47展示了当禁用的PE请求互PE中断时的操作示例。在这个图中，每个寄存器的PE0到PE3位被省略。在这个示例中，初始设置提前设定为IPI0EN1，以禁止从PE2到PE1的互PE中断请求。关于初始设置的操作示例，请参见第3.4.3.1节，初始设置。

即使PE2将IPI0REQS[1]（= IPI0REQ2[1]）设置为1B，以请求PE1发出PE间中断，由于从PE2到PE1的互PE中断被禁止，因此IPInFLG1[2]位保持为0B，并且不会输出中断请求信号。

如果通过IPInENm寄存器设置禁用的发送PE意外地将1B写入IPI0REQm寄存器，可以使用IPInRCLRm寄存器清除IPI0REQm寄存器。详情请参见第3.4.3.5节，互PE中断请求清除功能。


###### 3.4 多系统的互PE中断请求
如果发送PE和接收PE的组合不同，可以在IPIR的一个通道上同时输出多个PE间中断请求。例如，可以在IPIR的通道1上同时输出PE间中断请求A（PE0到PE1）和中断请求B（PE1到PE0）。当发送PE和接收PE的组合不同时，每个寄存器独立操作，因此可以同时输出多个互PE中断请求。




###### 3.5 处理器间中断请求清除功能
通过将 IPInRCLRm[x] 位设置为 1B，可以清除 IPInREQm[x] 位，从而取消来自发送 PE 的处理器间中断请求。如果 IPInENx[m] 位启用了来自发送 PE 的处理器间中断请求，当清除 IPInREQm[x] 位时，IPInFLGx[m] 位也会同时清除。但是，在进行取消处理时，接收 PE 可能在某些情况下接受处理器间中断。

图 3.49 显示了清除来自 PE0 和 PE2 到 PE1 的处理器间中断请求的操作示例。当 PE0 将 1B 写入 IPI0RCLRS[1] (= IPI0RCLR0[1]) 位时，IPI0REQ0[1] 位被清除。此外，由于 IPI0EN1 寄存器设置启用了来自 PE0 到 PE1 的处理器间中断请求，因此 IPI0FLG1[0] 也被清除。

当 PE2 将 1B 写入 IPI0RCLRS[1] (= IPI0RCLR2[1]) 位时，IPI0REQ2[1] 位被清除。在这里，由于 IPI0EN1 寄存器设置禁用了来自 PE2 到 PE1 的处理器间中断请求，因此 IPI0RCLR2 寄存器不影响 IPI0FLG1 寄存器的值。


##### 4 虚拟化启用时的使用
###### 4.1 概述

在本节中，VM 表示“主机或客户机”。

IPIR 旨在支持从一个 PE 到另一个 PE 的中断。更确切地说，它是从源 PE 到目标 PE 的 INTC1。IPIR 寄存器的位为每个 PE 而不是每个 VM 准备。当它用于从一个 VM 到另一个 VM 的中断时，需要另外两个设置。

目标 VM 使用 INTC1 的 EIBD 寄存器指定。源 VM 配置通过将不同的 SPID 分配给 VM 的保护功能间接实现。然后，通道的 PE 的寄存器只能由该 PE 的一个 VM 写入。非源 VM 的读取访问在正常使用中无害。IPIR 无法区分源 VM 到目标 VM。如果需要知道谁发送了中断，请通过某种方法发送 GPID 或避免时间相关的通道分配。

还可以使用 IPIR 进行 PE 内中断，即在同一 PE 中从一个 VM 到另一个 VM 的中断。




