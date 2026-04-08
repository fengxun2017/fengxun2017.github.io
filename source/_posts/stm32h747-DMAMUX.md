---
title: stm32h747 DMAMUX
date: 2025/12/21
categories: 
- stm32
---

<center>
stm32h747  DMAMUX
</center>

<!--more-->

***

STM32 MCU 嵌入了**直接存储器访问（DMA**）控制器。DMA可以在外设请求或软件触发时执行面向块的数据传输。在传统STM32产品上，通道请求选择在DMA控制器内实现，控制器内带有面向给定通道的受限外设请求列表。软件应用程序不能自由地将任何外设请求映射到任何通道。

**DMA请求复用器（DMAMUX**）增强了STM32 DMA请求路由功能。DMAMUX增加了更多灵活性，可以提供完全动态的DMA外设请求映射。它可以实现外围设备与DMA控制器之间DMA请求线路的路由。


### 1 主要特性 
- 最多 16 路可编程 DMA 请求线输出 
- 最多 8 路DMA请求生成器 
- 最多 32 路DMA请求生成器触发输入
- 最多 16 路同步输入 

每个 DMA 请求生成器通道支持： 
- DMA 请求触发输入选择器 
- DMA 请求计数器
- 触发输入事件溢出标志 

每个 DMA 请求线复用器通道输出支持： 
- 最多 107 路外设请求输入 
- 1 路 DMA 请求输出
- 同步输入选择器
- DMA 请求计数器
- 同步输入事件溢出标志
- 1 路事件输出（用于 DMA 请求链式触发）

STM32H747 集成了两个 DMA 请求复用器实例： 
- **DMAMUX1**：用于 DMA1 和 DMA2（D2 域） 
- **DMAMUX2**：用于 BDMA（D3 域） 

| Feature | DMAMUX1 | DMAMUX2 | 
|-------------------------|---------|---------|
| Number of DMAMUX output request channels | 16 | 8 | 
| Number of DMAMUX request generator channels | 8 | 8 | 
| Number of DMAMUX request trigger inputs | 8 | 32 | 
| Number of DMAMUX synchronization inputs | 8 | 16 | 
| Number of DMAMUX peripheral request inputs | 107 | 12 | 

### 2 DMAMUX 映射关系 
**DMAMUX1 的资源映射是硬连线的**。用于 D2 域的 DMA1 和 DMA2
 - DMAMUX1 通道 0–7 连接到 DMA1 通道 0–7 
 - DMAMUX1 通道 8–15 连接到 DMA2 通道 0–7

**DMAMUX2 的资源映射也是硬连线的**。用于 D3 域的 BDMA
- DMAMUX2 通道 0–7 连接到 BDMA 通道 0–7

![](/stm32h747-DMAMUX/DMAMUX.png)
- dmamux_hclk： DMAMUX AHB clock
- dmamux_req_inx： DMAMUX DMA request line inputs from peripherals
- dmamux_trgx：DMAMUX DMA request triggers inputs (to request generator sub-block)
- dmamux_req_genx：DMAMUX request generator sub-block channels outputs
- dmamux_reqx：DMAMUX request multiplexer sub-block inputs (from peripheral requests and request generator channels)
- dmamux_syncx：DMAMUX synchronization inputs (to request multiplexer sub-block)
- dmamux_req_outx：DMAMUX requests outputs (to DMA controllers)
- dmamux_evtx：DMAMUX events outputs
- dmamux_ovr_it：DMAMUX overrun interrupts

### 3 DMAMUX Channels 概要 

DMAMUX Channel 定义：
- 一个 DMAMUX channel = 请求复用器通道。 
- 可选择输入源：外设请求线或 DMAMUX 请求生成器。
- 每个 DMAMUX channel **专门连接到一个 DMA 控制器的 channel/stream**。

配置流程 
1. 完全配置 DMA channel y（不使能 EN）。 
2. 完全配置对应的 DMAMUX channel x。 
3. 最后设置 DMA channel y 的 EN 位，启动传输。

### 4 DMAMUX 请求线复用器 
负责路由 DMA 请求/应答信号（DMA request lines）。 
每个请求线并行连接到所有 DMAMUX channel。
请求来源：外设 or DMAMUX 请求生成器
请求源选择方式：在 **DMAMUX_CxCR.DMAREQ_ID** 中配置请求 ID。 
   - `DMAREQ_ID = 0` → 无请求线选中（空值）。 
   - 非 0 值 → 对应某个外设请求。 

**注意**： 同一个非零 ID **不能同时分配给多个 DMAMUX channel**，除非保证对应 DMA channel 不会同时激活。 


**同步模式**：
![](/stm32h747-DMAMUX/dmamux-sync.png)
- 每个 DMAMUX channel 可独立启用同步模式： 
  - 通过 **DMAMUX_CxCR.SE** 位使能。 
  - 同步输入通过 **SYNC_ID** 字段选择。 
  - 边沿检测由 **SPOL[1:0]** 配置（上升/下降沿）。 
- 行为： 
  - 检测到同步边沿 → 将选中的请求线传递到输出。 
  - 若没有待处理请求线 → 同步事件被丢弃。
- 内部请求计数器： 
  - 每次 DMA 服务一个请求 → 计数器减 1。 
  - 计数器减到 0 → 自动重新加载 **NBREQ** 值，并断开输入请求线。 
  - 输出请求次数 = `NBREQ + 1`。 

**事件生成**： 
![](/stm32h747-DMAMUX/dmamux-event-gen.png)
  - 通过 **EGE** 位使能。 
  - 当计数器重新加载时，产生一个 AHB 时钟周期的事件脉冲。 
  - 特殊情况：若 `NBREQ=0` 且 EGE=1 → 每次请求都会生成事件。 

**备注**：对于DMAMUX_CCR 寄存器中的 NBREQ，当使能了同步模式，NBREQ 影响发生同步事件后可以传递的dma请求数；当使能了事件生成，NBREQ 影响几次DMA触发产生一个事件。


同步溢出与中断 
- 若在计数器未减到 0 前再次检测到同步事件 → 设置 **SOFx** 溢出标志。
- 必须在 DMA channel 使用完成后关闭同步（SE=0），否则会因无 DMA 应答导致溢出。 
- 溢出标志清除：写 **CSOFx** 位于 DMAMUX_CFR。 
- 若 **SOIE** 使能 → 溢出时产生中断。




### 5 DMAMUX Request Generator

基本功能：
- **作用**：根据触发输入信号产生 DMA 请求。
- **结构**：
  - 有多个请求生成器通道。
  - 所有触发输入并行连接到所有生成器通道。
  - 每个生成器通道的输出作为 DMAMUX 请求线复用器的输入。


通道配置：
- 每个生成器通道 x 有一个 **GE (Generator Enable)** 位，位于 `DMAMUX_RGxCR`。
- 触发输入信号通过 **SIG_ID** 字段选择。
- 触发边沿类型通过 **GPOL** 字段选择：
  - 上升沿
  - 下降沿
  - 双边沿


工作机制：
- 检测到触发事件 → 通道开始产生 DMA 请求。
- 每次 DMA 控制器服务一个请求 → 内部请求计数器减 1。
- 当计数器减到 0 (underrun)：
  - 通道停止产生请求。
  - 计数器在下一次触发事件时自动重新加载为 **GNBREQ**。
- **请求数量**：每次触发事件后产生的请求数 = `GNBREQ + 1`。


溢出与中断：
- 若在计数器未减到 0 前再次检测到触发事件 → 设置 **OFx 溢出标志** (`DMAMUX_RGSR`)。
- 使用完成后必须禁用通道 (GE=0)，否则会因无 DMA 应答导致溢出。
- 清除溢出标志：写 **COFx** 位于 `DMAMUX_RGCFR`。
- 若 **OIE** 使能 → 溢出时产生中断。


总结：
- **DMAMUX Request Generator** = 触发输入 → 产生 DMA 请求。  
- 配置：`GE` 启用，`SIG_ID` 选择输入，`GPOL` 选择边沿，`GNBREQ` 设置请求数。  
- 行为：触发事件 → 产生 `GNBREQ+1` 个请求 → 计数器减到 0 → 停止，等待下一次触发。  



### 参考
[1] STM32H7xx Reference Manual, RM0399
