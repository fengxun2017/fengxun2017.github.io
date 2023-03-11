---
title: FreeRTOS-使用消息队列
date: 2022/12/12
categories: 
- FreeRTOS
---

<center>
FreeRTOS提供了一个消息队列模块，通过消息队列，我们可以实现任务和任务间的数据传递；中断服务程序和任务间的数据传递。
</center>
<!--more-->

***
一些更详细的FreeRTOS消息队列内部实现细节，可以参考[FreeRTOS-消息队列内部细节](https://fengxun2017.github.io/2022/12/08/FreeRTOS-queue-internal-details/)

FreeRTOS提供的消息队列，是固定大小的消息队列（可以同时存放的最大消息数量，是在创建时就指定的），并且其中每个消息的大小也是固定的，如下图所示：
![](./FreeRTOS-use-queue/queue_size.png)

<br/>


创建消息队列的API为：
```c
QueueHandle_t xQueueCreate( UBaseType_t uxQueueLength, UBaseType_t uxItemSize )
```
- uxQueueLength：消息队列大小
- uxItemSize：消息队列中每个消息的大小。

- 返回值：消息队列句柄，用来唯一识别创建的这个消息队列。如果堆内存资源不够，则返回NULL。

消息队列创建成功后，即可向该消息队列中发送消息，或从中提取消息。
发送消息的API为：
```c
BaseType_t xQueueSend( QueueHandle_t xQueue, 
                        const void * pvItemToQueue, 
                        TickType_t xTicksToWait );
```
- xQueue：上述的创建消息队列API 返回的，用来标识一个消息队列的句柄。
- pvItemToQueue：待发送的消息。
- xTicksToWait：如果消息队列当前已经满了，那么本次发送数据就会导致调用该api的任务进入阻塞态，以等待消息队列有空闲位置可以放下本次要发送的数据。等待的时间即为`xTicksToWait`。如果为0，则表示不等待，立刻返回。注意，该值需要使用宏`pdMS_TO_TICKS(ms)`将时间转换成内核可以识别的tick数。

- 返回值：发送成功则返回`pdPASS`，否则即队列满了，无法发送。


从消息队列中获取消息的API为：
```c
BaseType_t xQueueReceive( QueueHandle_t xQueue, 
                            void *pvBuffer, 
                            TickType_t xTicksToWait );
```
- xQueue：上述创建消息队列API 返回的，用来标识一个消息队列的句柄。
- pvBuffer：接收消息使用的缓存。
- xTicksToWait：如果消息队列当前是空的，那么本次提取数据就会导致调用该api的任务进入阻塞态，以等待消息队列有消息可以提取。等待的时间即为`xTicksToWait`。如果为0，则表示不等待，立刻返回。

- 返回值：提取消息成功则返回`pdPASS`，否则即队列为空，获取不到数据。


<br/>

示例代码：我们创建两个任务`sender1`和`sender2`分别向同一个消息队列中发送消息。创建一个`receiver`任务，消费消息队列中的消息。
```c

struct MyData {
    int cmd_id;
    int other;
};

QueueHandle_t my_queue;

int main(void) {

    bsp_init();

    // 创建消息队列
    my_queue = xQueueCreate(5, sizeof(struct MyData));
    if(NULL != my_queue) {
        
        // 创建发送/接收任务，并将创建的消息队列作为参数传递给任务
        // 任务内部，使用创建的消息队列
        if (pdPASS == xTaskCreate(sender1_task, "sender1_task", 100, my_queue, 1, NULL)
         && pdPASS == xTaskCreate(sender2_task, "sender2_task", 100, my_queue, 1, NULL)
         && pdPASS == xTaskCreate(receiver_task, "receiver_task", 100, my_queue, 1, NULL)) {
            
            SEGGER_RTT_printf(0, "start FreeRTOS\n");
            vTaskStartScheduler();
        } 

    }

    // 正常启动后不会运行到这里
    SEGGER_RTT_printf(0, "something wrong\n");

    for( ;; );
    return 0;    
}
```

发送任务中：通过参数获得消息队列
```c
void sender1_task( void *pvParameters ) {
    QueueHandle_t my_queue = (QueueHandle_t)pvParameters;
    
    struct MyData data;
    data.cmd_id = 0;
    data.other = 0;
    for(;;) {
        // 发送消息
        xQueueSend(my_queue, &data, 0);
        vTaskDelay(pdMS_TO_TICKS(1000));
    }
}

void sender2_task( void *pvParameters ) {
    QueueHandle_t my_queue = (QueueHandle_t)pvParameters;
    
    struct MyData data;
    data.cmd_id = 1;
    data.other = 1;
    for(;;) {
        // 发送消息
        xQueueSend(my_queue, &data, 0);
        vTaskDelay(pdMS_TO_TICKS(1000));
    }
}

```

数据接收任务：通过参数获得消息队列
```c
void receiver_task( void *pvParameters ) {
    QueueHandle_t my_queue = (QueueHandle_t)pvParameters;
   
    struct MyData data;
    for(;;) {
        if (pdPASS == xQueueReceive(my_queue, &data, pdMS_TO_TICKS(1500))) {
            SEGGER_RTT_printf(0, "receive cmd:%d\n", data.cmd_id);
        }

    }
}
```
程序运行结果如下所示：
```
start FreeRTOS
receive cmd:0
receive cmd:1
receive cmd:0
receive cmd:1
receive cmd:0
receive cmd:1
...

```

通过上面的例子，我们可以发现几个FreeRTOS消息队列特性：
- 每次发送的消息是固定大小的，即在创建消息队列时指定的uxItemSize。但是现实开发中，有些场景需要发送的数据是变长的。针对这种问题可以通过传递数据指针来解决，例如将消息队列中存放的消息定义成如下格式：
  ```c
    struct MyData {
        int data_len;
        void *pdata;
    };
  ```
  消息接收方提取消息后，通过`pdata`获得实际数据存储位置，通过`data_len`获取实际数据长度。
<br/>
- FreeRTOS提供的消息队列，是多任务（多线程）安全的。示例代码中，存在多个发送/接收任务对同一个消息队列进行操作，但我们并未对该消息队列（在这里是属于共享资源）的访问进行临界区保护。因为FreeRTOS的消息队列模块内部实现中，已经做了临界区保护。


实际开发中，另一个常见的需求可能是，我们在收到某个中断事件后，需要向消息队列中发送一个消息。中断环境下，不能使用上文的发送API（上文的发送api可能会导致调用者阻塞，中断处理函数中是不允许阻塞的），需要使用FreeRTOS提供的带FromISR 后缀的发送API：
```c
BaseType_t xQueueSendFromISR( QueueHandle_t xQueue, 
                                const void *pvItemToQueue, 
                                BaseType_t *pxHigherPriorityTaskWoken );
```
- xQueue：标识一个消息队列的句柄。
- pvItemToQueue：待发送的消息。
- pxHigherPriorityTaskWoken：当我们在中断处理程序中向消息队列发送消息一个消息后，如果存在另一个任务刚好在等待消息队列有数据，那么该任务就会恢复就绪。如果该任务的优先级比当前任务（被中断服务程序打断的任务）更高，pxHigherPriorityTaskWoken就会被设置为True。表示有更高优先级的任务就绪了。

例如，我们在前面的例子上，再一个按键中断处理函数中向消息队列发送消息的示例：
```c

void GPIOTE_IRQHandler(void){

    BaseType_t higher_task_woken = pdFALSE;
    if ( NRF_GPIOTE->EVENTS_PORT == 1 ){

        //中断处理函数中要清除event,不然会导致一直产生中断
        NRF_GPIOTE->EVENTS_PORT = 0;      

        // 判断按键，发送消息
        if(IS_BUTTON_PRESSED(BUTTON_1)) {
            struct MyData data;
            data.cmd_id = 2;
            data.other = 2;
            xQueueSendFromISR(my_queue, &data, &higher_task_woken);
        }
    }
    // 如果higher_task_woken=True，表示有更高优先级任务就绪了
    // 我们使用的是抢占式调度，只要有更高优先级的任务就绪，应该让其立刻运行。
    // 下面的代码就是，判断有更高优先级就绪时，就会设置任务切换中断。那么当前中断函数退出后，就会立刻触发任务切换，让最高优先级的就绪任务运行。
    portYIELD_FROM_ISR(higher_task_woken);
}
```
程序运行结果如下所示，每次按键按下，都能收到 `cmd_id = 2` 的消息：
```
receive cmd:0
receive cmd:1
receive cmd:2
receive cmd:0
receive cmd:1
receive cmd:2
receive cmd:0
receive cmd:1
receive cmd:2

```

<br/>
ps：需要注意文章代码中的日志输出函数，产品代码中如果需要使用的话，需要考虑线程安全性（多任务安全性），因为中断/任务切换可能发生在另一个任务正在输出日志但还未输出完的时候，这就可能造成日志错乱

<br/>
<br/>
FreeRTOS交流QQ群-663806972
<br/>
