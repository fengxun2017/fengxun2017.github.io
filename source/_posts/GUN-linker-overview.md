---
title: GNU linker ld —— Overview
date: 2024/4/1
categories: 
- GUN-linker
---

<center>
GNU linker ld (GNU Binutils) version 2.42 —— Overview
</center>

<!--more-->

***

#### Overview
ld 工具将对象和归档文件进行组合，重新定位它们的数据并绑定符号引用。通常编译程序的最后一步是运行 ld ，对所有对象进行链接。

ld 接受链接器命令语言文件，以提供对链接过程明确和完全地控制。

此版本 ld 使用通用的 [BFD](https://sourceware.org/binutils/docs/ld/BFD.html) 库，对对象文件进行操作。这允许ld读取、组合和写入许多不同格式的目标文件。不同的格式可以链接在一起以产生任何可用类型的目标文件。

除了灵活性之外，GUN链接器相比其他链接器能提供更有用的诊断信息。许多链接器在遇到错误时立即停止执行，而 ld 总是尽可能地继续运行，这可以帮忙我们识别其它错误（或者，在某些情况下，尽管有错误，也可以获得输出文件）。

#### 参考连接：
【1】[https://sourceware.org/binutils/docs/ld/Overview.html](https://sourceware.org/binutils/docs/ld/Overview.html)