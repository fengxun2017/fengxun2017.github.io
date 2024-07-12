---
title: GNU linker ld —— SECTIONS Command
date: 2024/7/2
categories: 
- GUN-linker
---

<center>
GNU linker ld (GNU Binutils) version 2.42 —— Linker Scripts：SECTIONS Command
</center>

<!--more-->

***

**个人笔记，仅供参考！！**

链接脚本中的 **SECTIONS** 命令用于描述如何将输入段映射到输出段，并决定它们在内存中的位置。

SECTIONS 命令的格式如下：
``` 
 
SECTIONS
{
  sections-command
  sections-command
  …
}
```
每个 sections-command 可以是以下之一：

- ENTRY 命令（参见[Simple-Linker-Script-Commands/](https://fengxun2017.github.io/2024/06/28/GUN-linker-Linker-Scripts-Simple-Linker-Script-Commands/)）
- 符号赋值（Assigning Values to Symbols）（参见[Simple-Linker-Script-Commands/](https://fengxun2017.github.io/2024/06/28/GUN-linker-Linker-Scripts-Simple-Linker-Script-Commands/)）
- 输出段描述
- 覆盖描述(overlay description)

输出段描述和覆盖描述将在下面进行描述。

如果没有在链接器脚本中使用 **SECTIONS** 命令，链接器将按照输入文件中段出现的顺序，将每个输入段放入同名的输出段中。

#### 1 Output Section Description

链接脚本中输出段的完整描述如下：
```
 
section [address] [(type)] :
  [AT(lma)]
  [ALIGN(section_align) | ALIGN_WITH_INPUT]
  [SUBALIGN(subsection_align)]
  [constraint]
  {
    output-section-command
    output-section-command
    …
  } [>region] [AT>lma_region] [:phdr :phdr …] [=fillexp] [,]
```

大多数输出段不使用可选的段属性。在 **section** 周围的空格是必需的，以确保段名称不会引起歧义。冒号和花括号也是必需的。如果使用了 fillexp 并且下一个 sections-command 看起来是表达式的延续，那么末尾的逗号可能是必需的。换行和其他空白是可选的。

输出段描述中的 `output-section-command` 可以是以下之一：
- 符号赋值
- 输入段描述（见后文“Input Section Description”）
- 直接包含的数据值（见后文“Output Section Data”）
- 特殊的输出段关键字（“Output Section Keywords”）

##### 1.1 Output Section Name
`section`为输出段的名称，需要满足输出格式的要求。
输出节名`/DISCARD/`是特殊的，参考后文“Output Section Discarding”。

##### 1.2 Output Section Address
`address`是虚拟内存地址（VMA）的表达式，用于指定输出段的位置。这个地址是可选的，但如果提供了，输出地址将会被精确地设置为指定的值。
如果没有指定输出地址，链接器将根据以下启发式规则为该段选择一个地址。该地址将根据输出段的对齐要求进行调整。输出段的对齐要求是包含在输出段内的任何输入段的最严格对齐要求。输出段地址的启发式规则如下：
- 如果为该段设置了输出内存区域（输出段描述中的`[>region]`），则将其添加到该区域，并且其地址将是该区域中的下一个空闲地址。
- 如果使用 **MEMORY** 命令创建了一组内存区域，则选择与该段兼容的第一个区域来包含它。该段的输出地址将是该区域中的下一个空闲地址.
- 如果没有指定内存区域，或者没有与该段匹配的区域，则输出地址将基于当前位置计数器的值。

例如：`.text . : { *(.text) }` 和 `.text : { *(.text) }`
这两者之间有微妙的区别。第一个示例将 .text 输出段的地址设置为当前位置计数器的值。第二个示例将其设置为与 .text 输入段中最严格对齐要求对齐的当前位置计数器的值。

`address`可以是任意表达式（参考链接脚本中的表达式）。例如，如果想要将该段对齐到 0x10 字节边界，使得段地址的最低四位为零，可以这样做：
```
.text ALIGN(0x10) : { *(.text) }
```
这里的 **ALIGN** 返回当前位置计数器向上对齐到指定值的结果。

如果为某个段指定了地址，那么该段的地址将改变位置计数器的值，前提是该段非空（空段将被忽略）。

##### 1.3 Output Section Type
输出段可以使用`type`属性，来标记一些段特性，存在以下段类型定义：
- **NOLOAD**：该类型用于标记一个段为不可加载，当运行程序时，该段不会被加载到内存中。通常用于包含只读数据或其他不需要在运行时加载的数据的段。

- **READONLY**：该类型将段标记为只读。可以将一些只读数据，例如常量或配置信息，放在只读段中。

- **DSECT、COPY、INFO、OVERLAY**：这些类型是为了向后兼容而保留的，很少使用。它们都具有相同的效果：将段标记为“不可分配”，因此在运行程序时不会为该段分配内存。

- **TYPE = type**：可以在链接脚本中使用`TYPE = type`来显式设置输出段的类型，如果不显式设置类型，链接器将根据输入段的内容自动推断输出段的类型。在生成ELF输出文件时，`type`可以取值： 
  - SHT_PROGBITS：代码或只读数据段。
  - SHT_STRTAB：字符串表段。
  - SHT_NOTE：注释段。
  - SHT_NOBITS：不占用实际内存的段（例如BSS段）。
  - SHT_INIT_ARRAY：初始化函数数组段。
  - SHT_FINI_ARRAY：终止函数数组段。
  - SHT_PREINIT_ARRAY：预初始化函数数组段。

  需要注意的是，**只有当输出段的内容没有自己的隐式类型时，TYPE 才会生效**。
  `隐式类型`：输入文件中的每个段都有一个隐式类型。例如，代码段（.text）的隐式类型是 SHT_PROGBITS，而只包含零初始化数据的段（.bss）的隐式类型是 SHT_NOBITS。链接器通常会根据输入段的隐式类型来设置输出段的属性。
  `类型继承`：例如，`.foo . TYPE = SHT_PROGBITS { *(.bar) }` 将会将输出段`.foo`的类型设置为与输入段中的`.bar`段相同的类型，但不一定是 SHT_PROGBITS 类型（虽然使用了 TYPE 指令显式地设置了输出段的类型）。
  `覆盖隐式类型`：如果需要强制设置输出段的类型（即使输入段具有不同的隐式类型），则需要添加一个额外的无类型数据。例如：`.foo . TYPE = SHT_PROGBITS { BYTE(1); *(.bar) }`

- **READONLY ( TYPE = type )**：这种语法将 READONLY 类型与指定的类型结合起来。
  
综上，链接器通常根据映射到输出段的输入段来设置输出段的属性。可以通过使用段类型来覆盖这一点。例如，在下面的示例脚本中，“ROM”段位于内存位置“0”，在运行程序时不需要加载：
```

SECTIONS {
  ROM 0 (NOLOAD) : { … }
  …
}
```

##### 1.4 Output Section LMA

每个段都有一个虚拟地址（VMA）和一个加载地址（LMA）。虚拟地址由前面描述的输出段地址指定。加载地址由`AT`或`AT>`关键字指定。指定加载地址是可选的。

`AT`关键字接受一个表达式作为参数。这指定了段的确切加载地址。
`AT>`关键字接受一个区域的名称作为参数。请参阅 [MEMORY命令](https://fengxun2017.github.io/2024/07/07/GUN-linker-Linker-Scripts-MEMORY-Command/)。使用`AT>`，则段的加载地址设置为该区域中的下一个空闲地址，对齐到段的对齐要求。

**可分配的段**：是指在链接过程中需要在内存中分配空间的段。这些段包含程序的代码、数据或其他资源。
例如，`.text` 节通常包含程序的可执行代码，`.data` 节包含已初始化的全局变量，而 `.bss` 节用于存放未初始化的全局变量。
**不可分配的段**：是指在链接过程中不需要在内存中分配空间的段。这些段通常包含调试信息、只读数据或其他不需要在运行时加载的内容。
例如，`.rodata` 节通常包含只读数据，如字符串常量。

对于一个可分配的段，如果既没有指定`AT`也没有指定`AT>`，链接器将使用以下启发式方法来确定加载地址：
- 如果该段具有特定的VMA地址，则该地址也用作LMA地址。
- 如果该段不可分配，则其LMA设置为其VMA。
- 否则，如果可以找到与当前段兼容的内存区域，并且该区域包含至少一个段，则设置LMA，使当前段VMA与LMA之间的差等于该区域中最后一个段的VMA与LMA之间的差。
- 如果没有声明内存区域，则在上一步中使用“覆盖整个地址空间的默认区域”。
- 如果找不到合适的区域，或者没有先前的段，则LMA设置为等于VMA。


这个特性使构建ROM映像变得容易。例如，以下链接脚本创建了三个输出段：
一个名为“.text”，VMA为0x1000，LMA未指定，因此和VMA值相同。
一个名为“.mdata”，尽管其VMA为0x2000，但加载地址在“.text”段的末尾。
一个名为“.bss”的段，用于保存地址0x3000处的未初始化数据。符号_data的值定义为0x2000（这表明位置计数器保存的是VMA值，而不是LMA值）。
```
SECTIONS
{
  .text 0x1000 : { *(.text) _etext = . ; }
  .mdata 0x2000 :
    AT ( ADDR (.text) + SIZEOF (.text) )
    { _data = . ; *(.data); _edata = . ;  }
  .bss 0x3000 :
    { _bstart = . ;  *(.bss) *(COMMON) ; _bend = . ;}
}
```

使用此链接脚本生成的程序，其运行时初始化代码将包括类似以下的内容，以将初始化数据从ROM映像复制到其运行时地址。请注意，此代码利用了链接脚本定义的符号。

```
extern char _etext, _data, _edata, _bstart, _bend;
char *src = &_etext;
char *dst = &_data;

/* ROM中有数据位于文本末尾；将其复制。 */
while (dst < &_edata)
  *dst++ = *src++;

/* 清零bss。 */
for (dst = &_bstart; dst< &_bend; dst++)
  *dst = 0;
```

##### 1.5 Forced Output Alignment

**ALIGN** 属性用于设置输出节的对齐方式。具体来说，它会插入填充字节，直到当前位置对齐到指定的字节边界。
例如，`. = ALIGN(8)` 表示在当前位置插入填充字节，直到对齐到 8 字节的边界。

**ALIGN_WITH_INPUT** 属性可以用来确保 VMA（虚拟内存地址）和 LMA（加载内存地址）之间的差距在整个输出节中保持不变。这意味着，如果在某个输出节中使用了 ALIGN_WITH_INPUT，那么在该节中的每个输入段的 VMA 和 LMA 之间的差距将保持不变。

##### 1.6 Forced Input Alignment

在输出段描述中使用 **SUBALIGN** 属性时，它将在该输出段内强制指定输入段的对齐方式。例如：
```
.text : SUBALIGN(4)
{
    *(.text*)
} > flash
```
该例中，`.text` 段的对齐方式被设置为 4 字节。无论输入段的对齐方式如何，都会被 SUBALIGN 指定的值所覆盖。

##### 1.7 Output Section Constraint

**Constraint** 属性可以使用的关键字有`ONLY_IF_RO`和`ONLY_IF_RW`，分别用来指定，只有在输出段中的所有输入段都是只读的或都是读写的情况下，才创建该输出段。

##### 1.8 Output Section Region

通过使用 `>region`属性，可以将一个段（section）分配给 [MEMORY命令](https://fengxun2017.github.io/2024/07/07/GUN-linker-Linker-Scripts-MEMORY-Command/)定义的区域（region）。
示例：
```
MEMORY {
    rom : ORIGIN = 0x1000, LENGTH = 0x1000
}

SECTIONS {
    ROM : {
        *(.text)
    } >rom
}
```

MEMORY 部分定义了一个名为 `rom` 的内存区域，其起始地址（ORIGIN）为 0x1000，长度（LENGTH）为 0x1000。
SECTIONS 部分定义了一个名为 `ROM` 的输出段，它包含了所有`.text`段的内容。`>rom` 表示将 `ROM` 输出段分配到 `rom` 内存区域。


##### 1.9 Output Section Phdr
可以使用':phdr '将一个输出段分配给先前定义的程序段。参见[PHDRS命令](https://fengxun2017.github.io/2024/07/09/GUN-linker-Linker-Scripts-PHDRS-Command/)。如果一个section被分配给一个或多个段(segment)，那么所有后续分配的section也将被分配给这些段，除非它们显式地使用:phdr修饰符。可以使用:NONE来告诉链接器不将该节放在任何段中。

示例：
```
PHDRS { text PT_LOAD ; }
SECTIONS { .text : { *(.text) } :text }
```


##### 1.10 Output Section Fill
输出段属性`=fillexp`可以用来设置整个段的填充模式，`fillexp` 是一个表达式。输出段内的任何未指定的内存区域（例如，由于输入段所需的对齐而留下的间隙）将使用该值进行填充，并根据需要重复。如果填充表达式是一个简单的十六进制数，即以“0x”开头且没有尾随的“k”或“M”的十六进制数字字符串，那么可以使用任意长的十六进制数字序列来指定填充模式；前导零也成为模式的一部分。对于其他所有情况，包括额外的括号或一元加号，填充模式是表达式值的四个最低有效字节。如果值的大小小于四个字节，则会将其零扩展为四个字节。在所有情况下，数字都是大端序。
```
Fill Value     Fill Pattern
0x90           90 90 90 90
0x0090         00 90 00 90
144            00 00 00 90
```
还可以通过输出段命令中的 **FILL** 命令更改填充值（见后文）。

以下是一个简单的示例：
```
SECTIONS { .text : { *(.text) } =0x90909090 }
```
这将使用填充值 0x90909090 来填充 .text 节。



##### 1.11 Output Section Data
通过使用 **BYTE**、**SHORT**、**LONG**、**QUAD** 或 **SQUAD** 作为输出段命令，可以在输出段中包含确切字节的数据。每个关键字后跟一个括号中的表达式，提供要存储的值。

**BYTE**、**SHORT**、**LONG** 和 **QUAD** 命令分别存储 1、2、4 和 8个字节。存储字节后，位置计数器按存储的字节数递增。

例如，以下命令将存储字节1，后跟符号“addr”的四字节值：
```
 
BYTE(1)
LONG(addr)
```

当使用64位主机或目标时，**QUAD** 和 **SQUAD**是相同的;它们都存储8字节或64位的值。当主机和目标都是32位时，表达式计算为32位。在这种情况下，QUAD存储一个32位按 0 扩展到64位的值，SQUAD存储一个32位按符号扩展到64位的值。

通常情况下，输出文件的目标文件格式具有显式字节序，值会将按照该字节序存储。如果目标文件格式没有显式字节序，值将按照第一个输入目标文件的字节序存储。

除了存储数值，还可以使用 **ASCIZ** 关键字在输出段中包含以零结尾的字符串。该关键字后面跟着一个字符串，该字符串存储在当前位置计数器的值处，**末尾添加一个零字节**。如果字符串包含空格，则必须用双引号括起来。字符串可以包含 \n、\r、\t 和八进制数字，但不支持十六进制数字。

例如，下面这个16个字符的字符串将创建一个17字节的区域：
```
ASCIZ "This is 16 bytes"
```

**需要注意的是，这些命令仅在段描述内部有效，在段之间是无效的**。因此，下面的代码将导致链接器错误：
```
SECTIONS {
    .text : { *(.text) }
    LONG(1)
    .data : { *(.data) }
}
```
而以下代码将正常工作：
```
SECTIONS {
    .text : { *(.text) ; LONG(1) }
    .data : { *(.data) }
}
```


段内数据通常可能存在一些未指定的区域（例如，由于输入段所需的对齐而留下的间隙），可以使用 **FILL** 命令为当前段设置填充模式，该命令后面跟着一个括号中的表达式。在段内的其它未指定的内存区域将用表达式的值填充，必要时会重复。**FILL** 语句作用于它在**section**定义中出现的点之后的内存位置；通过包含多个**FILL**语句，可以在输出部分的不同部分使用不同的填充模式。

以下示例使用值“0x90”填充未指定的内存区域：
```
FILL(0x90909090)
```

**FILL** 命令类似于`[=fillexp]`输出段属性，但它仅影响在 **FILL** 命令之后的段部分，而不是整个段。如果两者都使用，**FILL** 命令将优先生效。

备注：填充模式是表达式值的四个最低有效字节。如果值的大小小于四个字节，则会将其零扩展为四个字节。实际填充时，如果待填充区域小于4字节，则填充值为对应的低字节区域。

通常情况下，如果填充模式表达式的值小于四字节，会按照 0 扩展到4字节，并用于填充。因此，`FILL(144)`将用重复的模式`0 0 0 144`填充一个区域。`FILL(22 * 256 + 23)`将用重复的模式`0 0 22 23`填充该区域。如果表达式的结果值大于4个有效字节，则只使用该值的最低4个字节。

**当填充表达式是一个简单的十六进制数时，上述规则不适用**。在这种情况下，不会执行零扩展，所有字节都是有效的。因此，`FILL(0x90)`将用`0x90`重复填充一个区域，没有零字节，`FILL(0x9192)`将用`0x910x92`重复填充该区域。十六进制表达式中的零字节即使在开始时也是重要的，因此`FILL(0x0090)`将用`0x00 0x90`的重复填充一个区域。
十六进制数可以大于4字节，并且所有字节都是有效的，因此`FILL(0x123456789a)`将用重复的5字节序列`0x12 0x34 0x56 0x78 0x9a`填充一个区域。十六进制值中超出区域大小的多余字节将被静默忽略。
以上只适用于指定为' 0x[0-9][a-f][a-f] '的十六进制数。使用' $ '前缀或' h '， ' h '， ' x '或' x '后缀指定的十六进制数将遵循通常情况下的填充值规则。

##### 1.12 Output Section Discarding

链接器通常不会创建没有内容的输出节。例如：
```
.foo : { *(.foo) }
```

当至少有一个输入文件中存在一个名为`.foo`的节，并且输入节不都为空时，链接器才会在输出文件中创建一个`.foo`节。

链接器将忽略丢弃的输出段上的地址分配（见前文 Output Section Address），除非链接脚本在输出节中定义了符号。在这种情况下，链接器将遵守地址分配，即使该节被丢弃，也可能推进`.`(location counter)，这是因为我们在输出段中定义了一个符号，链接器会考虑这个符号的位置。

特殊的输出段名称`/DISCARD/`可用于丢弃输入段。分配给名为`/DISCARD/`输出段的任何输入节都不会包含在输出文件中。 

#### 2 Input Section Description

最常见的输出段命令是输入段描述。

输入段描述是最基本的链接器脚本操作。通过使用输出段，可以告诉链接器如何在内存中布局程序，通过使用输入段描述，可以告诉链接器如何将输入文件映射到内存布局中。

##### 2.1 Input Section Basics
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

#### 参考连接：
【1】[https://sourceware.org/binutils/docs/ld/SECTIONS.html](https://sourceware.org/binutils/docs/ld/SECTIONS.html)