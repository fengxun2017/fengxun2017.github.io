---
title: FreeRTOS-stream buffer
date: 2023/2/22
categories: 
- FreeRTOS
---

<center>
stream buffer 是一种轻量级的数据流传递方式，通过限制只有一个写，并且也只有一个读，使得内部实现逻辑更高效，并且由于只有一个读和一个写，内部实现中需要保护的临界区“很小”，这些都提高了stream buffer传递数据的效率。
</center>

<!--more-->

***

当任务间需要传递较多数据时，一般会选择使用消息队列作为消息传递的媒介。FreeRTOS中的消息队列，属于“重资源”，其内部控制复杂占用资源较多。
例如，FreeRTOS的消息队列是允许多个任务并发读/写，因此，存在写和写的并发访问数据冲突，写和读的并发冲突，以及读和读的并发冲突。所以，其内部的数据读取全部过程都需要放在“临界区”中，数据写入的全部过程也需要放在“临界区”中，从而避免并发的读/写冲突。
FreeRTOS的消息队列可以参考：[FreeRTOS-使用消息队列](https://fengxun2017.github.io/2022/12/12/FreeRTOS-use-queue/)，[FreeRTOS-消息队列内部细节](https://fengxun2017.github.io/2022/12/08/FreeRTOS-queue-internal-details/)


但实际应用中，大多数时候，我们的应用场景是一个任务/中断服务（writer）向另一个任务传递(reader)数据。
也就是对于一些数据的传递场景，只存在一个writer负责写入数据（产生数据），也只存在一个reader负责读数据（消费数据）。FreeRTOS针对这种应用场景，提供了`stream buffer`模块。

由于只能应用于只有一个写和读的场景。因此`stream buffer`内部实现中不需要考虑写和写的竞争冲突，也不需要考虑读和读的竞争冲突，只需要考虑写和读的竞争冲突即可。所以`stream buffer`逻辑相对`消息队列`来说更简洁高效，并且内部的“临界区”也更小。
例如，当我们使用`stream buffer`来发送/获取数据时，其内部只在判断可写入的空闲大小/可读取的数据量时才需要同时访问**读指针**和**写指针**，如下图所示： 
蓝色区域为有效数据。
**Head**：表示可写空间起始位置。
**Tail**：表示可读数据起始位置。
![](./FreeRTOS-stream-buffer/readable-writeable.png)

因此，我们的临界区只要保护判断可写/可读大小的代码段即可。因为：
后续的**写操作**只会访问**Head**指针，而由于只存在**一个写**任务，不存在对**Head**的竞争访问，所以不需要保护。
同样，后续的**读操作**只会访问**Tail**指针，而由于只存在**一个读**任务，不存在对**Tail**的竞争访问，所以不需要保护。
PS：这里只是举例，`stream buffer`的实际实现中，判断可写/可读大小的代码段同样不需要临界区保护，不过还是存在一个小的临界区用来保护另一段代码，具体可参考文章[FreeRTOS-stream buffer内部细节](https://fengxun2017.github.io/2023/03/04/FreeRTOS-stream-buffer-details/)。

综上，`stream buffer`是FreeRTOS针对 只存在一个写，并且也只存在一个读的场景，提出的一个高效数据流传递方案。这里所说的数据流，是指数据的发送单位是以字节为单位。
例如，在文章[FreeRTOS-使用消息队列](https://fengxun2017.github.io/2022/12/12/FreeRTOS-use-queue/)中，我们创建消息队列时，就需要指定消息的大小（n字节），后续的每次发送/接收消息，都是以该大小为单位的（每次发送/接收n字节）。
而`stream buffer`使用的数据流形式，每次发送/接收是以字节为单位的。每次可以发送任意字节的数据，也可以接收任意字节的数据。

使用`stream buffer`需要将源文件`stream_buffer.c`添加到工程中，并且需要在工程配置文件`FreeRTOSConfig.h`中添加：
```c
#define configUSE_TASK_NOTIFICATIONS    (1)
```
**注意**：`stream buffer`使用了[Task Notification](https://fengxun2017.github.io/2023/02/07/FreeRTOS-task-notification/)的`index=0`的那个数据，如果你的工程里使用了`task notification`，需要注意这个问题。

创建一个 stream buffer：
```c
StreamBufferHandle_t xStreamBufferCreate( size_t xBufferSizeBytes, size_t xTriggerLevelBytes );
```
- xBufferSizeBytes：这个`stream buffer`的大小，一共可以缓存多少字节的数据。
- xTriggerLevelBytes：当任务从空的`stream buffer`获取数据时，可以阻塞任务（当设置了超时等待）。之后当某个任务向`stream buffer`中发送了大于等于`xTriggerLevelBytes`大小的字节后，就会自动唤醒在该`stream buffer`上等待数据的任务。
**注意**：如果等待时间到达设置的超时时间了，任务恢复就绪并运行后，即使buffer中所存的数据还没到xTriggerLevelBytes，任务还是会读取当前已有的数据。
- 返回值：返回一个可用的`stream buffer`。

向目标`stream buffer`中发送数据：
```c
size_t xStreamBufferSend( StreamBufferHandle_t xStreamBuffer,
                          const void *pvTxData,
                          size_t xDataLengthBytes,
                          TickType_t xTicksToWait );
```
- xStreamBuffer：用来标识目标`stream buffer`
- pvTxData：需要发送的数据。
- xDataLengthBytes：发送数据的字节大小。
- xTicksToWait：如果当前`stream buffer`中的可写空间不够写入目标字节数据，则等待`xTicksToWait`设置的超时时间。**注意**：如果超时时间到达了，即使当前`stream buffer`的可写空间不够写入全部目标数据，该函数仍旧会根据当前可写空间写入尽可能多的数据。
- 返回值：最终实际写入`stream buffer`中的数据量。

PS：如果数据写入方是中断处理函数，则使用`xStreamBufferSendFromISR`。数据写入方只能有一个，是一个任务或者是一个中断处理函数。

从目标`stream buffer`中读取数据：
```c
size_t xStreamBufferReceive(StreamBufferHandle_t xStreamBuffer,
                            void *pvRxData,
                            size_t xBufferLengthBytes,
                            TickType_t xTicksToWait );
```
- xStreamBuffer：目标`stream buffer`。
- pvTxData：用来存储读取的数据。
- xDataLengthBytes：指明`pvTxData`所指数组的大小。每次读取，会从`stream buffer`中读取尽可能多的字节，但不会超过`xDataLengthBytes`（用来放所读数据的buffer就这么大）。
- xTicksToWait：如果当前`stream buffer`为空，则任务会等待`xTicksToWait`设置的超时时间。
  如果超时时间到达前，写任务向`stream buffer`中发送了大于等于`xTriggerLevelBytes`字节的数据，会唤醒读任务，（如果读任务可以立刻运行会）读任务会读取尽可能多的数据（不超过`xDataLengthBytes`）。
  如果`stream buffer`中的数据一直小于`xTriggerLevelBytes`，当超时时间到达后，该函数仍旧会从`stream buffer`中读取尽可能多的数据。
- 返回值：最终实际从`stream buffer`中读取的数据量。

PS：如果数据读取方是中断处理函数，则使用`xStreamBufferReceiveFromISR`。数据读取方只能有一个，是一个任务或者是一个中断处理函数。


一个简单的使用样例： task_a 向 `stream buffer`写入数据，task_b 从`stream buffer`中读取数据。

任务代码：
```c

void task_a( void *pvParameters ) {

    StreamBufferHandle_t sbuffer = (StreamBufferHandle_t)pvParameters;
    char data[9] = {1,2,3,4,5,6,7,8,9};
    int send_len = 0;
    for(;;) {

        // 每次最多发送 9 字节。
        send_len = xStreamBufferSend(sbuffer, data, 9, pdMS_TO_TICKS(1000));
        SEGGER_RTT_printf(0, "sent %d bytes!\n", send_len);

        vTaskDelay(pdMS_TO_TICKS(1000));
    }
}


void task_b( void *pvParameters ) {
    
    StreamBufferHandle_t sbuffer = (StreamBufferHandle_t)pvParameters;
    char data[5];
    int recv_len = 0;
    
    for(;;) {
        
        //data 大小为5，所以每次最多读取 5 字节
        recv_len = xStreamBufferReceive(sbuffer, data, sizeof(data), pdMS_TO_TICKS(1000));
        SEGGER_RTT_printf(0, "%d bytes received\n", recv_len);
    }
}
```

main 函数代码：
```c
#include "FreeRTOS.h"
#include "task.h"
#include "stream_buffer.h"

int main(void) {

    // 创建一个stream buffer，最多可以放 20字节。
    // xTriggerLevelBytes=1，表示只要向stream buffer发送了大于等于1字节的数据，
    // 之前由于stream buffer为空，而处于阻塞态，等待数据的读数据任务，会被立刻唤醒，恢复就绪。
    StreamBufferHandle_t sbuffer= xStreamBufferCreate(20,1);

    if(NULL != sbuffer) {
        
        if (pdPASS == xTaskCreate(task_a, "task_a", 100, sbuffer, 1, NULL)
            && pdPASS == xTaskCreate(task_b, "task_b", 100, sbuffer, 1, NULL)){
            
            SEGGER_RTT_printf(0, "start FreeRTOS\n");
            vTaskStartScheduler();
        } 
    }

    // 正常启动后不会运行到这里
    SEGGER_RTT_printf(0, "insufficient resource\n");
    for( ;; );
    return 0;    
}
```

运行结果：
```
start FreeRTOS
sent 9 bytes!
5 bytes received        # 由于用来存所读数据的buffer只有5字节，因此每次最多读取5字节
4 bytes received
sent 9 bytes!
5 bytes received
4 bytes received
```

我们在 `task_b` 任务中加一个delay，让其消费变慢，使得写入方每次不能写完：
```c
void task_b( void *pvParameters ) {
    
    StreamBufferHandle_t sbuffer = (StreamBufferHandle_t)pvParameters;
    char data[5];
    int recv_len = 0;
    
    for(;;) {
        
        //data 大小为5，所以每次最多读取 5 字节
        recv_len = xStreamBufferReceive(sbuffer, data, sizeof(data), pdMS_TO_TICKS(1000));
        SEGGER_RTT_printf(0, "%d bytes received\n", recv_len);

        // 让数据消费变慢
        vTaskDelay(pdMS_TO_TICKS(3000));
    }
}
```

重新编译运行结果如下：
```

start FreeRTOS
sent 9 bytes!       # stream buffer 被写入了9 字节，还剩11 字节空闲空间
5 bytes received    # 被读取了5 字节，还剩16 字节空间
sent 9 bytes!       # 被写入了9 字节，还剩7 字节空闲空间
5 bytes received    # 被读取了5 字节，还剩12 字节空间
sent 7 bytes!       # 由于任务调度的原因，本次可写入空间的判断依据还停留在上一次判断，认为还剩7 字节空闲空间，因此只写了 7 字节。
                    # 而实际有12 字节，本次写入后，还剩5 字节空间。
sent 5 bytes!       # 只剩5字节空间，因此实际只写了5 字节，还剩0 字节空闲空间
5 bytes received    # 在这之后，由于消费慢，并且每次只取5 字节，
                    # 所以后续的每次发送，只能发送5字节
sent 5 bytes!
5 bytes received
sent 5 bytes!
sent 0 bytes!
5 bytes received
sent 5 bytes!
5 bytes received
sent 5 bytes!
sent 0 bytes!
```
<br/>
ps：需要注意文章代码中的日志输出函数，产品代码中如果需要使用的话，需要考虑线程安全性（多任务安全性），因为中断/任务切换可能发生在另一个任务正在输出日志但还未输出完的时候，这就可能造成日志错乱





