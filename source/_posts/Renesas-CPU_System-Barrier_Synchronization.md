---
title: Renesas-RH850  CPU System - Barrier-Synchronization
date: 2024/12/30
categories: 
- Renesas
---

<center>

Renesas学习笔记
CPU System : Barrier-Synchronization
</center>

<!-- more -->

***

##### 1 Barrier-Synchronization Overview
在使用多核处理器进行并行处理时，可能需要使核心等待，直到下一处理所需的数据准备就绪，处理才能继续。通常，设置一个屏障以防止在所有相关核心完成预定处理之前启动下一处理的控制称为屏障同步。仅使用软件可以实现屏障同步，但这可能会因为来自各个核心的屏障到达通知和各个核心的内存访问而降低系统处理性能。

屏障同步寄存器通过接收来自各个 PE 的完成通知检测所有相关核心是否完成了所需处理，同时提供自动清除来自各个 PE 的完成通知以准备下一个屏障同步的屏障同步支持功能，从而实现轻松的屏障同步。此功能减少了屏障同步的内存访问次数，从而最大限度地减少了屏障同步对处理性能的负面影响。此功能还可用于形成集群的 PE 之间的屏障同步。屏障同步寄存器具有以下特点：

- 提供 16 通道屏障同步寄存器。
- 以使用相同代码实现所有核心的屏障同步。
- 系统内所有集群和所有 PE 都可以访问。
- 支持地址 EDC 功能
- 支持数据 ECC 功能。
- 支持防止未授权访问的保护功能。

##### 2 Barrier-Synchronization Registers

| 寄存器名称                                          | 寄存器符号   | 地址                                      | 访问大小       | 访问保护（IBG）   |
| ------------------------------------------------- | ---------- | ----------------------------------------- | ------------- | -------- | 
| 屏障同步 n<sup>*2</sup> 初始化寄存器                         | BRnINIT    | <BARR_base> + 000H + 010H × n             | 8             | BRGPROT0_n | —     |
| 屏障同步 n<sup>*2</sup> 屏障参与 PE 设置寄存器                | BRnEN      | <BARR_base> + 004H + 010H × n             | 8             | BRGPROT0_n | —     |
| 屏障同步 n<sup>*2</sup> 屏障检查寄存器 self<sup>*1</sup>                | BRnCHKS    | <BARR_base> + 100H + 010H × n             | 8             | BRGPROT0_n | —     |
| 屏障同步 n<sup>*2</sup> 屏障同步完成寄存器 self<sup>*1</sup>            | BRnSYNCS   | <BARR_base> + 104H + 010H × n             | 8             | BRGPROT0_n | —     |
| 屏障同步 n<sup>*2</sup> 屏障检查寄存器 m<sup>*3</sup>                   | BRnCHKm    | <BARR_base> + 800H + 010H × n + 100H × m  | 8             | BRGPROT0_16 | —    |
| 屏障同步 n<sup>*2</sup> 屏障同步完成寄存器 m<sup>*3</sup>               | BRnSYNCm   | <BARR_base> + 804H + 010H × n + 100H × m  | 8             | BRGPROT0_16 | —    |

注意：
- *1：当 PE 访问 self 寄存器时，PE 可以访问每个 PE 对应的寄存器。例如，当 PE1 访问 BR0CHKS 寄存器时，PE1 可以访问 BR0CHK1 寄存器。
- *2：n = 0 到 15。n 是屏障同步通道号。
- *3：m = 0 到 3。m 是 PEID 的值。

自身区域
自身区域包含以下两种类型的寄存器：

BRnCHKS — 屏障同步 n 通道- 屏障检查自寄存器

BRnSYNCS — 屏障同步 n 通道- 屏障同步完成自寄存器

self 寄存器是虚拟寄存器，物理上不存在。从每个 PE 到 self 寄存器的访问被路由到与访问源 PE 对应的实际寄存器。下表列出了访问源 PE 和路由目标寄存器。有关每个寄存器位的功能，请参阅路由目标寄存器的规格。当除了 PEx 以外的主设备访问 self 寄存器时，寄存器返回 0，写访问被忽略，并会通知错误响应。

**基本假设是当 PE 使用屏障同步功能时，从 PE 通过 self 区域访问屏障同步寄存器。这使得 PE 可以使用相同的代码，因为不需要为每个 PE 指定不同的寄存器地址。**

也可以通过指定实际寄存器的地址而不使用 self 寄存器直接访问寄存器。在这种情况下，目的是检查另一个 PE 的寄存器状态，或通过调试工具引用各个寄存器。


| 访问源 PE | self 寄存器 | 路由目标寄存器     |
| -------- | ----------- | ------------------ |
| PE0      | BRnCHKS     | BRnCHK0            |
| PE1      | BRnCHKS     | BRnCHK1            |
| PE2      | BRnCHKS     | BRnCHK2            |
| PE3      | BRnCHKS     | BRnCHK3            |
| PE0      | BRnSYNCS    | BRnSYNC0           |
| PE1      | BRnSYNCS    | BRnSYNC1           |
| PE2      | BRnSYNCS    | BRnSYNC2           |
| PE3      | BRnSYNCS    | BRnSYNC3           |

###### 2.1 BRnINIT — 屏障同步 n 初始化寄存器
此寄存器初始化屏障同步通道 n（n = 0 到 15）。此寄存器清除通道 n 的 BRnCHKm 寄存器和 BRnSYNCm 寄存器。

| 位位置  | 位名称   | 功能                                                                                                 | 读/写 | 复位后的值 |
| ------ | ------- | ---------------------------------------------------------------------------------------------------- | ---- | ---------- |
| 7至1   | 保留     | 写入时写入复位后的值。                                                                                 | R    | 0          |
| 0      | BRINIT  | 屏障同步 n 初始化<br>将 1 写入此位会清除同一通道的 BRnCHK0 到 3 寄存器和 BRnSYNC0 到 3 寄存器。此位在读取时始终返回 0。 | W    | 0          |

###### 2.2 BRnEN — 屏障同步 n 屏障参与 PE 设置寄存器
此寄存器使 PE 能够参与通道 n 的屏障同步。

| 位位置  | 位名称   | 功能                                                                                               | 读/写 | 复位后的值 |
| ------ | ------- | ---------------------------------------------------------------------------------------------------- | ---- | ---------- |
| 7至4   | 保留     | 读取时返回复位后的值。写入时写入复位后的值。                                                          | R    | 0          |
| 3至0   | BREN[3:0]| 屏障同步通道 n 的参与使能<br>将 1 写入 x-th 位使能 PEx 的屏障同步支持功能。将 0 写入 x-th 位禁用 PEx 的屏障同步支持功能。 | R/W  | 0          |


###### 2.3 BRnCHKm — 屏障同步 n 屏障检查寄存器 m
此寄存器用于 PEm 通知信息：PEm 已到达通道 n 的屏障。到达屏障的 PE 将写入此寄存器的 BRCHK 位。

| 位位置  | 位名称   | 功能                                                                                                 | 读/写 | 复位后的值 |
| ------ | ------- | ---------------------------------------------------------------------------------------------------- | ---- | ---------- |
| 7至1   | 保留     | 读取时返回复位后的值。写入时写入复位后的值。                                                          | R    | 0          |
| 0      | BRCHK   | 通道 n 的屏障检查位<br>此位可以通过软件和硬件更新。<br>**软件**：写入 BRnCHKm 寄存器时，无论写入值为何，此位的值都变为 1。即使 BRnEN.BRENm 位为 0，此位也可以设置，但不会影响通道 n 的屏障同步。<br>**硬件**：当所有参与 PE（由 BRnEN 寄存器启用）的 BRnCHKm.BRCHK 位都设置时，硬件将清除此位。同时，所有参与 PE 的 BRnSYNCm.BRSYNC 位将设置。当写入 1 到 BRnINIT.BRINIT 位时，此位将被清除。 | R/W  | 0          |

注意：如果 BRnEN 寄存器的所有位都为 0（无 PE 参与屏障同步），BRCHK 位无法设置。

###### 2.4 BRnSYNCm — 屏障同步 n 屏障同步完成寄存器 m
此寄存器指示参与通道 n 屏障的所有 PE 已到达通道 n 的屏障。

| 位位置  | 位名称   | 功能                                                                                                 | 读/写 | 复位后的值 |
| ------ | ------- | ---------------------------------------------------------------------------------------------------- | ---- | ---------- |
| 7至1   | 保留     | 读取时返回复位后的值。写入时写入复位后的值。                                                          | R    | 0          |
| 0      | BRSYNC  | 屏障同步完成<br>此位可以通过软件和硬件更新。<br>**软件**：写入 BRnSYNCm 寄存器时，此位会根据写入的数据更新。<br>**硬件**：如果 BRnEN.BRENm 位为 1，当所有参与 PE 的 BRnCHKm.BRCHK 位都设置时，屏障同步建立，此位会自动设置。如果 BRnEN.BRENm 位为 0，此位不会由硬件更新。写入 1 到 BRnINIT.BRINIT 位将清除此位。 | R/W  | 0          |







##### 3 Barrier-Synchronization Function

###### 3.1 初始设置
在使用屏障同步寄存器之前，必须在 BRnEN 寄存器中进行初始设置。

首先，为清除先前操作的设置，通过写操作清除 BR0EN 寄存器。清除 BR0EN 寄存器后，将 07H 写入 BR0EN 寄存器，以使 PE0、PE1 和 PE2 参与屏障同步。最后，PE0 将 1B 写入 BR0INIT 寄存器，以清除 BR0CHK0 到 3 和 BR0SYNC0 到 3 寄存器。在完成参与 PE 设置寄存器和初始化设置后，可以使用屏障同步寄存器进行屏障同步。


###### 3.2 启用虚拟化时的用法

屏障同步设计用于支持同时运行的虚拟机的同步。在此架构中，通过分时复用实现虚拟化。因此，屏障同步不能用于在同一个 PE 上同步多个虚拟机。

屏障同步寄存器的位为每个 PE 准备，而不是每个虚拟机。如果允许 PE 参与同步通道，屏障同步无法区分参与的虚拟机和不参与的虚拟机。为避免这种情况，请使用保护功能并为虚拟机分配不同的 SPID。然后，同一个通道的 PE 寄存器只能由一个虚拟机写入。在正常使用中，不参与的虚拟机进行读取访问不会造成危害。
