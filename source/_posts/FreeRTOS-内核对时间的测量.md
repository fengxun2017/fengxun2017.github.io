---
title: FreeRTOS-内核对时间的测量
date: 2022/11/08
categories: 
- FreeRTOS
---

<center>
FreeRTOS 内核的时间测量功能，是内核能够实现多任务调度的基础条件。例如，当一个高优先级A，任务调用Delay（1秒）后，内核调度器选择了另一个低优先级的任务B开始运行，那么内核就需要感知 1秒时间的流逝，从而在1秒后，再次调度 高优先级的任务A，让其恢复运行。
</center>

<!-- more -->

***
#### FreeRTOS是如何实现时间测量的

嵌入式操作系统的基本运行方式是，系统需要周期性地进入任务调度函数中，检查当前是否有任务需要切换为就绪态（例如，delay的时间到期了，等待某个事件超时了），以及是否需要进行任务切换（有更高优先级的任务处于就绪状态了）。
定时器中断天然具有周期性，所以操作系统内核将定时器中断作为系统运行的“心跳”（FreeRTOS称为 **tick 中断**），从而实现周期性的检查任务状态。对于 cortex-m 处理器，硬件中都存在一个SysTick定时器，所以FreeRTOS在cortex-m处理器上的实现都是使用systick定时器中断来作为内核运行的“心跳”。

`注意：内核并不是只在tick 中断中才进行任务状态检查和任务切换。在FreeRTOS的内核实现中，总是会尽可能的让处于最高优先级的就绪任务立刻运行，如果只在tick中断才检查是否需要进行任务切换，那么就可能存在一个比较大的延迟（一个tick中断处理刚结束，这时创建了一个新的最高优先级任务，那就得等到下个tick中断到来才能被调度）。所以，FreeRTOS内核中有多处地方会检查是否需要进行任务切换。例如，FreeRTOS的创建任务函数，在创建一个新任务时，如果新任务的优先级是最高的，那么就会立刻运行这个新创建的新任务（任务切换）。又如，有一个高优先级任务A 处于阻塞态，正在等待某个消息队列有数据，如果此时正在运行的低优先级任务B 向该消息队列中发送了一条消息，那么任务A 就会立刻被“唤醒”，并运行（任务切换，抢占了任务B的运行）。`

基于定时器实现**tick 中断**，那么就需要明确定义多久产生一次中断。内核对 tick 频率的定义，在 FreeRTOSConfig.h 文件中：
```c
#define configTICK_RATE_HZ		( ( TickType_t ) 100 )
```
如上所示，定义了 tick 中断频率为 100，即每秒产生 100次中断。那么就需要设置systick的硬件，让其每 10 ms (`1/configTICK_RATE_HZ 秒`) 产生一次中断。

定时器都需要一个时钟源来驱动，cortex-m 处理器上 FreeRTOS 内核默认使用MCU中的内核时钟作为 sysytick的时钟源，所以还需要明确MCU 的内核运行时钟（即CPU运行频率）。其定义同样在 FreeRTOSConfig.h 文件中，例如我的CPU运行频率是64M：
```c
#define configCPU_CLOCK_HZ			( ( unsigned long ) 64000000 )	 // 64M运行频率
```
根据定时器内部的计时原理，使用上述cpu运行时钟作为其输入源时，systick每1/64000000（`1/configCPU_CLOCK_HZ`）秒就会产生一次计数。

现在我们希望让systick 每 10 ms (`1/configTICK_RATE_HZ 秒`) 产生一次中断，就要配置 systick 每次计数到：
 (1/configTICK_RATE_HZ) / (1/configCPU_CLOCK_HZ) =  configCPU_CLOCK_HZ / configTICK_RATE_HZ
就产生一次中断。 该配置可以在 port.c 文件的 `vPortSetupTimerInterrupt`函数中找到。

`注意：cortex-m中的systick定时器，实际是一个递减计数器，即减到0时产生中断。上面只是为了方便描述成了递增计数器。此外systick的定时器是24 bit，所以最大值为2^24-1 ,所以configCPU_CLOCK_HZ / configTICK_RATE_HZ 的结果应该在这个范围内`

配置好了 systick 周期性产生中断，那么整个系统就有了一个基本的时间单位了，即`1/configCPU_CLOCK_HZ 秒`。所以，在FreeRTOS中提供的一些API，其中的时间参数，并不是以**秒**或**毫秒**等实际时间为单位，而是以tick（`1/configCPU_CLOCK_HZ 秒`）为单位的。例如，FreeRTOS 提供的延迟函数：
```c
void vTaskDelay( const TickType_t xTicksToDelay )
```
我们在使用该函数时，就需要将 实际的延迟时间转换成 tick 数。 在FreeRTOS中可以自己转换，也可以调用提供的宏，如下所示：
```c

// 延迟2秒，自己将时间转换成tick 数
const TickType_t delay = 2000 / portTICK_PERIOD_MS;

// 延迟2秒，直接使用FreeRTOS宏，优先使用这个宏
vTaskDelay( pdMS_TO_TICKS(2000))
```

其中 `portTICK_PERIOD_MS`定义在文件`portmacro.h`中，其定义为每个tick有少个ms：
```c
#define portTICK_PERIOD_MS    ( ( TickType_t ) 1000 / configTICK_RATE_HZ )
```

而 `pdMS_TO_TICKS` 定义在文件`projdefs.h`中，直接将ms数转换成tick数：
```c
#define pdMS_TO_TICKS( xTimeInMs )    ( ( TickType_t ) ( ( ( TickType_t ) ( xTimeInMs ) * ( TickType_t ) configTICK_RATE_HZ ) / ( TickType_t ) 1000U ) )
```

#### 总结
FreeRTOS 根据配置的两个宏定义`configTICK_RATE_HZ, configCPU_CLOCK_HZ`，设置 systick定时器（cortex-m处理器上）的相关定时器，使其以 **configTICK_RATE_HZ** 频率产生中断，并作为内核运作的“心跳”。
对于内核来说，其对时间的概念是以 **1/configTICK_RATE_HZ 秒** 为单位的。所以，配置的内核运行频率（configTICK_RATE_HZ）越高，内核能感知的时间精度就越高。
例如，configTICK_RATE_HZ = 100 时，内核感知的时间单位是10ms，让任务delay5ms（vTaskDelay( pdMS_TO_TICKS(5))）是不起作用的(函数不会生效)。

但另一方面，配置的内核运行频率越高，就会频繁的产生内核tick中断。如果你的系统对功耗敏感，并且设置了在任务没有事情可做时进入休眠状态。那么更高的tick中断，会让MCU频繁进出休眠，反而会消耗更多的功耗（该问题可以通过配置configUSE_TICKLESS_IDLE来缓解，但并不是所有硬件平台上都支持）。

所以系统运行的频率需要根据实际应用选择一个合适的值。多数情况下 100HZ 的内核tick 频率足够了，如果实际应用的正确运行依赖于高精确的时间测量（例如，收到某个“信号”后，必须在某个时间点（要求误差是几十us）做出一些操作），那么应该考虑使用另一个高精度的定时器，并将其硬件中断优先级设置高于FreeRTOS系统（FreeRTOS系统不能屏蔽该中断，中断发生时，无论FreeRTOS在干什么都会优先响应中断）。

<br/>
FreeRTOS交流QQ群-663806972