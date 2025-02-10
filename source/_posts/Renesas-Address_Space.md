---
title: Renesas-RH850  Address Space
date: 2025/1/19
categories: 
- Renesas
---

<center>

Renesas学习笔记
Address Space

</center>

<!-- more -->

***

##### 1 Overview

16M code flash：0000 0000<sub>H</sub> -  00FF FFFF<sub>H</sub>
data flash:512 KB + 64 KB (dedicated to ICUMHA)：FF20 0000<sub>H</sub> - FF28 FFFF<sub>H</sub>

 Local RAM (CPU3)：FD60 0000<sub>H</sub> - FD60 FFFF<sub>H</sub>
 Local RAM (CPU2)：FD80 0000<sub>H</sub> - FD80 FFFF<sub>H</sub> 
 Local RAM (CPU1)：FDA0 0000<sub>H</sub> - FDA0 FFFF<sub>H</sub> 
 Local RAM (CPU0)：FDC0 0000<sub>H</sub> - FDC0 FFFF<sub>H</sub>
Local RAM (self)：FDE0 0000<sub>H</sub> - FDE0 FFFF<sub>H</sub>

Cluster RAM (Cluster0)：FE00 0000<sub>H</sub> - FE07 FFFF<sub>H</sub>   （512K）
Cluster RAM (Cluster1)：FE10 0000<sub>H</sub> - FE17 FFFF<sub>H</sub>   （512K）
Cluster RAM (Cluster2)： FE40 0000<sub>H</sub> - FE5F FFFF<sub>H</sub>      （2M）
Cluster RAM (Cluster3) (Retention RAM)：FE80 0000<sub>H</sub>  - FE83 FFFF<sub>H</sub>  (256k)



##### 2 各总线主设备视图中的地址空间 
###### 2.1 指令可获取的空间
CPU的指令可以从代码闪存、本地RAM和集群RAM中获取。
CPU的复位向量（RBASE初始值）：
- 在CPU0的用户启动模式下，其首地址为0800 0000H。
- 在CPUn（n: 1-3）的用户启动模式下，其首地址与正常操作模式相同。
- 在CPUn（n: 0-3）的正常操作模式下，其首地址可以通过复位向量PEn（OPT_RBASEn）在代码闪存（用户区域）内设置。
如果代码闪存在双映射模式下，它可以在有效区域内选择。 （用户区域的大小取决于产品。有关各变体的代码闪存映射，请参见第51.3.1节。有关OPT_RBASEn的设置，请参见第51.12节，配置设置区域（选项字节、复位向量）。

注意：有关指令代码的分配，请参见第3.9.6节，关于预取的使用注意事项。

###### 2.2 CPU可访问的数据空间

注意：在以下目标数据空间内的数据访问按程序顺序操作：

- 代码闪存
- H总线区域组0、1、2、3
- 调试（CPU0、CPU1、CPU2、CPU3）
- Instrumentation RAM
- 集群ERAM（集群0、1）
- 全局仿真RAM
- 集群RAM（集群0、1、2、3）
- P总线区域组0、1、2H/2L、3、4、5、6H/6L、7、8、9
- I总线区域组0、1、2
- PU外设区域（自身、CPU0、CPU1、CPU2、CPU3）

在以下目标数据空间内的加载访问可能在完成之前的访问（加载或存储）之前操作。为了保证访问顺序，需要通过软件进行同步处理：

- 本地RAM（自身、CPU0、CPU1、CPU2、CPU3）

对于不同目标空间之间的数据访问，数据访问的顺序不保证与程序顺序相同。为了保证访问顺序，需要通过软件进行同步处理。

关于同步处理的详细信息，请参见第3.9.1节，存储指令完成和后续指令生成的同步，以及第3.9.2节，加载指令完成和后续指令生成的同步。


###### 2.3 H总线模块可访问的数据空间

注意：只有RHSIF可以访问H总线从模块。对于其他H总线主控，这个区域作为保留区域。[仅适用于U2A-EVA、U2A16和U2A8]




##### 3 访问未映射区域的错误通知
当任何主设备访问未分配重要资源的“未映射区域”时，将通过总线向访问主设备发出错误响应。有关对总线主设备的错误响应，请参见表 4.3。有关每个区域的错误响应的反应，请参见第 4.3.1 节到第 4.3.9 节。
