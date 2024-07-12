---
title: GNU linker ld —— PHDRS Command
date: 2024/7/9
categories: 
- GUN-linker
---

<center>
GNU linker ld (GNU Binutils) version 2.42 —— Linker Scripts：PHDRS Command
</center>

<!--more-->

***

**个人笔记，仅供参考！！**

**PHDRS** 命令在链接脚本中的作用和目的是精确指定程序头（program headers）。

ELF（Executable and Linkable Format，可执行与可链接格式）的目标文件格式使用程序头（也称为段头）来描述如何将程序加载到内存中。可以使用 objdump 程序的 -p 选项来打印这些程序头。

程序头是一组数据结构，它们以二进制格式存储在可执行文件中。在 ELF 文件中，它们通常位于文件头之后，紧接着是各个段的数据。每个段对应于可执行文件中的一部分数据，例如代码、只读数据、只写数据等。而程序头告诉系统加载器如何将这些段加载到内存中。启动可执行程序时，系统加载器会读取程序头，以确定如何加载程序。
每个程序头具有以下字段：
- p_type：指示段的类型（例如加载段、动态段等）。
- p_offset：指示段在文件中的偏移量。
- p_vaddr：指示段在内存中的虚拟地址。
- p_filesz：指示段在文件中的大小。
- p_memsz：指示段在内存中的大小。
- p_flags：指示段的标志（例如可读、可写、可执行等）。


链接器默认会创建合理的程序头。但在某些情况下，可能需要更精确地指定程序头。可以使用 **PHDRS** 命令来实现这一目的。当链接器在链接脚本中看到 **PHDRS** 命令时，它将仅创建指定的程序头，而不会创建其他程序头。

链接器只在生成 ELF 输出文件时关注 PHDRS 命令。在其他情况下，链接器将简单地忽略 PHDRS。

 **PHDRS** 命令的语法如下。其中 PHDRS、FILEHDR、AT 和 FLAGS 是关键字。
```
PHDRS
{
  name type [ FILEHDR ] [ PHDRS ] [ AT ( address ) ]
        [ FLAGS ( flags ) ] ;
}
```

`name` 仅用于链接脚本中的 SECTIONS 命令的引用。它不会放入输出文件中。程序头名称存储在单独的命名空间中，不会与符号名称、文件名或节名称冲突。每个程序头必须具有唯一的名称。这些头按顺序处理，通常会映射到按升序加载地址排列的段。

`type` 可以是以下之一。数字表示关键字的值。
- PT_NULL（0）：表示未使用的程序头。
- PT_LOAD（1）：表示此程序头描述要从文件中加载的段。
- PT_DYNAMIC（2）：表示包含动态链接信息的段。
- PT_INTERP（3）：表示包含程序解释器名称的段。
- PT_NOTE（4）：表示包含注释信息的段。
- PT_SHLIB（5）：保留的程序头类型，由 ELF ABI 定义但未指定。
- PT_PHDR（6）：表示包含程序头的段。
- PT_TLS（7）：指示包含线程本地存储的段。
- expression ：给出程序头的数字类型的表达式。这可以用于上面没有定义的类型。

某些程序头类型描述了系统加载器将从文件中加载的内存段（segments）。在链接脚本中，通过将可分配的输出段（sections）放置在这些段（segments）中来指定这些段的内容。可以使用输出段属性“:phdr”将输出段放置在特定段（segment）中。

允许将某些输出段放置在多个段（segement）中。这仅意味着一个内存段(segment )包含另一个内存段(segment)。可以重复使用 “:phdr”，每个应包含该输出段的段（segment）使用一次。

如果使用 “:phdr” 将输出段放置在一个或多个段（segment）中，那么链接器将在相同的段（segment）中放置所有后续的可分配输出段，这是为了方便起见，因为通常会将一整组连续的输出段放置在单个段（segment）中。可以使用 “:NONE” 来覆盖默认段，并告诉链接器不要将输出段节放入任何段（segment）中。

可以在程序头类型之后使用 **FILEHDR** 和 **PHDRS** 关键字进一步描述段的内容。FILEHDR 关键字表示该段应包含 ELF 文件头。PHDRS 关键字表示该段应包含 ELF 程序头本身。如果应用于可加载段（type=PT_LOAD），则所有先前的可加载段必须具有这些关键字之一。

`AT ( address )`：可以通过使用 AT 表达式来指定内存中的特定地址来加载段（segment）。这与输出段属性的中的 AT 属性作用相同。程序头的 AT 命令会覆盖输出段属性。

`FLAGS ( flags )`：链接器通常会基于组成段(segment)的各个输出段来设置段标志。可以使用 FLAGS 关键字显式指定段标志。flags 的值必须是整数。它用于设置程序头中的 p_flags 字段。

一个使用 **PHDRS** 命令的示例。展示了在ELF系统上使用的典型程序头集合。
```
PHDRS
{
  headers PT_PHDR PHDRS ;
  interp PT_INTERP ;
  text PT_LOAD FILEHDR PHDRS ;
  data PT_LOAD ;
  dynamic PT_DYNAMIC ;
}
```

以下是链接脚本的示例。显示了在内存中使用的一组典型程序头。
```
SECTIONS
{
  . = SIZEOF_HEADERS;
  .interp : { *(.interp) } :text :interp
  .text : { *(.text) } :text
  .rodata : { *(.rodata) } /* 默认为 :text */
  …
  . = . + 0x1000; /* 移动到内存中的新页面 */
  .data : { *(.data) } :data
  .dynamic : { *(.dynamic) } :data :dynamic
  …
}
```



#### 参考连接：
【1】[https://sourceware.org/binutils/docs/ld/PHDRS.html](https://sourceware.org/binutils/docs/ld/PHDRS.html)