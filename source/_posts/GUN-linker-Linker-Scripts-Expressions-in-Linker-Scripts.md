---
title: GNU linker ld —— Expressions in Linker Scripts
date: 2024/7/17
categories: 
- GUN-linker
---

<center>
GNU linker ld (GNU Binutils) version 2.42 —— Linker Scripts：Expressions in Linker Scripts
</center>

<!--more-->

***

**个人笔记，仅供参考！！**
链接器脚本语言中表达式的语法与C表达式的语法相同，只是在某些地方需要使用空白来解决语法歧义。所有表达式都计算为整数。所有表达式都以相同的大小计算，如果主机和目标都是32位，则为32位，否则为64位。

可以在表达式中使用和设置符号值。

链接器定义了几个在表达式中使用的特殊用途的内置函数:

##### 1 Constants

所有常数都是整数。
与C语言一样，链接器认为以' 0 '开头的整数为八进制，以' 0x '或' 0x '开头的整数为十六进制。或者，链接器接受十六进制的' h '或' h '后缀，八进制的' o '或' o '后缀，二进制的' b '或' b '后缀，十进制的' d '或' d '后缀。任何没有前缀或后缀的整数值都被认为是十进制。

此外，可以使用后缀 K 和 M 分别将常数缩放为1024或1024*1024。例如，以下都是指同一数量:
```
 
_fourk_1 = 4K;
_fourk_2 = 4096;
_fourk_3 = 0x1000;
_fourk_4 = 10000o;
```
注意- K和M后缀不能与上面提到的基本后缀一起使用。

##### 2 Symbolic Constants

可以通过使用 CONSTANT(name) 来引用特定于目标的常量，其中name是以下之一:
- MAXPAGESIZE ：目标的最大页面大小。
- COMMONPAGESIZE ：目标的默认页面大小。

使用示例:
```
.text ALIGN (CONSTANT (MAXPAGESIZE)): {*(.text)}
```
将创建一个与目标支持的最大页面边界对齐的输出段 .text。

##### 3 Symbol Names
除非加引号，否则符号名称以字母、下划线或句点开头，可以包括字母、数字、下划线、句点和连字符。未加引号的符号名不能与任何关键字冲突。可以指定包含特殊字符或与关键字名称相同的符号，方法是将符号名称用双引号括起来，例如：
```
 
"SECTION" = 9;
"with a space" = "also with a space" + 10;
```
由于符号可以包含许多非字母字符，因此用空格分隔符号是最安全的。例如，`A-B`是一个符号，而`A - B`是一个涉及减法的表达式。

##### 4 Orphan Sections
“孤儿段” 是指在输入文件中存在的段，但在链接脚本中没有明确放置到输出文件中。链接器仍然会通过查找或创建一个合适的输出段来放置孤立的输入段，从而将这些段复制到输出文件中。

如果孤立输入段的名称恰好与现有输出段的名称匹配，那么孤立输入段将放置在该输出段的末尾。

如果没有与之匹配的输出段，则会创建新的输出段。每个新的输出段将与其中的孤儿段具有相同的名称。如果有多个具有相同名称的孤儿段，它们将合并为一个新的输出段。

如果创建新的输出段来容纳孤儿段，那么链接器必须决定在现有输出段中如何放置这些新的输出段。在大多数现代目标上，链接器会尝试将孤儿段放置在具有相同属性的段之后。如果找不到具有匹配属性的段，或者不支持此功能，那么孤儿段将放置在文件的末尾。

命令行选项“–orphan-handling”和“–unique”可用于控制将孤儿段放置在哪些输出段中

##### 5 The Location Counter
**dot（“.”）变量**：
- “.”是一个特殊的链接器变量，它始终包含当前的输出位置计数器（output location counter）。
- 在链接脚本中，我们可以使用“.”来引用输出节（output section）中的位置。
- 但是，“.”只能出现在SECTIONS命令内的表达式中。

**给“.”赋值：**
- 将一个值赋给“.”会导致位置计数器移动。这可以用来在输出节中创建空隙。
- 位置计数器不能在输出部分内向后移动，也不能在输出部分外向后移动，如果这样做会创建具有重叠LMAs（load memory addresses）。

示例：在下面的示例中，定义了一个名为“output”的输出段，其中包含了三个文件的“.text”节。
其中，使用了“. = . + 1000”；和“. += 1000;” 创建1000字节的空隙。
“= 0x12345678”指定了在这些空隙中写入的数据（见“Output Section Fill”）。
```
SECTIONS
{
  output :
  {
    file1(.text)
    . = . + 1000;
    file2(.text)
    . += 1000;
    file3(.text)
  } = 0x12345678;
}
```

**关于“.”的实际含义：**
- 实际上，“.”表示相对于当前包含对象的起始字节偏移量。
- 通常情况下，这个包含对象是SECTIONS语句，其起始地址为0，因此“.”可以用作绝对地址。
- 但是，**如果在段描述内部使用“.”，它将表示相对于该节的起始字节偏移量，而不是绝对地址**。

示例：
```
SECTIONS
{
    . = 0x100;
    .text: {
        *(.text)
        . = 0x200;
    }
    . = 0x500;
    .data: {
        *(.data)
        . += 0x600;
    }
}
```
上述示例中，".text" 输出段将被分配一个起始地址0x100和正好0x200字节的大小，即使.text输入段中没有足够的数据填充此区域。(如果有太多的数据，将产生一个错误，因为这将是一个将“.”向后移动的尝试)。输出段".data"将从0x500开始，并且在输入段'.data'填充结束后，还具有额外的0x600字节的空间。数据“输入”部分和“结束”之前。数据输出部分本身。


如果链接器需要放置孤儿段，**将符号设置为输出段语句之外的位置计数器的值可能会导致意外值**。例如，给定以下内容:
```
SECTIONS
{
    start_of_text = . ;
    .text: { *(.text) }
    end_of_text = . ;

    start_of_data = . ;
    .data: { *(.data) }
    end_of_data = . ;
}
```
如果现在输入中包含一些上述脚本中没有提到的输入段，例如“.rodata”，由于链接器不会将上述符号名（'start_of_data'这些）与段关联起来。相反，**链接器假设所有赋值或其它语句都属于前一个输出部分，除了赋值给“.”的特殊情况**。也就是说，链接器会像这样放置“.rodata”部分
```
SECTIONS
{
    start_of_text = . ;
    .text: { *(.text) }
    end_of_text = . ;

    start_of_data = . ;
    .rodata: { *(.rodata) }
    .data: { *(.data) }
    end_of_data = . ;
}
```
这大概率不是脚本作者对“start_of_data”值的意图。影响孤立段位置的一种方法是将位置计数器赋值给自身，**因为链接器假定赋值给"."正在设置以下输出部分的起始地址，因此应该与该部分分组**。所以可以这样写:

```
SECTIONS
{
    start_of_text = . ;
    .text: { *(.text) }
    end_of_text = . ;

    . = . ;
    start_of_data = . ;
    .data: { *(.data) }
    end_of_data = . ;
}
```

这样，孤儿段“.rodata”将放在“end_of_text”和“start_of_data”之间。

##### 6 Operators

链接器能识别标准的 C 算术运算符集
```
precedence      associativity   Operators                           Notes
(highest)
1               left            !  -  ~                                (1)
2               left            *  /  %
3               left            +  -
4               left            >>  <<
5               left            >  <  <=  >=
6               left            ==  !=
7               left            &
8               left            ^
9               left            |
10              left            &&
11              left            ||
12              right           ? :
13              right           +=  -=  *=  /=  <<=  >>=  &=  |= ^=    (2)
```
备注：这里(1)为前缀操作符 (2)参见[符号赋值](https://fengxun2017.github.io/2024/07/01/GUN-linker-Linker-Scripts-Assigning-Values-to-Symbols/)


##### 7 Evaluation

链接器对表达式进行“惰性计算”（lazy evaluation）：只有在绝对必要时才计算表达式的值。

链接器需要一些信息，例如第一个段的起始地址的值以及内存区域的起始地址和长度，以便进行链接。这些值会在链接器读取链接脚本时尽早计算出来。

然而，其它值（例如符号值）直到存储分配之后才会知道或需要。这些符号会在其它信息（例如输出段的大小）可用于符号赋值表达式时再求值。

段的大小只有在分配之后才能知道，因此依赖于这些大小的赋值直到分配之后才执行。

某些表达式，例如依赖于位置计数器“.”的表达式，在段分配期间进行求值。

如果需要表达式的结果，但值不可用，则会出现错误。例如，以下脚本：
```
SECTIONS
{
  .text 9+this_isnt_constant :
    { *(.text) }
}
```
将导致错误消息“初始地址的表达式不是常量”。

##### 8 The Section of an Expression
见[The Section of an Expression](https://fengxun2017.github.io/2024/06/30/GUN-linker-Linker-Scripts-The-Section-of-an-Expression/)

##### 9 Builtin Functions
***ABSOLUTE(exp)***:
返回表达式exp的绝对值。主要用于在段定义中为符号赋绝对值，其中符号值通常是相对于段的。

***ADDR(section)***:
返回指定section的地址(VMA)。脚本中必须事先定义了该section的位置。例如：
```
SECTIONS { …
  .output1 :
    {
    start_of_output_1 = ABSOLUTE(.);
    …
    }
  .output :
    {
    symbol_1 = ADDR(.output1);
    symbol_2 = start_of_output_1;
    }
… }
```
在上面的例子中，start_of_output_1, symbol_1和symbol_2被赋相等的值，除了symbol_1是相对于输出段`.output1`，而其它两个是绝对的:


***ALIGN(align)***：
***ALIGN(exp,align)***：
将当前的位置计数器（location counter）或任意表达式对齐到下一个指定的边界。这个命令有两种形式：
- 单操作数形式：ALIGN(align)。这种形式不会改变位置计数器的值，只是在其上执行算术运算。
- 双操作数形式：ALIGN(exp,align)。这种形式允许将任意表达式向上对齐（ALIGN(align) 等效于 ALIGN(ABSOLUTE(.), align)）。

以下是一个示例，展示如何将输出段 `.data` 对齐到前面的段之后的下一个 0x2000 字节边界，并在该段内设置一个变量，使其对齐到输入段之后的下一个 0x8000 边界：
```
SECTIONS {
  ...
  .data ALIGN(0x2000): {
    *(.data)
    variable = ALIGN(0x8000);
  }
  ...
}
```


***ALIGNOF(section)***：
如果指定的section已被分配，则返回该section的对齐方式(以字节为单位)，如果该section尚未被分配，则返回零。如果链接器脚本中不存在该section，则链接器将报告错误。

```
SECTIONS{ …
  .output {
    LONG (ALIGNOF (.output))
    …
    }
… }
```

***BLOCK(exp)***：
与 ALIGN 作用相同，以便与旧链接器脚本兼容。一般用于设置输出段的地址。

***DATA_SEGMENT_ALIGN(maxpagesize, commonpagesize)***：
这个函数的目的是将数据段（这个表达式的结果和**DATA_SEGMENT_END**之间的区域）对齐到指定的页面大小（page size）。具体来说，它会根据情况选择以下两种形式之一计算对齐后的地址：
- 如果使用第一种形式，那么对齐后的地址为：(ALIGN(maxpagesize)+(.&(maxpagesize−1)))
- 如果使用第二种形式，那么对齐后的地址为：(ALIGN(maxpagesize)+((.+commonpagesize−1)&(maxpagesize−commonpagesize)))

具体选择哪种形式取决于是否使用第二种形式可以减少数据段所需的 commonpagesize 大小的页面数。如果使用第二种形式，这意味着在磁盘文件上可能会浪费 commonpagesize 字节，但在运行时内存中将节省 commonpagesize 字节。

需要注意的是，**这个表达式只能直接在 SECTIONS 命令中使用，而不能在任何输出段描述中使用，且在链接脚本中只能使用一次**。此外，commonpagesize 应该小于等于 maxpagesize，并且应该是对象希望在最大页面大小为 maxpagesize 的系统上进行优化的系统页面大小。注意，如果系统页面大小大于 commonpagesize，则 `-z relro`保护将不会生效。

***DATA_SEGMENT_END(exp)***:
这为DATA_SEGMENT_ALIGN定义的数据段的结束。例如：
`. = DATA_SEGMENT_END(.);`


***DATA_SEGMENT_RELRO_END(offset, exp)***:
当使用 -z relro 选项时，这个函数用于定义 PT_GNU_RELRO 段的结束位置。如果没有使用 -z relro 选项，DATA_SEGMENT_RELRO_END 不会产生任何效果。否则，它会对 DATA_SEGMENT_ALIGN 进行填充，以使 exp + offset 对齐到 DATA_SEGMENT_ALIGN 中给定的 commonpagesize 参数。如果在链接脚本中存在该函数，它必须放置在 DATA_SEGMENT_ALIGN 和 DATA_SEGMENT_END 之间。该函数的计算结果是第二个参数加上由于段对齐而在 PT_GNU_RELRO 段末尾所需的填充。

***DEFINED(symbol)***:
该函数的功能是判断一个符号是否在链接器的全局符号表中，并且在使用 DEFINED 语句之前是否已经定义。
如果 symbol 存在于全局符号表并且在 DEFINED 语句之前已经定义，那么函数返回 1。否则，返回 0。
可以使用这个函数为符号提供默认值。例如，以下链接脚本片段展示了如何将全局符号 begin 设置为 .text 段中的第一个位置。但是，如果名为 begin 的符号已经存在，则保留其值：
```
SECTIONS { …
  .text : {
    begin = DEFINED(begin) ? begin : . ;
    …
  }
  …
}
```

***LENGTH(memory)：***
这个函数返回名为 memory 的内存区域的长度。

****LOADADDR(section)：***
返回指定段（section）的绝对加载地址（Load Memory Address，LMA）。

***LOG2CEIL(exp)：***
返回以2为底的 exp 的对数，向上取整。LOG2CEIL(0)返回 0

***MAX(exp1, exp2)：***
返回 exp1 和 exp2 中的较大值。

***MIN(exp1, exp2)：***
返回 exp1 和 exp2 中的较小值。

***NEXT(exp)：***
返回下一个未分配的地址，该地址是 exp 的倍数。这个函数与 ALIGN(exp) 密切相关；除非使用 MEMORY 命令为输出文件定义不连续的内存，否则这两个函数是等效的。

***ORIGIN(memory)：***
返回名为 memory 的内存区域的起始地址。

***SEGMENT_START(segment, default)：***
返回指定段（segment）的基地址。如果已经使用命令行的 -T 选项为该段设置了显式值，那么将返回该值；否则将返回 default。目前，-T 命令行选项只能用于设置“text”、“data”和“bss”段的基地址，但 SEGMENT_START中可以使用任何段名。


***SIZEOF(section)：***
返回指定段（section）的字节大小，如果该段已分配，则返回其大小；如果该段未分配，则返回0。如果链接脚本中不存在该段，链接器将报告错误。如果 section 是 NEXT_SECTION，则 SIZEOF 将返回链接脚本中指定的下一个已分配段的对齐值，如果没有这样的段，则返回0。在以下示例中，symbol_1 和 symbol_2 被赋予相同的值：
```
SECTIONS {
  ...
  .output {
    .start = . ;
    ...
    .end = . ;
  }
  symbol_1 = .end - .start ;
  symbol_2 = SIZEOF(.output);
  ...
}
```

***SIZEOF_HEADERS：***
返回输出文件头部的字节大小。这是出现在输出文件开头的信息。如果需要，可以使用此数字来设置第一个段的起始地址，以便进行分页。
注意：当生成 ELF 输出文件时，如果链接脚本使用了 SIZEOF_HEADERS 内置函数，链接器必须在确定所有段的地址和大小之前计算程序头的数量。如果链接器后来发现需要更多的程序头，它将报告错误“没有足够的空间用于程序头”。为避免此错误，应该必须避免使用 SIZEOF_HEADERS 函数，或者重新设计链接脚本以避免强制链接器使用额外的程序头，或者使用 PHDRS 命令自己定义程序头。


#### 参考连接：
【1】[https://sourceware.org/binutils/docs/ld/Expressions.html](https://sourceware.org/binutils/docs/ld/Expressions.html)