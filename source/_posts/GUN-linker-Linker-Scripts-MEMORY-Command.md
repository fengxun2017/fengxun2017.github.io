---
title: GNU linker ld —— MEMORY Command
date: 2024/7/7
categories: 
- GUN-linker
---

<center>
GNU linker ld (GNU Binutils) version 2.42 —— Linker Scripts：MEMORY Command
</center>

<!--more-->

***

**个人笔记，仅供参考！！**
链接器的默认配置允许分配所有可用内存。可以使用**MEMOR**Y命令来覆盖此设置。

MEMORY 命令描述了一个内存块的位置和大小。可以使用 MEMORY 命令来描述哪些内存区域可以被链接器使用，以及哪些内存区域必须避免使用。此外，还可以将输出段分配到特定的内存区域。链接器会根据这些内存区域设置段的地址，并在内存区域变得过满时发出警告。但是，链接器不会重新排列段以适应可用的内存区域。

每个链接脚本可以包含多个 MEMORY 命令，但所有定义的内存块都被视为在单个 MEMORY 命令中指定的。MEMORY 命令的语法如下：
```
MEMORY
{
  name [(attr)] : ORIGIN = origin, LENGTH = len
  ...
}
```

`name` 是在链接脚本中用于引用该区域的名称。该区域名称在链接脚本之外没有其他含义。区域名称存储在单独的命名空间中，不会与符号名称、文件名称或段名称冲突。每个内存区域在 MEMORY 命令中必须具有唯一的名称。但是，可以使用**REGION_ALIAS**命令为现有内存区域添加别名。

`attr` 是一个可选的属性列表，**用于指定是否将某个未在链接脚本中显式映射的输入段放入特定的内存区域**。如在**SECTIONS**命令中所描述，如果你没有为某个输入段指定输出段，链接器将创建一个与输入段同名的输出段。如果定义了区域属性，链接器将使用它们为它创建的输出段选择合适的区域。
`attr` 字符串只能由以下字符组成：
- R：只读段
- W：读/写段
- X：可执行段
- A：可分配段
- I：已初始化段
- L：与‘I’相同
- !：反转随后的任何属性的含义
- 
如果某个未映射的段与列出的属性中的任何一个匹配（除了‘!’），它将被放置在对应内存区域中。‘!’ 属性会反转随后的属性的匹配，因此，只有在该段不匹配随后列出的属性时，未映射的段才会放置在内存区域中。因此，属性字符串“RW!X”将匹配具有‘R’和‘W’属性的任何未映射段，但前提是该段不具有‘X’属性。

`origin` 是区域的起始地址的数值表达式。该表达式必须求值为常量，不能包含任何符号。关键字**ORIGIN**可以缩写为**org**或**o**（但不能缩写为ORG）。
`len` 是区域的大小（以字节为单位）的表达式。与起始地址表达式一样，该表达式必须仅为数字，并且必须求值为常量。关键字**LENGTH**可以缩写为**len**或**l**。

以下是一个示例，这个示例中指定了两个可用于分配的内存区域：一个从‘0’开始，大小为256k字节，另一个从‘0x40000000’开始，大小为4M字节。连接器将把未显式映射到内存区域且为只读或可执行的每个段放置到‘rom’内存区域中。连接器将把未显式映射到内存区域的其他段放置到‘ram’内存区域中。

MEMORY
{
  rom (rx)  : ORIGIN = 0, LENGTH = 256K
  ram (!rx) : ORIGIN = 0x40000000, LENGTH = 4M
}

一旦定义了一个内存区域，可以通过使用输出段属性`>region`来指示连接器将特定输出段放置到该内存区域中。
如果未为输出段指定地址，则连接器将将地址设置为内存区域内的下一个可用地址。如果指向内存区域的组合输出段对该区域来说太大，连接器将发出错误消息。

可以通过**ORIGIN(memory)**和**LENGTH(memory)**函数在表达式中访问内存的起始地址和长度：

#### 参考连接：
【1】[https://sourceware.org/binutils/docs/ld/MEMORY.html](https://sourceware.org/binutils/docs/ld/MEMORY.html)