---
title: GNU linker ld —— Simple Linker Script Example
date: 2024/6/22
categories: 
- GUN-linker
---

<center>
GNU linker ld (GNU Binutils) version 2.42 —— Linker Scripts：Simple Linker Script Example
</center>

<!--more-->

***

**个人笔记，仅供参考！！**

最简单的链接脚本只有一个命令：`SECTIONS`，用来描述输出文件的内存布局。

这里提供一个简单用法示例：假设程序（输入文件）中只包含代码、已初始化数据和未初始化数据，并分别位于`.text`、`.data`和`.bss`段中，期望将代码在`0x10000`开始的区域，数据在`0x8000000`开始的区域。以下是一个能够实现此目的的链接脚本：
```
SECTIONS
{
  . = 0x10000;
  .text : { *(.text) }
  . = 0x8000000;
  .data : { *(.data) }
  .bss : { *(.bss) }
}
```

这个示例中，关键字`SECTIONS`后跟一系列符号赋值和输出段描述，这些描述被包含在花括号中。

`SECTIONS`命令中的第一行设置了特殊符号`.`的值，即位置计数器。如果没有以其他方式指定输出段的地址，则该输出段地址将从位置计数器的当前值设置开始。然后，位置计数器会增加该输出段的大小。
在`SECTIONS`命令开始时，位置计数器的值为 0 。

第二行定义了一个输出段`.text`，冒号是必需的语法，在输出段名称后的花括号内，列出了应该放置到此输出段中的输入段的名称。`*`是一个通配符，匹配任何文件名。表达式`(.text)`表示输入文件中`.text`输入段。

由于在定义输出段`.text`时位置计数器是`0x10000`，链接器将设置输出文件中`.text`段的地址为‘0x10000’。

剩余行定义了输出文件中的`.data`和`.bss`段。链接器脚本指定`.data`输出段放置在地址`0x8000000`。在`.data`输出段后，位置计数器的值将是`0x8000000`加上`.data`输出段的大小。因此，`.bss`输出段紧接在内存中的`.data`输出段之后。

链接器将确保每个输出段都具有所需的对齐。此外，还可以手动通过增加位置计数器或特殊命令（如ALIGN）来实现特殊的对齐要求。


#### 参考连接：
【1】[https://sourceware.org/binutils/docs/ld/Simple-Example.html](https://sourceware.org/binutils/docs/ld/Simple-Example.html)