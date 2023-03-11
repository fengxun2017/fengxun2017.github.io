---
title: FreeRTOS-使用 task notification
date: 2023/2/13
categories: 
- FreeRTOS
---

<center>
本文介绍使用Task Notification，在一些常见开发场景中来替代二值信号量、计数信号量、event group、以及消息邮箱（存储单个uint32_t数值的邮箱），以提高系统运行效率。
</center>


<!--more-->

***
使用`Task Notification`需要在工程配置文件`FreeRTOSConfig.h`中定义:
```c
#define configUSE_TASK_NOTIFICATIONS	1
```
在文章[FreeRTOS-Task Notification](https://fengxun2017.github.io/2023/02/07/FreeRTOS-task-notification/)中，我们提到在FreeRTOS版本`FreeRTOSv202112`之后，内核任务控制块结构的`Task Notification`储存区，不再是仅能存储单个`uint32_t` 值，而是定义为一个可配置的数据 `uint32_t ulNotifiedValue [ configTASK_NOTIFICATION_ARRAY_ENTRIES ]`（默认配置大小为1）。

因此，如果有需求，可以将数组设置更大，使得`Task Notifacation`可以存储多个 `uint32_t`值，每个值用作不同的目的。

`Task Notification`提供的API，每次仅能读/写`ulNotifiedValue`数组中的单个值。因此，`Task Notification`的 API 中包含一个`index`参数用来指定操作数组中的哪个值。同时，为了和历史版本的`Task Notification`兼容，也保留了旧版本中不需要指定`index`参数的 API（因为旧版本中就是单个uint32_t值），它们操作`ulNotifiedValue`中的第 **0** 个值:

例如，两个发送数据相关的API:
xTaskNotify——默认是将数据写入到目标任务`ulNotifiedValue`数组中的第**0**个值。
xTaskNotifyIndexed——需要指定数据是写入目标任务`ulNotifiedValue`数组中的哪个值。

例如，两个获取数据相关的API：
ulTaskNotifyTake——默认是获取目标任务`ulNotifiedValue`数组中的第 **0** 个值。
ulTaskNotifyTakeIndexed——需要指定获取目标任务`ulNotifiedValue`数组中的哪个值。

在下文中，我们均使用不需要指定`index`参数的API，即操作`ulNotifiedValue`数组中的第 **0** 个值。 
后文中提及的`ulNotifiedValue`也默认指的是该数组的第 **0** 个值，而不是实际的数组。

#### 1 使用Task Notifacation 模拟二值信号量
[二值信号量](https://fengxun2017.github.io/2022/12/15/FreeRTOS-use-binary-semaphore/)的外在表现就是两种状态：
- **值为0**： 表示不可用，此时如果任务请求获取信号量，就会进入阻塞状态，等待其可用（当设置了等待超时时间）。
- **值为1**：表示可用。此时任务请求信号量时，可以立刻获得，并且信号量会变为不可用状态（值为0）



当使用`Task Notification`来模拟二值信号量时，API：
```c
uint32_t ulTaskNotifyTake( BaseType_t xClearCountOnExit, TickType_t xTicksToWait );
```
可以用来模拟任务**获取信号量**，实际上该函数内部实现是等待任务自身控制块中的`ulNotifiedValue`值**大于**0。调用该函数时如果`ulNotifiedValue`的值：
- **等于0**，可以认为当前“信号量不可用”，任务会阻塞并等待其值大于0（参数`xTicksToWait`为等待时间）。
- **大于0**，可以看做“信号量可用”了，该函数成功返回（等于拿到信号量），而二值信号量的特性是一旦被获取，就会再次变为“不可用状态”。为了实现该特性，该函数的参数`xClearCountOnExit`需要指定为`pdTRUE`，表示成功获取到非0值时，就将任务控制块中的`ulNotifiedValue`归零（变为“不可用状态”）。


<br/>

用来模拟**设置信号量**的 API为：
```c
BaseType_t xTaskNotifyGive( TaskHandle_t xTaskToNotify );

// 中断处理函数中调用时，需要使用中断版本：
void vTaskNotifyGiveFromISR( TaskHandle_t xTaskToNotify, BaseType_t *pxHigherPriorityTaskWoken );
```
- 参数`xTaskToNotify`：为目标任务句柄。该函数内部实现就是将目标任务控制块中的`ulNotifiedValue`值加1（非0，变为可用了）。如果目标任务当前正在等待“信号量可用”（等待`ulNotifiedValue`值**大于**0），那么调用该API 会使得目标任务恢复为就绪态。
- 参数`pxHigherPriorityTaskWoken`：作用见下文代码中的注释。

我们用上述两个API，来实现[二值信号量](https://fengxun2017.github.io/2022/12/15/FreeRTOS-use-binary-semaphore/)一文中的任务等待按键事件：

任务代码和main函数如下所示：
```c
#include <stdint.h>
#include <stdbool.h>
#include "task.h"
#include "FreeRTOS.h"
#include "SEGGER_RTT.h"

void button_pressed_handler( void *pvParameters ) {

    for(;;) {
        // 使用portMAX_DELAY，表示一直等待直到可用。
        // 使用portMAX_DELAY，需要在FreeRTOSConfig.h 文件中定义 INCLUDE_vTaskSuspend = 1
        // 正式产品代码最好使用一个超时值，这样异常发生时（中断源发生异常，没按预期触发），至少能获得超时错误，可以在超时错误中做一些恢复操作。
        ulTaskNotifyTake(pdTRUE, portMAX_DELAY);
        SEGGER_RTT_printf(0, "in task, do something\n");

    }
}


TaskHandle_t  pxCreatedTask;
int main(void) {
    
    bsp_init();
    
    // pxCreatedTask 保存创建的任务button_pressed_handler的句柄。
    // 向任务button_pressed_handler发送notification时，需要该值指明任务。
    if (pdPASS == xTaskCreate(button_pressed_handler, "task_a", 100, NULL, 1, &pxCreatedTask)){
        
        SEGGER_RTT_printf(0, "start FreeRTOS\n");
        vTaskStartScheduler();
    } 

    // 正常启动后不会运行到这里
    SEGGER_RTT_printf(0, "insufficient resource\n");

    for( ;; );
    return 0;    
}
```

按键中断处理函数中调用通知 API：`vTaskNotifyGiveFromISR`
```c

// 目标任务的句柄
extern TaskHandle_t  pxCreatedTask;

// 在自己硬件平台的按键中断处理函数中调用 vTaskNotifyGiveFromISR 即可
void GPIOTE_IRQHandler(void){

    BaseType_t higher_task_woken = pdFALSE;
    if ( NRF_GPIOTE->EVENTS_PORT == 1 ){

        //中断处理函数中要清除event,不然会导致一直产生中断
        NRF_GPIOTE->EVENTS_PORT = 0;    

        // 确认是按键按下，则设置信号量
        if(IS_BUTTON_PRESSED(BUTTON_1)) {
            vTaskNotifyGiveFromISR(pxCreatedTask, &higher_task_woken);
            SEGGER_RTT_printf(0, "in interrupt service function, button pressed\n");
        }
    }

    // 如果higher_task_woken=True，表示有更高优先级任务就绪了
    // 我们使用的是抢占式调度，只要有更高优先级的任务就绪，应该让其立刻运行。
    // 下面的代码就是，判断有更高优先级就绪时，就会设置任务切换中断，那么当前中断函数退出后，就会立刻触发任务切换，让最高优先级的就绪任务运行。
    portYIELD_FROM_ISR(higher_task_woken);
}

```
运行如下：
```
start FreeRTOS
in interrupt service function, button pressed
in task, do something
in interrupt service function, button pressed
in task, do something
```

#### 2 使用Task Notifacation 模拟计数信号量
二值信号量的特性是：非 “**0**” 即 “**1**” ，信号量一旦被获取，立刻又会变为不可用状态（“0”）。因此，我们在使用`ulTaskNotifyTake(  xClearCountOnExit,  xTicksToWait )`模拟获取信号量时，设置参数`xClearCountOnExit = pdTRUE`，使其实现“信号量被获取后就立刻变为不可用”。也是因为该特性，二值信号量不具备计数特性。

[计数信号量](https://fengxun2017.github.io/2022/12/17/FreeRTOS-use-counting-semaphore/)相比二值信号量多了计数功能。
使用`Task Notification`模拟计数信号量的方式和上文一样，只是在调用
```c
uint32_t ulTaskNotifyTake( BaseType_t xClearCountOnExit, TickType_t xTicksToWait );
```
时，设置参数`xClearCountOnExit = pdFALSE`，这样该函数的行为就是将目标任务控制块中的`ulNotifiedValue`值**减 1**（没有清零，如果“信号量”被设置多次，那么就可以调用对应次数的`ulTaskNotifyTake`，每次都能立刻获得到“信号量”）。

我们沿用文章[FreeRTOS-使用计数信号量](https://fengxun2017.github.io/2022/12/17/FreeRTOS-use-counting-semaphore/)中使用计数信号量来记录按键次数的例子。使用`Task Notification`模拟计数信号量，并记录按键次数。这样，任务即使因为忙，无法立即响应短时间内发生的多次按键，但由于事件发生的次数已经被记录，使得任务在忙完之后仍旧能够感知到每个发生的事件。

按键中断函数，以及main 函数均不用改动，沿用上文的代码。
按键处理任务修改如下：
```c
void button_pressed_handler( void *pvParameters ) {

    for(;;) {
        // 这里等待时间使用portMAX_DELAY，表示一直等待直到可用。
        // 使用portMAX_DELAY，需要在FreeRTOSConfig.h 文件中定义 INCLUDE_vTaskSuspend = 1
        // 正式产品代码最好使用一个超时值，这样异常发生时（中断源发生异常，没按预期触发），至少能获得超时错误，可以在超时错误中做一些恢复操作。
        
        // xClearCountOnExit = pdFALSE，使得每次获得“信号量”后，“信号量”的计数值仅递减
        ulTaskNotifyTake(pdFALSE, portMAX_DELAY);

        SEGGER_RTT_printf(0, "in task, do something\n");
        // 阻塞任务2秒，模拟在做其它工作，如果这期间发生了 n 次按键事件，事件的次数会记录在内部
        // 之后对应可以无阻塞的调用ulTaskNotifyTake n 次，使得每次事件都有对应的一次处理。  
        vTaskDelay(pdMS_TO_TICKS(2000));
    }
}
```

运行如下：在按键处理任务正在做其它工作时，这期间发生了三次按键，由于按键处理任务正在“忙”，无法立即响应。但事件发生的次数被记录下来了。并在之后，任务会依次处理每个发生过的事件。
```
start FreeRTOS
in interrupt service function, button pressed
in task, do something
in interrupt service function, button pressed
in task, do something
in interrupt service function, button pressed   #在按键处理任务正在做其它工作时，这期间发生了三次按键，并被记录下来。
in interrupt service function, button pressed
in interrupt service function, button pressed
in task, do something   #之后，根据记录的次数，调用相应次数的处理过程
in task, do something
in task, do something
```


#### 3 使用Task Notifacation 模拟 event group
在32位MCU中，[event group](https://fengxun2017.github.io/2023/01/13/FreeRTOS-event-group/)本质就是一个`uint32_t`数值，其高8位是内部使用的控制位，不可以使用。低24位开发者可以使用，开发者可以使用其中的任意一个位来表示一个自定义的事件。
任务可以等待某个事件发生（等待相应位被置位），等待多个事件同时发生（等待多个位同时被置位），或者等待多个事件中的任意一个事件发生（等待多个位中任意一个被置位）。

`Task Notification`的`ulNotifiedValue`也是一个`uint32_t`数值，我们也可以使用其中的位，表示一些自定义的事件，来实现模拟`event group`的功能。

使用`Task Notification`来模拟`event group`时，使用 API:
```c
BaseType_t xTaskNotifyWait( uint32_t ulBitsToClearOnEntry,
                            uint32_t ulBitsToClearOnExit,
                            uint32_t *pulNotificationValue,
                            TickType_t xTicksToWait );
```
可以实现“等待事件发生”功能。
其中：
- **ulBitsToClearOnEntry**：用来指定在开始等待前，哪些位需要清除。例如，应用中存在一个交易功能，在发起交易后，调用xTaskNotifyWait来等待按键事件。这种情况下设置ulBitsToClearOnEntry=1<<n（第n位表示按键事件），使得等待前确保按键事件位被清零，避免之前遗留的数据对这里按键事件造成误判。
- **ulBitsToClearOnExit**：当收到其它任务发送的notification数据后，任务解除阻塞恢复就绪，同时会自动清零任务控制块中`ulNotifiedValue`的那些由`ulBitsToClearOnExit`指定的位。（等待的事件发生之后，清除事件标记位，避免重复触发）
- **pulNotificationValue**：当函数返回时（收到数据或等待超时），会将任务控制块中当前的`ulBitsToClearOnExit`值保存在该参数中。（注意，是保存之后，才会根据第二个参数 ulBitsToClearOnExit 来清零需要清零的位）
- **xTicksToWait**：最长等待时间。
- **返回值**：返回`pdTRUE`表示收到数据，返回`pdFALSE`表示等待超时。

而设置事件，则通过`Task Notification`的发送 notification 数据来实现：
```c
BaseType_t xTaskNotify( TaskHandle_t xTaskToNotify, 
                        uint32_t ulValue, 
                        eNotifyAction eAction );
```
- **xTaskToNotify**：目标任务的句柄。
- **ulValue**：需要发送的值
- **eAction**：对于模拟`event group`来说，设置`eAction = eSetBits`，表示根据参数`ulValue`设置目标任务`ulNotifiedValue`中的相应位（表示这些位对应的自定义事件发生了）。
  

需要注意的是，`Task Notification`的等待数据 API：`xTaskNotifyWait`，在收到**任意数据**时后就会返回（解除阻塞状态），因此用该函数无法实现`event group`的等待多个事件同时发生才返回的特性。

我们使用`Task Notification`作为`event group`来实现下面的例子：
每当`task_a`完成一些特定工作，就会设置事件`TASK_A_DONE`
每当`task_b`完成一些特定工作，就会设置事件`TASK_B_DONE`
`task_c`等待`TASK_A_DONE`、`TASK_B_DONE`任意一个事件发生，则执行相关处理，代码如下所示：

任务代码：
```c
// 使用第 0 位来表示事件：task_a工作执行完
#define TASK_A_DONE_FLAG ( 1 << 0 )    
void task_a( void *pvParameters ) {
    TaskHandle_t  task_c = (TaskHandle_t)pvParameters;
    for(;;) {
        
        // 延迟1秒，模拟 task_a 在处理一些工作
        vTaskDelay(pdMS_TO_TICKS(1000));
        SEGGER_RTT_printf(0, "task_a done!\n");
        
        // 工作做完了，设置事件对应的位，通知task_c
        xTaskNotify(task_c, TASK_A_DONE_FLAG, eSetBits);
    }
}

// 使用第 1 位来表示事件：task_b工作执行完
#define TASK_B_DONE_FLAG ( 1 << 1 )    
void task_b( void *pvParameters ) {
    TaskHandle_t  task_c = (TaskHandle_t)pvParameters;
    for(;;) {
        
        // 延迟1秒，模拟 task_b 在处理一些工作
        vTaskDelay(pdMS_TO_TICKS(1000));
        SEGGER_RTT_printf(0, "task_b done!\n");
        
        // 工作做完了，设置事件对应的位，通知task_c
        xTaskNotify(task_c, TASK_B_DONE_FLAG, eSetBits);
    }
}


void task_c( void *pvParameters ) {
    
    uint32_t notification_value;
    BaseType_t ret;
    for(;;) {
        
        // 这里等待事件（task_a 或 task_b 完成工作）
        // task_c 会阻塞在该函数内部，直到事件发生，或者等待超过2秒
        ret = xTaskNotifyWait(0, TASK_A_DONE_FLAG|TASK_B_DONE_FLAG, &notification_value, pdMS_TO_TICKS(2000));
        if (pdTRUE == ret) {
            if (notification_value & TASK_A_DONE_FLAG) {
                SEGGER_RTT_printf(0, "in task_c: task_a done!\n");

                // do something
            }
            if (notification_value & TASK_B_DONE_FLAG) {
                SEGGER_RTT_printf(0, "in task_c: task_b done!\n");

                // do something
            }
        }
    }
}

```

main函数实现如下：
```c
#include <stdint.h>
#include <stdbool.h>
#include "task.h"
#include "FreeRTOS.h"
#include "SEGGER_RTT.h"


int main(void) {
    
    TaskHandle_t  taskc_handle;

    if (pdPASS == xTaskCreate(task_c, "task_c", 100, NULL, 1, &taskc_handle)
        && pdPASS == xTaskCreate(task_a, "task_a", 100, taskc_handle, 1, NULL)
        && pdPASS == xTaskCreate(task_b, "task_b", 100, taskc_handle, 1, NULL)){
        
        SEGGER_RTT_printf(0, "start FreeRTOS\n");
        vTaskStartScheduler();
    } 

    // 正常启动后不会运行到这里
    SEGGER_RTT_printf(0, "insufficient resource\n");

    for( ;; );
    return 0;    
}

```


运行结果如下所示：
```
start FreeRTOS
task_a done!
task_b done!
in task_c: task_a done!
in task_c: task_b done!
task_a done!
task_b done!
in task_c: task_a done!
in task_c: task_b done!
task_a done!
task_b done!
in task_c: task_a done!
in task_c: task_b done!
```

#### 使用Task Notifacation 作为一个轻量级消息邮箱
如上一节文所述，任务控制块中的`ulNotifiedValue`为一个`uint32_t`数值，我们可以将其作为一个4字节的存储空间，因此每次可以发送单个`uint32_t`数值，或发送4个`char`类型数值，使其成为一个轻量级的消息邮箱（消息直接发送给目标，不需要创建额外对象，非常适合在任务间传递数值，或几个字节的数据）。

对于发送notification的 API
```c
BaseType_t xTaskNotify( TaskHandle_t xTaskToNotify, 
                        uint32_t ulValue, 
                        eNotifyAction eAction );
```
其参数`eAction`除了可以设置为上一节所述为`eSetBits`，使其能单独设置某些位。
还可以设置为：
- **eIncrement**：让目标任务的`ulNotifiedValue`值自增1。实际上前文所述的`xTaskNotifyGive`就是一个宏，其本质就是使用`eAction= eIncrement`的`xTaskNotify`。
- **eSetValueWithOverwrite**：设置目标任务的`ulNotifiedValue`值为参数中的`ulValue`。
- **eSetValueWithoutOverwrite**：同上，但如果目标任务没有读取过上一次设置的值，则本次不会执行写入。

因此，发送Notification时，将`xTaskNotify`参数`eAction`设置为`eSetValueWithOverwrite`或`eSetValueWithoutOverwrite`即可用`Task Notification`作为轻量级邮箱使用。

如下所示样例:
创建任务`task_a`，周期性地向任务`task_b`发送数据。（例如周期采样某个传感器数据，发送给任务，如果是中断服务函数中向目标任务发送数据，需要使用发送函数的中断版本`xTaskNotifyFromISR`）
```c
#include <stdint.h>
#include <stdbool.h>
#include "task.h"
#include "FreeRTOS.h"
#include "SEGGER_RTT.h"


void task_a( void *pvParameters ) {
    TaskHandle_t  task_c = (TaskHandle_t)pvParameters;
    int i = 0;
    for(;;) {
        
        // 将 i 值发送给 task_b
        xTaskNotify(task_c, i++, eSetValueWithOverwrite);

        vTaskDelay(pdMS_TO_TICKS(2000));
    }
}



void task_b( void *pvParameters ) {
    
    uint32_t notification_value;
    BaseType_t ret;
    for(;;) {
        
        // task_b 会阻塞在该函数内部，直到收到数据，或者等待超过2秒
        // ulBitsToClearOnExit=0xffffffff，表示读取完ulNotifiedValue的之后，将其清零。
        ret = xTaskNotifyWait(0, 0xffffffff, &notification_value, pdMS_TO_TICKS(2000));
        if (pdTRUE == ret) {
            SEGGER_RTT_printf(0, "task_b recv: %u\n", notification_value);
        }
    }
}


int main(void) {
    
    bsp_init();
    TaskHandle_t  taskb_handle;

    if (pdPASS == xTaskCreate(task_b, "task_b", 100, NULL, 1, &taskb_handle)
        && pdPASS == xTaskCreate(task_a, "task_a", 100, taskb_handle, 1, NULL)){
        
        SEGGER_RTT_printf(0, "start FreeRTOS\n");
        vTaskStartScheduler();
    } 

    // 正常启动后不会运行到这里
    SEGGER_RTT_printf(0, "insufficient resource\n");

    for( ;; );
    return 0;    
}
```


运行输出如下：
```
start FreeRTOS
task_b recv: 0
task_b recv: 1
task_b recv: 2
task_b recv: 3
```

<br>
<br/>
ps：需要注意文章代码中的日志输出函数，产品代码中如果需要使用的话，需要考虑多线程安全性（多任务安全性），因为中断/任务切换可能发生在另一个任务正在输出日志但还未输出完的时候，这就可能造成日志错乱

<br/>
<br/>
FreeRTOS交流QQ群-663806972