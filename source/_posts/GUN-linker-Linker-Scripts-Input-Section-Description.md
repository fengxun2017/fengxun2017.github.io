---
title: GNU linker ld —— Input Section Description
date: 2024/7/14
categories: 
- GUN-linker
---

<center>
GNU linker ld (GNU Binutils) version 2.42 —— Linker Scripts：Input Section Description
</center>

<!--more-->

***

**个人笔记，仅供参考！！**


最常见的输出段命令是输入段描述。

输入段描述是最基本的链接器脚本操作。通过使用输出段，可以告诉链接器如何在内存中布局程序，通过使用输入段描述，可以告诉链接器如何将输入文件映射到内存布局中。

##### Input Section Basics
链接脚本中的输入段描述由一个文件名（可选）和一个括号内的段名列表组成。文件名和段名可以是通配符模式，见后文输入段通配符模式。

最常见的输入段描述是在输出段中包含具有特定名称的所有输入段。例如，要包含所有输入的`.text`段，可以编写：
```
*(.text)
```

这里的“*”是一个通配符，匹配任何文件名。要从匹配文件名通配符的文件列表中排除一些文件，可以使用 **EXCLUDE_FILE** 来匹配除 EXCLUDE_FILE 列表中指定的文件之外的所有文件。例如：
```
EXCLUDE_FILE (*crtend.o *otherfile.o) *(.ctors)
```

这将包含除了 crtend.o 和 otherfile.o 以外的所有文件中的 .ctors 输入段。

EXCLUDE_FILE 命令也可以放在输入段列表内，例如：
```
*(EXCLUDE_FILE (*crtend.o *otherfile.o) .ctors)
```
这与前面的示例效果完全相同。为 EXCLUDE_FILE 支持两种语法对于包含多个输入段的列表很有用，例如：有两种方法可以包含多个节
```
*(.text .rdata)
*(.text) *(.rdata)
```
这两者之间的区别在于“.text”和“.rdata”输入段在输出段中出现的顺序。在第一个示例中，它们将混合在一起，以与在链接器输入中的顺序相同的顺序出现。在第二个示例中，所有“.text”输入段将首先出现，然后是所有“.rdata”输入段。

当在多个输入段中使用 EXCLUDE_FILE 时，如果排除在输入段列表内，则排除仅适用于紧随其后的节，例如：
```
*(EXCLUDE_FILE (*somefile.o) .text .rdata)
```
这会从除 somefile.o 以外的所有文件中包含所有“.text”输入段，而从所有文件中包含所有“.rdata”节，包括 somefile.o文件。要从 somefile.o 中排除“rdata”节，可以修改示例为：
```
*(EXCLUDE_FILE (*somefile.o) .text EXCLUDE_FILE (*somefile.o) .rdata)
```

或者，将 EXCLUDE_FILE 放在节列表之外，在输入文件选择之前，将导致排除适用于所有输入段。因此，前面的示例可以重写为：
```
EXCLUDE_FILE (*somefile.o) *(.text .rdata)
```

此外，输入按段描述支持指定文件名以包含来自特定文件的输入段。例如：
```
data.o(.data)
```
如果要根据输入段的标志来细化包含的输入段，可以使用 **INPUT_SECTION_FLAGS**。以下是使用 ELF 节的节头标志的简单示例：
```
SECTIONS {
  .text : { INPUT_SECTION_FLAGS (SHF_MERGE & SHF_STRINGS) *(.text) }
  .text2 :  { INPUT_SECTION_FLAGS (!SHF_WRITE) *(.text) }
}
```
在此示例中，输出节“.text”将包含任何匹配 *(.text) 且其输入段的标志 SHF_MERGE 和 SHF_STRINGS 已设置的输入段。输出节“.text2”将包含任何匹配 *(.text) 且其输入段标志 SHF_WRITE 未设置的输入段。

输入段描述还支持在归档文件中指定文件，方法是先写入与归档文件匹配的模式，然后写入与文件匹配的模式，冒号周围不要有空格。例如：
 
`archive:file`用来匹配归档中的文件。
`archive:`用来匹配整个存档。
`:file`匹配文件，但不匹配归档中的文件

`archive`和`file`中的一个或两个都可以包含shell通配符。在基于DOS的文件系统上，链接器将假定后面跟着冒号的单个字母是驱动器说明符，因此`c:myfile.o`仅表明一个文件，而不是`myfile.O`在一个名为`c `的归档中。`archive:file`文件也可以在 EXCLUDE_FILE 列表中使用，但可能不会出现在其他链接器脚本上下文中。例如，不能通过在INPUT命令中使用`archive:file`从归档中提取文件。

如果使用的文件名中没有输入段列表，那么输入文件中的所有输入段都将包含在输出段中。

当使用的文件名不是`archive:file`说明符并且不包含任何通配符时，链接器将首先查看是否还在链接器命令行或INPUT命令中指定了文件名。如果没有这样做，链接器将尝试将该文件作为输入文件打开，就像它出现在命令行上一样。注意，这与INPUT命令不同，因为链接器不会在存档搜索路径中搜索文件。

#####  Input Section Wildcard Patterns
在输入段描述中，文件名或输入段名都可以是通配符模式。

在许多示例中看到的“*”文件名是文件名的简单通配符模式。
通配符模式类似于Unix shell中使用的模式。
- `*` ：用来匹配任意数量的字符
- `?` ：匹配任意单个字符
- `[chars]` ： 匹配chars中的任意一个字符；可以使用“-”字符来指定字符范围，例如`[a-z]`匹配任何小写字母
- `\` ： 用于引用后面的字符，以避免其被解释为特殊字符

文件名通配符模式仅匹配在命令行上或在INPUT命令中明确指定的文件。链接器不会搜索目录中文件以匹配通配符。

如果文件名与多个通配符模式匹配，或者如果文件名明确出现并且也与通配符模式匹配，**链接器将在链接器脚本中使用第一个匹配**。例如，以下输入节描述序列可能存在错误，因为对data.o的匹配规则不会被使用：

.data : { *(.data) }
.data1 : { data.o(.data) }

通常，链接器将按照在链接期间看到的顺序放置由通配符匹配的文件和输入段。可以通过在通配符模式之前使用**SORT_BY_NAME**关键字（例如，SORT_BY_NAME(.text*)）来更改这一点。使用 SORT_BY_NAME 关键字时，链接器将按名称升序对文件或输入段进行排序，然后将它们放置在输出文件中。SORT是SORT_BY_NAME的别名。

**SORT_BY_ALIGNMENT** 类似于 SORT_BY_NAME。SORT_BY_ALIGNMENT 将输入段按对齐的降序排列，然后将其放置在输出文件中。将较大的对齐放在较小的对齐之前可以减少所需的填充空间。

**SORT_BY_INIT_PRIORITY** 也类似于SORT_BY_NAME。SORT_BY_INIT_PRIORITY 将输入段按照编码在段名中的 GCC init_priority 属性的升序数字顺序排列，然后将其放置在输出文件中。在.init_array.NNNNN和.fini_array.NNNNN中，NNNNN是init_priority。在.ctors.NNNNN和.dtors.NNNNN中，NNNNN是65535减去init_priority。



**REVERSE** 表示应该反转排序。如果仅在自身上使用，则 REVERSE 意味着 SORT_BY_NAME，否则它会反转封闭的 SORT.. 命令。
**注意：当前不支持对齐的反向排序。**

注意：排序命令仅接受单个通配符模式。因此，例如以下内容将无法工作：
```
*(REVERSE(.text* .init*))
```

要解决此问题，请单独列出模式，如下所示：
```
*(REVERSE(.text*))
*(REVERSE(.init*))
```

可以将EXCLUDE_FILE命令放在排序命令内部，但不能反过来。例如：
```
*(SORT_BY_NAME(EXCLUDE_FILE(foo) .text*))
```
将起作用，但：
```
*(EXCLUDE_FILE(foo) SORT_BY_NAME(.text*))
```
不起作用。

当链接器脚本中存在嵌套的输入段排序命令时，输入段排序命令最多可以有1级嵌套。例如：
- SORT_BY_NAME (SORT_BY_ALIGNMENT (wildcard section pattern))：首先按名称对输入段进行排序，然后按对齐方式对其进行排序（如果两个输入段具有相同的名称）。
- SORT_BY_ALIGNMENT (SORT_BY_NAME (wildcard section pattern))：首先按对齐方式对输入段进行排序，然后按名称对其进行排序（如果两个节具有相同的对齐方式）。
- SORT_BY_NAME (SORT_BY_NAME (wildcard section pattern))：与SORT_BY_NAME (wildcard section pattern)相同。
- SORT_BY_ALIGNMENT (SORT_BY_ALIGNMENT (wildcard section pattern))：与SORT_BY_ALIGNMENT (wildcard section pattern)相同。
- SORT_BY_NAME (REVERSE (wildcard section pattern))：按名称反向排序。
- REVERSE (SORT_BY_NAME (wildcard section pattern))：按名称反向排序。
- SORT_BY_INIT_PRIORITY (REVERSE (wildcard section pattern))：按初始化优先级反向排序。
- 其他嵌套的输入段排序命令无效。

当同时使用命令行段排序选项和链接器脚本段排序命令时，链接脚本中的段排序命令始终优先于命令行选项。如果链接器脚本中的段排序命令没有嵌套，命令行选项将使段排序命令被视为嵌套排序命令。
- SORT_BY_NAME (wildcard section pattern) 与 --sort-sections alignment 等效于 SORT_BY_NAME (SORT_BY_ALIGNMENT (wildcard section pattern))。
- SORT_BY_ALIGNMENT (wildcard section pattern) 与 --sort-section name 等效于 SORT_BY_ALIGNMENT (SORT_BY_NAME (wildcard section pattern))。

**如果链接器脚本中的段排序命令是嵌套的，则命令行选项将被忽略**。
SORT_NONE通过忽略命令行section排序选项来禁用section排序。

可以使用“-M”链接器选项生成映射文件，以便清楚地了解输入段是如何映射到输出段的。

以下示例展示了通配符模式如何用于分割文件。这个链接器脚本指示链接器将所有“.text”节放置在“.text”中，将所有“.bss”节放置在“.bss”中。链接器将以大写字母开头的所有文件中的“.data”节放置在“.DATA”中；对于其他文件，链接器将“.data”节放置在“.data”中。
```
SECTIONS {
  .text : { *(.text) }
  .DATA : { [A-Z]*(.data) }
  .data : { *(.data) }
  .bss : { *(.bss) }
}
```

##### Input Section for Common Symbols
链接脚本（Linker Scripts）中需要一种特殊的表示方式来处理 common symbols，因为在许多目标文件格式中，common symbols 并没有特定的输入段。链接器会将 common symbols 视为位于名为“COMMON”的输入段中。

可以像其它输入段一样使用带有“COMMON”段的文件名。这样，可以将来自特定输入文件的 common symbols 放在一个段中，而来自其他输入文件的 common symbols 放在另一个段中。

在大多数情况下，输入文件中的 common symbols 会被放置在输出文件的“.bss”段中。例如：
```
.bss { *(.bss) *(COMMON) }
```

某些目标文件格式中存在多种类型的 common symbols。例如，MIPS ELF 目标文件格式区分标准 common symbols 和小型 common symbols。在这种情况下，链接器会为其他类型的 common symbols 使用不同的特殊段名称。对于 MIPS ELF，链接器使用“COMMON”表示标准 common symbols，使用“.scommon”表示小型 common symbols。这使得可以将不同类型的 common symbols 映射到内存的不同位置。

在旧的链接脚本中，有时会看到废弃的标识方法“[COMMON]”，等效于“*(COMMON)”。

#####  Input Section and Garbage Collection
当使用链接时垃圾收集(`--gc-sections`)功能时，标记不应该被消除的段通常很有用。这是通过在输入段的通配符条目周围加上KEEP()来实现的，如`KEEP(*(.init))`或`KEEP(SORT_BY_NAME(*)(.ctors))`。

##### Input Section Example

```
 
SECTIONS {
  outputa 0x10000 :
    {
    all.o
    foo.o (.input1)
    }
  outputb :
    {
    foo.o (.input2)
    foo1.o (.input1)
    }
  outputc :
    {
    *(.input1)
    *(.input2)
    }
}
```
该示例是一个完整的链接脚本（linker script）。它告诉链接器从文件 all.o 中读取所有的节（sections），并将它们放置在名为 outputa 的输出段（output section）的开头，该段的起始地址是 0x10000。紧接着，来自文件 foo.o 的 .input1 节也放在同一个输出段中。而来自 foo.o 的 .input2 节则放在名为 outputb 的输出段中，随后是来自 foo1.o 的 .input1 节。剩余的所有 .input1 和 .input2 节都被写入名为 outputc 的输出段。

如果一个输出段的名称与输入段的名称相同，并且可以表示为 C 标识符（ C 语言中合法的标识符），那么链接器会自动创建两个符号：__start_SECNAME 和 __stop_SECNAME，其中 SECNAME 是段的名称。这些符号分别表示输出段的起始地址和结束地址。
注意：大多数段的名称无法表示为 C 标识符，因为它们包含了 `.` 字符

#### 参考连接：
【1】[https://sourceware.org/binutils/docs/ld/Input-Section-Common.html](https://sourceware.org/binutils/docs/ld/Input-Section-Common.html)