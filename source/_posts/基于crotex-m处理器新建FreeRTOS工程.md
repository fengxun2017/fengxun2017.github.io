---
title: 基于crotex-m处理器新建FreeRTOS工程
date: 2022/10/25
categories: 
- FreeRTOS
---
<center>

**前提**：首先需要基于自己的开发板实现一个点灯的程序，确保工程编译通过，烧录后程序能正常运行。
</center>

<!-- more -->

***

#### 提取FreeRtos必要文件
1. FreeRtos[官网](https://freertos.org/a00104.html)下载源码
2. 自己工程里新建一个目录用来保存提取的FreeRtos核心必要文件
3. 提取如下文件保存到自己新建的文件夹中
    - 3.1 FreeRTOS/Source/tasks.c <font color=#008000>// 包含FreeRtos核心功能和数据，如任务的创建、调度。</font>
    - 3.2 FreeRTOS/Source/list.c <font color=#008000>// tasks.c中的任务就绪任务列表、阻塞任务列表都是基于 list.c中的连接数据结构来实现的。</font>
    - 3.3 FreeRTOS/Source/queue.c <font color=#008000>// 提供基础的队列应用功能，同时freertos的信号量、互斥量也是基于队列来实现的。</font>
    - 3.4 FreeRTOS/Source/portable/[compiler]/[architecture]/port.c. <font color=#008000>// 与实际MCU和使用的编译工具相关的文件，例如使用keil工具和crotex-m4的mcu时可以选择：/RVDS/ARM_CM4F下的文件(port.c portmacro.h)</font>
    <font color=#008000>如果是较新的Keil工具，使用了V6版本的编译工具，由于和旧版本的汇编不兼容，需要替换成/GCC/ARM_CM4下的文件。(实际应该使用ARMClang目录下的，但这个目录下的文件指明应该使用/GCC目录下的，即新版本编译器是兼容GCC语法的)</font>

    - 3.5 FreeRTOS/Source/portable/MemMang/heap_3.c <font color=#008000>// FreeRtos内核默认创建系统相关数据结构时会使用动态内存分配，因此需要提供一个动态内存分配的算法，这里直接使用heap_3.c，即内容就是对 c标准库 malloc、free的封装。熟悉后可以根据自己的应用场景选择FreeRtos提供的其它合适方案</font>
    -  3.6 拷贝文件夹FreeRTOS/Source/include 到自己的工程中，并将其添加到工程的头文件目录配置中。
    -  前面 3.4 拷贝的 portmacro.h 文件所在目录也要添加到工程头文件目录配置中。
<br/>

#### FreeRtos功能配置文件FreeRTOSConfig.h
移植完上面的文件，就搭建好了FreeRtos的源码环境。但是想要实际运行起来，还需要一个应用配置文件，配置一些全局参数。FreeRtos的源码中有很多Demo都已配置好FreeRTOSConfig.h文件，一般找一个同系列的芯片配置，进行小改动下就能用了。

例如，FreeRtos的任务调度周期需要设置多少，即多久产生一时钟中断，该中断即系统的时钟滴答(tick)，是rtos能运行的核心。例如，当调用vTaskDelay 函数时，当前任务即进入阻塞态，那么rtos如何知道何时唤醒这个被阻塞的任务？就是基于tick中断来统计当前运行了多久，以及是否需要唤醒某个被阻塞的任务。

如下，将内核设置为 1秒中产生 10 次时钟滴答，即调度周期为100ms：
`#define configTICK_RATE_HZ			( ( TickType_t ) 10 )`
内核的时钟滴答越快，精度越高(例如FreeRtos提供的vTaskDelay的精度)，但由于频繁产生中断并执行内核调度过程，功耗也会相应增加，所以需要更具实际应用选择一个合适的tick频率。

那么设置了 `configTICK_RATE_HZ=10` 后如何就能让系统没 100ms 产生一次中断？
crotex-m系列的内核都自带SysTick定时器，FreeRtos默认使用该定时器来产生定时中断来作为内核的tick，FreeRtos默认配置使用内核时钟作为时钟源的计数器，所以知道了MCU的运行频率，FreeRtos就可以计算出，应该给SysTick计数器配置成多少值，才能让其每100ms产生一次中断。例如，我的MCU运行在 64M:
`#define configCPU_CLOCK_HZ			( ( unsigned long ) 64000000 )	`
具体如何根据`configTICK_RATE_HZ`和`configCPU_CLOCK_HZ`来配置SysTick，可以在前面移植的文件port.c中的vPortSetupTimerInterrup 函数中查看。

另一个需要配置的重要参数：
`#define configPRIO_BITS       		3 `
该值定义了几个bit来进行优先级分组配置（cortex-m3/4内核定义的优先级寄存器是具有8 bit的，但是芯片厂商是可以自己决定使用更少的bit来简化芯片和降低功耗，一般芯片厂商只会用3 bit或4 bit）。所以这个值需要根据你使用的MCU来决定(芯片手册里肯定能找到相关描述)。

此外，FreeRTOS默认配置是抢占式调度系统，即优先级高的任务会立即抢占优先级低的任务，这里的优先级指的是FreeRTOS自己定义的优先级，FreeRTOS一共可以有的优先级是通过`configMAX_PRIORITIES`来配置的，即创建任务时允许的优先级范围是[0,configMAX_PRIORITIES-1]，数值越大，优先级越高，越优先被调度。如果有多个任务的优先级相同，且处于当前最高优先级，FreeRTOS默认是会轮流调度它们(FreeRTOS的相同优先级时间片调度配置项`configUSE_TIME_SLICING`默认配置就是1)。

我们的样例需要闪烁LED等，需要使用FreeRtos的延迟函数，所以还需要添加如下定义:
`#define INCLUDE_vTaskDelay				1`

其余值使用默认即可，例如一个样例配置文件FreeRTOSConfig.h：
```c
#include "SEGGER_RTT.h"
#define configUSE_PREEMPTION		1
#define configUSE_IDLE_HOOK			0
#define configUSE_TICK_HOOK			0
#define configCPU_CLOCK_HZ			( ( unsigned long ) 64000000 )	
#define configTICK_RATE_HZ			( ( TickType_t ) 10 )
#define configMAX_PRIORITIES		( 5 )
#define configMINIMAL_STACK_SIZE	( ( unsigned short ) 128 )
#define configTOTAL_HEAP_SIZE		( ( size_t ) ( 10 * 1024 ) )
#define configMAX_TASK_NAME_LEN		( 16 )
#define configUSE_TRACE_FACILITY	0
#define configUSE_16_BIT_TICKS		0
#define configIDLE_SHOULD_YIELD		1

/* Co-routine definitions. */
#define configUSE_CO_ROUTINES 		0
#define configMAX_CO_ROUTINE_PRIORITIES ( 2 )

/* Software timer definitions. */
#define configUSE_TIMERS				1
#define configTIMER_TASK_PRIORITY		( 2 )
#define configTIMER_QUEUE_LENGTH		10
#define configTIMER_TASK_STACK_DEPTH	( configMINIMAL_STACK_SIZE * 2 )

/* Set the following definitions to 1 to include the API function, or zero
to exclude the API function. */
#define INCLUDE_vTaskPrioritySet		1
#define INCLUDE_uxTaskPriorityGet		1
#define INCLUDE_vTaskDelete				1
#define INCLUDE_vTaskCleanUpResources	1
#define INCLUDE_vTaskSuspend			1
#define INCLUDE_vTaskDelayUntil			1
#define INCLUDE_vTaskDelay				1
#define INCLUDE_xTaskGetCurrentTaskHandle 1
/* Cortex-M specific definitions. */
#ifdef __NVIC_PRIO_BITS
	/* __BVIC_PRIO_BITS will be specified when CMSIS is being used. */
	#define configPRIO_BITS       		__NVIC_PRIO_BITS
#else
	#define configPRIO_BITS       		3        /* 15 priority levels */
#endif

/* The lowest interrupt priority that can be used in a call to a "set priority"
function. */
#define configLIBRARY_LOWEST_INTERRUPT_PRIORITY			0xf

/* The highest interrupt priority that can be used by any interrupt service
routine that makes calls to interrupt safe FreeRTOS API functions.  DO NOT CALL
INTERRUPT SAFE FREERTOS API FUNCTIONS FROM ANY INTERRUPT THAT HAS A HIGHER
PRIORITY THAN THIS! (higher priorities are lower numeric values. */
#define configLIBRARY_MAX_SYSCALL_INTERRUPT_PRIORITY	5

/* Interrupt priorities used by the kernel port layer itself.  These are generic
to all Cortex-M ports, and do not rely on any particular library functions. */
#define configKERNEL_INTERRUPT_PRIORITY 		( configLIBRARY_LOWEST_INTERRUPT_PRIORITY << (8 - configPRIO_BITS) )
/* !!!! configMAX_SYSCALL_INTERRUPT_PRIORITY must not be set to zero !!!!
See http://www.FreeRTOS.org/RTOS-Cortex-M3-M4.html. */
#define configMAX_SYSCALL_INTERRUPT_PRIORITY 	( configLIBRARY_MAX_SYSCALL_INTERRUPT_PRIORITY << (8 - configPRIO_BITS) )


/* Definitions that map the FreeRTOS port interrupt handlers to their CMSIS
standard names. */
#define vPortSVCHandler SVC_Handler
#define xPortPendSVHandler PendSV_Handler
#define xPortSysTickHandler SysTick_Handler

extern void vAssertCalled( const char * pcFile, uint32_t ulLine );
/* Normal assert() semantics without relying on the provision of an assert.h
header file. */
#define configASSERT( x ) if( ( x ) == 0 )	vAssertCalled( __FILE__, __LINE__ )

```

**注意**：对于cortex-m处理器，一般启动文件(工程中的那个startup.s汇编文件，一般都是芯片厂商提供的)中都是默认使用CMSIS标准的名称，如`SysTick_Handler`，而Port.c文件中函数定义的名字用的是`xPortSysTickHandler`，因此需要在配置文件FreeRTOSConfig.h中使用`#define` 将其改名:

```c
#define vPortSVCHandler SVC_Handler
#define xPortPendSVHandler PendSV_Handler
#define xPortSysTickHandler SysTick_Handler
```
<br/>

#### 输出调试信息
在前面的FreeRTOSConfig.h中我们重定义了断言函数，FreeRtos在运行时会有充分的配置和参数检查，在调试阶段打开这个断言函数，可以快速定位一些配置和参数不正确的问题。
```c
extern void vAssertCalled( const char * pcFile, uint32_t ulLine );
#define configASSERT( x ) if( ( x ) == 0 )  vAssertCalled( __FILE__, __LINE__ )
```
`vAssertCalled( const char * pcFile, uint32_t ulLine )`的具体实现可以使用任意的工具(例如通过串口工具)将出现问题的函数所在的文件名和行号显示出来。这里，我使用了J-Link RTT工具来输出错误日志信息，J-Link RTT的使用可以参考这篇文章：[使用JLink-RTT输出调试信息](https://fengxun2017.github.io/2022/10/20/use-jlink-RTT/)

```c
void vAssertCalled( const char * pcFile, uint32_t ulLine ){
    SEGGER_RTT_printf(0, "error at:%s, line:%d\r\n", pcFile, ulLine);
    for(;;){}
}
```
<br/>

#### 实现一个翻转LED的测试程序
1. 首先确保自己的裸机LED翻转测试可以正常运行。
    <br/>
2. 创建每500ms翻转一个LED的任务：其中LED硬件相关代码使用自己的
    ```c
    void vTaskCode( void * pvParameters )
    {
        const TickType_t xDelay = 500 / portTICK_PERIOD_MS;
        bsp_board_leds_init();

        while(1) {
            for (uint32_t i = 0; i < LEDS_NUMBER; i++)
            {
                vTaskDelay( xDelay );
                nrf_gpio_pin_toggle(m_board_led_list[i]);
            }
        }
    }
    ```
    <br/>
3. 实现main函数：
   ```c
    #include <stdint.h>
    #include <stdbool.h>
    #include "FreeRTOS.h"
    #include "task.h"

    int main(void) {

        BaseType_t xReturned;
        TaskHandle_t xHandle = NULL;
        
        
        /* Create the task, storing the handle. */
        xReturned = xTaskCreate(
                        vTaskCode,       /* Function that implements the task. */
                        "NAME",          /* Text name for the task. */
                        configMINIMAL_STACK_SIZE,      /* Stack size in words, not bytes. */
                        NULL,    /* Parameter passed into the task. */
                        2,/* Priority at which the task is created. */
                        &xHandle );      /* Used to pass out the created task's handle. */
        
        /* Start the created tasks running. */
        vTaskStartScheduler();
        
        /* Execution will only reach here if there was insufficient heap to
        start the scheduler. */
        for( ;; );
        return 0;    
    }
   ```

烧录到开发板上，配置没有问题的话，这里应该就能看到自己开发板的LED闪烁了。

<br/>
FreeRTOS交流QQ群-663806972

<br/>
<br/>
<br/>
<br/>

参考链接：[https://freertos.org/Creating-a-new-FreeRTOS-project.html](https://freertos.org/Creating-a-new-FreeRTOS-project.html)