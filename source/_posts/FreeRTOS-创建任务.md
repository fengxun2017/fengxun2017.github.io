---
title: FreeRTOS——创建任务
date: 2022/10/29
categories: 
- FreeRTOS
---
<center>

本文介绍，如何在FreeRTOS中创建一个任务，并设置合理的任务栈大小。

</center>

<!-- more -->

***

如何基于自己的开发板新建FreeRTOS工程可以参考：[基于crotex-m处理器新建FreeRTOS工程](https://fengxun2017.github.io/2022/10/25/%E5%9F%BA%E4%BA%8Ecrotex-m%E5%A4%84%E7%90%86%E5%99%A8%E6%96%B0%E5%BB%BAFreeRTOS%E5%B7%A5%E7%A8%8B/)

#### FreeRTOS如何创建任务：

在多任务操作系统中，每个任务都可以看做是一个独立的"小应用"，操作系统负责对各个任务进行调度。FreeRTOS对任务的函数原型定义如下：

`void ATaskFunction( void *pvParameters );`

即每个任务的入口函数必须为上面的形式(无RTOS的裸机程序，可以认为入口函数就是Main函数)。一般任务的入口函数实现形式如下：

```c
void ATaskFunction( void *pvParameters )
{

    for( ;; )
    {
        // 这里实现任务功能代码
    }

    // 任务退出前需要删除自己
    vTaskDelete( NULL );
}

```
嵌入式设备通常都是实现某个单一应用功能，当设备上电后，其程序就一直保持运行，直到设备断电。所以裸机开发时，都是main函数中实现一个无线循环，让其不停运行。

而到了RTOS上，应用的整体功能被拆分成多个任务，可以认为每个任务都是作为整体功能一部分的“小应用”。所以,一般情况下，这些任务也是一直运行的。因此，其实现也是内部一个无限的循环(如上述代码所示)。当然，也有一些情况下，某些任务不需要在运行了，这种情况下，我们可以让它跳出这个大循环，然后删除自己(上述代码，`NULL传入vTaskDelete`即表示删除调用这个函数的任务本身)。
ps：严格来说，是将自己放入等待结束队列中，RTOS自带的idle任务会负责实际的删除。因为任务无法删除自己，删除自己意味着要释放任务所占的任务控制块结构体信息，以及这个任务所拥有的栈，而任务执行代码又依赖于这些资源。

实现了任务函数后，还需要将其创建出来，即实例化出一个具体的任务。FreeRTOS提供了如下任务创建接口(该接口会动态申请任务需要的内核结构和任务需要的栈空间，不需要外部提供)：
```c
BaseType_t xTaskCreate( TaskFunction_t pvTaskCode, 
                        const char * const pcName, 
                        uint16_t usStackDepth, 
                        void *pvParameters, 
                        UBaseType_t uxPriority, 
                        TaskHandle_t *pxCreatedTask );
 ```
 其中：
 - `pvTaskCode：`即任务函数
 - `pcName：`可以给该任务命名，该名字是用来调试用的，FreeRTOS提供了一些辅助函数，可以获取系统中当前运行的任务的状态信息，通过这个名字可以更清楚的知道每个任务的具体作用。
 - `usStackDepth：`为任务栈大小(如果任务函数中没有很深的函数嵌套调用，任务函数内部也没有定义大的临时数组，则一般设置个200左右的值即可)。
 - `pvParameters：` 在任务的入口函数定义中也有一个参数`void ATaskFunction( void *pvParameters )`，任务创建时填入的该参数，在任务实际运行时就会传递给任务。
 - `uxPriority：`任务优先级，值越大，优先级越高。FreeRTOS会优先让优先级更高的任务运行（该值需要小于FreeRTOSConfig.h中配置的`configMAX_PRIORITIES`）。
 - `pxCreatedTask： `任务句柄，FreeRTOS会将该值设置为当前任务的句柄（即在系统内唯一标识一个任务），例如你的应用中存在任务A在某个条件下会删除任务B。那么，任务A就需要保存任务B的句柄，并且调用`vTaskDelete( TaskHandle_t pxTask )`时传入任务B的句柄，即可删除任务B。
- **返回值**：创建成功时返回pdPASS，其它值表示失败(例如，堆空间所剩的ram资源已经不足够创建任务的内核数据结构和栈空间了)。

任务创建成功后，就是启动FreeRTOS的内核，让其开始进行任务的调度。
即调用：`vTaskStartScheduler()`

如下是一个启动两个任务的demo，分别闪烁两个LED灯，两个任务都是使用的同一个入口函数，通过传递不同的LED参数使它们闪烁不同的LED。(虽然使用的是同一个入口函数，但是任务创建后它们是作为两个独立任务存在的)


``` c
include "FreeRTOS.h"
include "task.h"

void led_flash( void * pvParameters )
{
    const uint32_t LED_NUM = *(uint32_t *)pvParameters;
    const TickType_t xDelay = 500 / portTICK_PERIOD_MS;
    while(1) {
        vTaskDelay( xDelay );
        nrf_gpio_pin_toggle(LED_NUM);
    }
}

uint32_t LED_1 = 13, LED_2 = 14;
int main(void) 
{
    TaskHandle_t task_handle1=NULL, task_handle2=NULL;

    bsp_board_leds_init();

    xTaskCreate(led_flash,       
                "NAME",         
                200,    /* stack size */
                &LED_1,    
                2,
                &task_handle1);      

    xTaskCreate(led_flash,      
                "NAME",         
                200,     
                &LED_2,   
                2,
                &task_handle2);   


    /* Start the created tasks running. */
    vTaskStartScheduler();
    
    /* Execution will only reach here if there was insufficient heap to
    start the scheduler. */
    for( ;; );
    return 0;
}
```

**注意**：上述LED标记变量`uint32_t LED_1 = 13, LED_2 = 14;
`是定义成全局的，不能在main函数内部定义。因为，在调用`vTaskStartScheduler()`启动内核调度后，内核会重置main函数的栈以作它用。所以Main函数栈中的变量会变得无效。(FreeRTOS在启动内核调度后，**就永远不会再返回main函数中了**，只会调度创建的任务，所以FreeRTOS可以重置main的栈用来做其它用途)
**注意**：由于FreeRTOS在调用`vTaskStartScheduler`启动后就不会再返回到main函数中，所以也不要在Main函数中编写业务代码。main函数中基本只做一些必要的初始化工作，以及创建所有任务。


#### FreeRTOS 如何确定任务栈的大小：
初期经验不足时，可能不清楚创建任务时应该设置多大的任务栈比较合适（`xTaskCreate`的参数`usStackDepth`）。在调试阶段，我们可先设置一个比较大的值，例如如果你的任务函数内没有局部数组变量，也没有很深的函数嵌套函数调用，可以直接设置一个200。如果，你的任务函数中有大的局部数组（例如`char buffer[N]`），你可以设置栈大小为 N+200（当前前提是你的mcu中的确有那么多ram空间，没有的话就改小，先试试）。
之后，再利用FreeRTOS内核提供的 `UBaseType_t uxTaskGetStackHighWaterMark( TaskHandle_t xTask )` 函数来获取函数运行过程中，栈的使用量。
**注意**，使用该函数需要在头文件`FreeRTOSConfig.h`中增加配置`#define INCLUDE_uxTaskGetStackHighWaterMark 1`


任务栈使用量的检测原理简单解释如下：
例如，任务A创建时设置的栈大小为200，则内核会为任务A申请了一个 200 字大小的内存空间作为栈（32位处理器中**一个字是4字节**，所以实际申请的是**200*4字节**空间）。在初始化任务信息时，内核会将该段RAM空间设置为一个**特殊值(0xA5)**,假设该段内存起始地址为0，结束地址为799，栈增长时地址会增加（这里是为了解释方便，实际上cortex-m3/m4硬件都是"满减栈"，跟这里的描述是相反的）
初始时，栈内存状态如下，0-799地址空间都是0xA5：
|0|1|2|3|4|...|791|792|793|794|795|796|797|798|799|
| :----:| :---: | :----: | :----:| :---: | :----: | :----:| :---: | :----: | :----:| :---: | :----: | :----:| :---: | :----: |
|0xA5|0xA5|0xA5|0xA5|0xA5|...|0xA5|0xA5|0xA5|0xA5|0xA5|0xA5|0xA5|0xA5|0xA5|

程序运行一段时间后，最大时曾经使用到过791地址处，则内存状态如下(x表示其它值)：
|0|1|2|3|4|...|791|792|793|794|795|796|797|798|799|
| :----:| :---: | :----: | :----:| :---: | :----: | :----:| :---: | :----: | :----:| :---: | :----: | :----:| :---: | :----: |
|x|x|x|x|x|...|x|0xA5|0xA5|0xA5|0xA5|0xA5|0xA5|0xA5|0xA5|

FreeRTOS提供的函数`uxTaskGetStackHighWaterMark`就是从799地址处往前检查，看还有多少值保持为0x5A，这里例子为8，则会返回2（返回的是字个数）这样曾经使用过的最大深度就是(200-2)，根据该值就可以将创建任务时的栈大小设置为一个更合适的值。

一个使用demo如下，`SEGGER_RTT_printf`替换成自己的调试输出函数：
```c
void test_task( void * pvParameters )
{
    UBaseType_t uxHighWaterMark;
    const TickType_t xDelay = 1000 / portTICK_PERIOD_MS;
    
    while(1) {
        vTaskDelay( xDelay );
        uxHighWaterMark = uxTaskGetStackHighWaterMark( NULL );
        SEGGER_RTT_printf(0, "The maximum stack size ever used:%d\r\n", 200-uxHighWaterMark);
    }
}

int main(void) {

    xTaskCreate(test_task,       
                "NAME",         
                200,      
                NULL,    
                2,
                NULL);      
    
    vTaskStartScheduler();
    for( ;; );
    return 0;    
}
```
我的调试信息显示如下，所以设置200大小的栈是完全没必要的：
`The maximum stack size ever used:62`

**最后：** `uxTaskGetStackHighWaterMark`函数应该在开发阶段使用，正式发布的产品还是不要再使用了，毕竟每次调用都会遍历内存检查值。


<br/>
ps：需要注意文章代码中的日志输出函数，产品代码中如果需要使用的话，需要考虑线程安全性（多任务安全性），因为中断/任务切换可能发生在另一个任务正在输出日志但还未输出完的时候，这就可能造成日志错乱

<br/>
<br/>
FreeRTOS交流QQ群-663806972
