---
title: GNU binary utilities —— objcopy
date: 2024/8/1
categories: 
- GNU-binary-utilities
---

<center>
GNU binary utilities (GNU Binutils) version 2.42: —— objcopy
</center>

<!--more-->

***
**个人笔记，仅供参考！！**
```
objcopy [-F bfdname|--target=bfdname]
        [-I bfdname|--input-target=bfdname]
        [-O bfdname|--output-target=bfdname]
        [-B bfdarch|--binary-architecture=bfdarch]
        [-S|--strip-all]
        [-g|--strip-debug]
        [--strip-unneeded]
        [-K symbolname|--keep-symbol=symbolname]
        [--keep-file-symbols]
        [--keep-section-symbols]
        [-N symbolname|--strip-symbol=symbolname]
        [--strip-unneeded-symbol=symbolname]
        [-G symbolname|--keep-global-symbol=symbolname]
        [--localize-hidden]
        [-L symbolname|--localize-symbol=symbolname]
        [--globalize-symbol=symbolname]
        [--globalize-symbols=filename]
        [-W symbolname|--weaken-symbol=symbolname]
        [-w|--wildcard]
        [-x|--discard-all]
        [-X|--discard-locals]
        [-b byte|--byte=byte]
        [-i [breadth]|--interleave[=breadth]]
        [--interleave-width=width]
        [-j sectionpattern|--only-section=sectionpattern]
        [-R sectionpattern|--remove-section=sectionpattern]
        [--keep-section=sectionpattern]
        [--remove-relocations=sectionpattern]
        [--strip-section-headers]
        [-p|--preserve-dates]
        [-D|--enable-deterministic-archives]
        [-U|--disable-deterministic-archives]
        [--debugging]
        [--gap-fill=val]
        [--pad-to=address]
        [--set-start=val]
        [--adjust-start=incr]
        [--change-addresses=incr]
        [--change-section-address sectionpattern{=,+,-}val]
        [--change-section-lma sectionpattern{=,+,-}val]
        [--change-section-vma sectionpattern{=,+,-}val]
        [--change-warnings] [--no-change-warnings]
        [--set-section-flags sectionpattern=flags]
        [--set-section-alignment sectionpattern=align]
        [--add-section sectionname=filename]
        [--dump-section sectionname=filename]
        [--update-section sectionname=filename]
        [--rename-section oldname=newname[,flags]]
        [--long-section-names {enable,disable,keep}]
        [--change-leading-char] [--remove-leading-char]
        [--reverse-bytes=num]
        [--srec-len=ival] [--srec-forceS3]
        [--redefine-sym old=new]
        [--redefine-syms=filename]
        [--weaken]
        [--keep-symbols=filename]
        [--strip-symbols=filename]
        [--strip-unneeded-symbols=filename]
        [--keep-global-symbols=filename]
        [--localize-symbols=filename]
        [--weaken-symbols=filename]
        [--add-symbol name=[section:]value[,flags]]
        [--alt-machine-code=index]
        [--prefix-symbols=string]
        [--prefix-sections=string]
        [--prefix-alloc-sections=string]
        [--add-gnu-debuglink=path-to-file]
        [--only-keep-debug]
        [--strip-dwo]
        [--extract-dwo]
        [--extract-symbol]
        [--writable-text]
        [--readonly-text]
        [--pure]
        [--impure]
        [--file-alignment=num]
        [--heap=reserve[,commit]]
        [--image-base=address]
        [--section-alignment=num]
        [--stack=reserve[,commit]]
        [--subsystem=which:major.minor]
        [--compress-debug-sections]
        [--decompress-debug-sections]
        [--elf-stt-common=val]
        [--merge-notes]
        [--no-merge-notes]
        [--verilog-data-width=val]
        [-v|--verbose]
        [-V|--version]
        [--help] [--info]
        infile [outfile]
```


**objcopy** 的功能是将一个目标文件的内容复制到另一个目标文件中。它可以读取和写入目标文件，且可以将目标文件的格式转换为与源文件不同的格式。objcopy 的确切行为由命令行选项控制。可以使用不同的选项来指定所需的操作。objcopy 可以在任意两种格式之间复制完全链接的文件，即它可以处理已经链接并包含所有符号和重定位信息的文件。然而，在任意两种格式之间复制可重定位的目标文件可能不会像预期的那样工作。

objcopy 创建临时文件来进行转换，之后删除它们。objcopy使用BFD完成所有的转换工作;它可以访问BFD中描述的所有格式，因此无需明确告知即可识别大多数格式。

objcopy 可以通过使用'srec'的输出目标(例如，使用' -O srec ')来生成S-records。
备注：S-Records 是一种用于表示二进制数据的文本格式。它以字节为单位，包含地址、数据和校验和。每个 S-Record 都以字母 ‘S’ 开头，后面跟着一个数字，表示记录的类型。例如，S0 表示文件头，S1 表示数据记录，S9 表示文件结束。

objcopy可以通过使用' binary '作为输出目标(例如，使用-O binary)来生成原始二进制文件。当objcopy生成原始二进制文件时，它将生成输入对象文件内容的内存转储。所有的符号和重定位信息将被丢弃。内存转储将从复制到输出文件中的最低部分的加载地址开始。

在生成 S-record 或原始二进制文件时，使用-S删除包含调试信息的部分可能会有所帮助。在某些情况下-R将有助于删除包含二进制文件不需要的信息的部分。

注意：objcopy不能更改其输入文件的端序。如果输入格式具有端序(某些格式没有)，objcopy只能将输入复制到具有相同端序或没有端序的文件格式中(例如' srec ')。(参阅——reverse-bytes选项。)

给选项参数如下：


- infile: 输入文件。

- outfile: 输出文件。如果不指定 outfile，objcopy 会创建一个临时文件，并将结果重命名为 infile 的名称。

- -I bfdname || --input-target=bfdname
将源文件的对象格式视为 bfdname，而不是尝试推断它。

- -O bfdname / --output-target=bfdname
使用对象格式 bfdname 写入输出文件。

- -F bfdname / --target=bfdname
使用 bfdname 作为输入和输出文件的对象格式，即从源文件到目标文件的数据传输不进行转换。

- -B bfdarch / --binary-architecture=bfdarch
当将无架构的输入文件转换为对象文件时很有用。在这种情况下，输出架构可以设置为 bfdarch。如果输入文件有已知的 bfdarch，则忽略此选项。
可以通过引用转换过程中创建的特殊符号来访问这些二进制数据。这些符号是：
_binary_objfile_start: 这个符号指向嵌入在对象文件中的二进制数据的起始地址。
_binary_objfile_end: 这个符号指向嵌入在对象文件中的二进制数据的结束地址。
_binary_objfile_size: 这个符号表示嵌入在对象文件中的二进制数据的大小（以字节为单位）。
例如，可以将图片文件转换为对象文件，然后可以使用这些符号在程序中引用和处理这张图片的数据。

- -j sectionpattern / --only-section=sectionpattern
仅从输入文件中复制指定的部分到输出文件。可以多次使用此选项。使用通配符字符。注意，不当使用此选项可能会使输出文件不可用。如果sectionpattern的第一个字符是感叹号(!)，那么匹配的部分将不会被复制，即使先前在同一命令行上使用 --only-section会复制它。例如:
` --only-section=.text.* --only-section=!.text.foo`
将复制所有匹配`.text.*`的段，除了`.text.foo`。

- -R sectionpattern / --remove-section=sectionpattern
从输出文件中删除任何匹配 sectionpattern 的部分。可以多次使用此选项。使用通配符字符。注意，不当使用此选项可能会使输出文件不可用。如果sectionpattern的第一个字符是感叹号(!)，那么匹配的部分将不会被删除，即使早先在同一命令行上使用 --remove-section会删除它。例如:
` --remove-section=.text.* --remove-section=!.text.foo`
将删除与模式`.text.*`匹配的所有段。但不会删除`.text.foo`段。

- –keep-section=sectionpattern
当从输出文件中删除段时，保留匹配 sectionpattern 的段。

- –remove-relocations=sectionpattern
从输出文件中删除任何匹配 sectionpattern 的非动态重定位。可以多次使用此选项。注意，不当使用此选项可能会使输出文件不可用，且尝试删除动态重定位部分（如 .rela.plt）将不起作用。支持通配符字符，例如：
`--remove-relocations=.text.*`
将删除所有匹配 `.text.*` 模式的section的重定位。 如果 sectionpattern 的第一个字符是感叹号（!），则匹配的部分不会删除其重定位。例如：
`--remove-relocations=.text.* --remove-relocations=!.text.foo`
将删除所有匹配 `.text.*` 模式的section的重定位，但不会删除 `.text.foo` 的重定位。

- –strip-section-headers
删除section头部。此选项特定于 ELF 文件，隐含 --strip-all 和 --merge-notes。

- -S / --strip-all
不复制源文件中的重定位和符号信息，同时删除调试段。

- -g / --strip-debug
不复制源文件中的调试符号或调试段。

- –strip-unneeded
除了 --strip-debug 删除的调试符号和调试段外，还删除所有不需要用于重定位处理的符号。

- -K symbolname / --keep-symbol=symbolname
在剥离符号时，保留指定符号（symbolname）。可以多次使用此选项。

- -N symbolname / --strip-symbol=symbolname
不复制源文件中的指定符号（symbolname）。可以多次使用此选项。


- –strip-unneeded-symbol=symbolname
除非符号 symbolname 被重定位需要，否则不复制源文件中的该符号。可以多次使用此选项。

- -G symbolname / --keep-global-symbol=symbolname
仅保留符号 symbolname 为全局符号。将所有其他符号设为本地符号，使其在外部不可见。可以多次使用此选项。注意：此选项不能与 --globalize-symbol 或 --globalize-symbols 选项一起使用。

- –localize-hidden
在 ELF 对象中，将所有具有隐藏或内部可见性的符号标记为本地符号。此选项适用于符号特定的本地化选项，如 -L。

- -L symbolname / --localize-symbol=symbolname
将名为 symbolname 的全局或弱符号转换为本地符号，使其在外部不可见。可以多次使用此选项。注意：唯一符号不会被转换。

- -W symbolname / --weaken-symbol=symbolname
将符号 symbolname 设为弱符号。可以多次使用此选项。

- –globalize-symbol=symbolname
将符号 symbolname 设为全局作用域，使其在定义文件外可见。可以多次使用此选项。注意：此选项不能与 -G 或 --keep-global-symbol 选项一起使用。

- -w / --wildcard
允许在其他命令行选项中使用symbolnames的正则表达式。问号（?）、星号（*）、反斜杠（\）和方括号（[]）操作符可以在符号名的任何位置使用。如果符号名的第一个字符是感叹号（!），则该符号的开关意义相反。例如：
`-w -W !foo -W fo*`
将使 objcopy 将所有以 “fo” 开头的符号设为弱符号，除了符号 “foo”。

- -x / --discard-all
不复制源文件中的非全局符号。

- -X / --discard-locals
不复制编译器生成的本地符号（这些符号通常以 ‘L’ 或 ‘.’ 开头）。

- -b byte / --byte=byte
如果通过 --interleave 选项启用了交错，则从第 *byte* 个字节开始保留字节范围。*byte* 的范围可以是 0 到 *breadth*-1，其中 *breadth* 是通过 --interleave 选项指定的值。

- -i [breadth] / --interleave[=breadth]
仅复制每 breadth 字节中的一个范围（头部数据不受影响）。使用 --byte 选项选择开始复制的字节，使用 --interleave-width 选项选择范围的宽度。此选项对于创建用于编程 ROM 的文件很有用，通常与 *srec* 输出目标一起使用。
默认的交错宽度是 4，因此如果 --byte 设置为 0，objcopy 将每四个字节中的第一个字节从输入复制到输出。

- –interleave-width=width
与 --interleave 选项一起使用时，每次复制 width 个字节。要复制的字节范围的起始位置由 --byte 选项设置，范围的宽度由 --interleave 选项设置。
默认值为 1。*width* 加上 --byte 选项设置的字节值不得超过 --interleave 选项设置的交错宽度。
例如，对于输入“12345678”，将`-b 0 -i 4 --interleave-width=2`和`-b 2 -i 4 --interleave-width=2`传递给两个objcopy命令，那么输出将分别是“1256”和“3478”。

- -p / --preserve-dates
将输出文件的访问和修改日期设置为与输入文件相同。此选项还会复制存储在 PE 格式文件头中的日期，除非定义了 SOURCE_DATE_EPOCH 环境变量。如果定义了此变量，则该变量将用作头部中存储的日期，解释为自 Unix 纪元以来的秒数。

- -D / --enable-deterministic-archives
以确定性模式操作。在复制归档成员和写入归档索引时，使用零作为 UID、GID、时间戳，并为所有文件使用一致的文件模式。如果 binutils 是使用 --enable-deterministic-archives 配置的，则此模式默认开启。可以使用 -U 选项禁用此模式。

- -U / --disable-deterministic-archives
不以确定性模式操作。这是 -D 选项的反义词：在复制归档成员和写入归档索引时，使用它们的实际 UID、GID、时间戳和文件模式值。除非 binutils 是使用 --enable-deterministic-archives 配置的，否则这是默认设置。

- --debugging
如果可能，转换调试信息。这不是默认设置，因为只有某些调试格式受支持，转换过程可能耗时。

- --gap-fill val
用 val 填充section之间的间隙。此操作适用于部分的加载地址（LMA）。通过增加地址较低的section的大小，并用 val 填充创建的额外空间来完成。
- --pad-to address
将输出文件填充直到加载地址 address的区域。通过增加最后一个section的大小来完成。额外的空间用 --gap-fill 指定的值填充（默认值为零）。

- --set-start val
将新文件的起始地址（也称为入口地址）设置为 val。并非所有对象文件格式都支持设置起始地址。

- --change-start incr / --adjust-start incr
通过增加 incr 来更改起始地址（也称为入口地址）。并非所有对象文件格式都支持设置起始地址。

- --change-addresses incr / --adjust-vma incr
通过增加 incr 来更改所有section的 VMA 和 LMA 地址，以及起始地址。某些对象文件格式不允许任意更改 section 地址。

- --change-section-address sectionpattern{=,+,-}val / --adjust-section-vma sectionpattern{=,+,-}val
设置或更改任何匹配 sectionpattern 的section的 VMA 和 LMA 地址。如果使用 ‘=’，则将section地址设置为 val。否则，val 将被加上或减去section地址。如果 sectionpattern 不匹配输入文件中的任何section，将发出警告，除非使用 --no-change-warnings。

- --change-section-lma sectionpattern{=,+,-}val
设置或更改任何匹配 sectionpattern 的section的 LMA 地址。LMA 地址是程序加载时section将被加载到内存中的地址。通常这与 VMA 地址相同，但在某些系统上（特别是程序存储在 ROM 中时），两者可能不同。如果使用 ‘=’，则将section地址设置为 val。否则，val 将被加上或减去section地址。如果 sectionpattern 不匹配输入文件中的任何section，将发出警告，除非使用 --no-change-warnings。

- --change-section-vma sectionpattern{=,+,-}val
设置或更改任何匹配 sectionpattern 的section的 VMA 地址。VMA 地址是程序开始执行后section所在的位置。通常这与 LMA 地址相同，但在某些系统上（特别是程序存储在 ROM 中时），两者可能不同。如果使用 ‘=’，则将section地址设置为 val。否则，val 将被加上或减去section地址。如果 sectionpattern 不匹配输入文件中的任何section，将发出警告，除非使用 --no-change-warnings。
注意：在完全链接的二进制文件中更改section的 VMA 可能是危险的，因为可能有代码期望section位于其旧地址。

- --change-warnings / --adjust-warnings
如果使用 --change-section-address、--change-section-lma 或 --change-section-vma，且section模式不匹配任何section，将发出警告。这是默认设置。

- --no-change-warnings / --no-adjust-warnings
如果使用 --change-section-address、--adjust-section-lma 或 --adjust-section-vma，即使section模式不匹配任何section，也不发出警告。

- --set-section-flags sectionpattern=flags
设置任何匹配 sectionpattern 的section的标志。flags 参数是一个以逗号分隔的标志名称字符串。识别的名称包括 ‘alloc’、‘contents’、‘load’、‘noload’、‘readonly’、‘code’、‘data’、‘rom’、‘exclude’、‘share’、‘debug’ 和 ‘large’。可以为没有内容的section设置 ‘contents’ 标志，但清除有内容的section的 ‘contents’ 标志没有意义——只需删除该section即可。并非所有标志对所有对象文件格式都有意义。特别是 ‘share’ 标志仅对 COFF 格式文件有意义，而对 ELF 格式文件无意义。ELF x86-64 特定标志 ‘large’ 对应于 SHF_X86_64_LARGE。

- --set-section-alignment sectionpattern=align
设置任何匹配 sectionpattern 的 section 的对齐。align 指定对齐的字节数，必须是 2 的幂，例如 1、2、4、8 等。
注意：设置section的对齐不会自动对齐其 LMA 或 VMA 地址。如果这些地址也需要更改，则应使用 --change-section-lma 和/或 --change-section-vma 选项。还要注意，在完全链接的二进制文件中更改 VMA 可能会导致问题，因为可能有代码期望section内容位于其旧地址。

- --add-section sectionname=filename
在复制文件时添加一个名为 sectionname 的新section。新section的内容取自文件 filename。section的大小将是文件的大小。此选项仅适用于支持具有任意名称section的文件格式。注意，可能需要使用 --set-section-flags 选项来设置新创建section的属性。

- -dump-section sectionname=filename
将名为 sectionname 的section内容放入文件 filename 中，覆盖之前可能存在的任何内容。此选项是 --add-section 的反向操作。此选项类似于 --only-section 选项，但它不会创建格式化文件，只是将内容作为原始二进制数据转储，而不应用任何重定位。可以多次指定此选项。

- --update-section sectionname=filename
用文件 filename 的内容替换名为 sectionname 的现有section的内容。section的大小将调整为文件的大小。sectionname 的section标志将保持不变。对于 ELF 格式文件，section到段的映射也将保持不变，这在使用 --remove-section 后跟 --add-section 时是不可能的。可以多次指定此选项。
注意：可以使用 --rename-section 和 --update-section 从一个命令行同时更新和重命名一个section。在这种情况下，将原始section名称传递给 --update-section，并将原始和新section名称传递给 --rename-section。

- --add-symbol name=[section:]value[,flags]
在复制文件时添加一个名为 name 的新符号。此选项可以多次指定。如果给定section，则符号将与该section相关联并相对于该section，否则它将是一个 ABS 符号。指定未定义的section将导致致命错误。不会检查值，将按指定值使用。可以指定符号标志，但并非所有标志对所有对象文件格式都有意义。默认情况下，符号将是全局的。特殊标志 before=othersym 将在指定的 othersym 之前插入新符号，否则符号将按它们在符号表中出现的顺序添加到符号表的末尾。

- --rename-section oldname=newname[,flags]
将section从 oldname 重命名为 newname，并在此过程中可选地将section的标志更改为 flags。与使用链接器脚本执行重命名相比，此选项的优点是输出保持为对象文件，而不会变成链接的可执行文件。此选项接受与 --set-section-flags 选项相同的一组标志。
此选项在输入格式为二进制时特别有用，因为这将始终创建一个名为 .data 的section。例如，如果你想创建一个包含二进制数据的名为 .rodata 的section，可以使用以下命令行来实现：
`objcopy -I binary -O <output_format> -B <architecture>
--rename-section .data=.rodata,alloc,load,readonly,data,contents
<input_binary_file> <output_object_file>`


- --long-section-names {enable,disable,keep}
控制处理 COFF 和 PE-COFF 对象格式时长section名称的处理。默认行为 ‘keep’ 是在输入文件中存在长section名称时保留它们。‘enable’ 和 ‘disable’ 选项强制启用或禁用输出对象中长section名称的使用；当 ‘disable’ 生效时，输入对象中的任何长section名称将被截断。‘enable’ 选项仅在输入中存在长section名称时才会发出长section名称；这与 ‘keep’ 基本相同，但是，' enable '选项是否会强制在输出文件中创建一个空字符串表还没有定义。

- --change-leading-char
一些对象文件格式在符号开头使用特殊字符。最常见的字符是下划线，编译器通常在每个符号前添加下划线。此选项告诉 objcopy 在转换对象文件格式时更改每个符号的前导字符。如果对象文件格式使用相同的前导字符，则此选项无效。否则，它将添加、删除或更改字符。

- --remove-leading-char
如果全局符号的第一个字符是对象文件格式使用的特殊符号前导字符，则删除该字符。最常见的符号前导字符是下划线。此选项将从所有全局符号中删除前导下划线。如果你想将不同文件格式的对象链接在一起，而这些格式对符号名称有不同的约定，这会很有用。这与 --change-leading-char 不同，因为它总是在适当时更改符号名称，而不管输出文件的对象文件格式如何。

- --reverse-bytes=num
反转具有输出内容的section中的字节。section长度必须能被给定值整除才能进行交换。反转在交错之前进行。
此选项通常用于为有问题的目标系统生成 ROM 映像。例如，在某些目标板上，从 8 位 ROM 获取的 32 位字无论 CPU 字节顺序如何，都会以小端字节顺序重新组装。根据编程模型，可能需要修改 ROM 的字节顺序。 考虑一个包含以下八个字节的部分的简单文件：12345678。 使用 --reverse-bytes=2，输出文件中的字节顺序将为 21436587。 使用 --reverse-bytes=4，输出文件中的字节顺序将为 43218765。 先使用 --reverse-bytes=2，然后在输出文件上使用 --reverse-bytes=4，第二个输出文件中的字节顺序将为 34127856。

- --srec-len=ival
仅对 srec 输出有意义。设置生成的 Srecords 的最大长度为 ival。此长度涵盖地址、数据和 CRC 字段。

- --srec-forceS3
仅对 srec 输出有意义。避免生成 S1/S2 记录，仅创建 S3 记录格式。

- --redefine-sym old=new
将符号 old 的名称更改为 new。当你试图链接两个没有源代码的东西并且存在名称冲突时，这会很有用。

- --redefine-syms=filename
对文件 filename 中列出的每对符号 “old new” 应用 --redefine-sym。filename 文件中，每行一个为符号对。行注释可以由井号字符引入。此选项可以多次使用。

- --weaken
将文件中的所有全局符号更改为弱符号。当使用 -R 选项链接器链接其它对象时，这会很有用。此选项仅在使用支持弱符号的对象文件格式时有效。

- --keep-symbols=filename
对文件 filename 中列出的每个符号应用 --keep-symbol 选项。filename 文件中每行一个符号名称。行注释可以由井号字符引入。此选项可以多次使用。

- --strip-symbols=filename
对文件 filename 中列出的每个符号应用 --strip-symbol 选项。filename 文件中每行一个符号名称。行注释可以由井号字符引入。此选项可以多次使用。

- --strip-unneeded-symbols=filename
对文件 filename 中列出的每个符号应用 --strip-unneeded-symbol 选项。filename 文件中每行一个符号名称。行注释可以由井号字符引入。此选项可以多次使用。

- --keep-global-symbols=filename
对文件 filename 中列出的每个符号应用 --keep-global-symbol 选项。filename 文件中每行一个符号名称。行注释可以由井号字符引入。此选项可以多次使用。

- --localize-symbols=filename
对文件 filename 中列出的每个符号应用 --localize-symbol 选项。filename 文件中每行一个符号名称。行注释可以由井号字符引入。此选项可以多次使用。

- --globalize-symbols=filename
对文件 filename 中列出的每个符号应用 --globalize-symbol 选项。filename 文件中每行一个符号名称。行注释可以由井号字符引入。此选项可以多次使用。注意：此选项不能与 -G 或 --keep-global-symbol 选项一起使用。

- --weaken-symbols=filename
对文件 filename 中列出的每个符号应用 --weaken-symbol 选项。filename 只是一个平面文件，每行一个符号名称。行注释可以由井号字符引入。此选项可以多次使用。

- --alt-machine-code=index
如果输出架构有备用机器代码，则使用第 index 个代码而不是默认代码。这在机器被分配了一个官方代码并且工具链采用了新代码，但其他应用程序仍依赖于使用原始代码时很有用。对于基于 ELF 的架构，如果不存在 index 备用，则将该值视为要存储在 ELF 头部的 e_machine 字段中的绝对数值。

- --writable-text
将输出文本标记为可写。此选项对所有对象文件格式都没有意义。

- --readonly-text
将输出文本设置为写保护。此选项对所有对象文件格式都没有意义。

- --pure
将输出文件标记为按需分页。此选项对所有对象文件格式都没有意义。

- --impure
将输出文件标记为不纯。此选项对所有对象文件格式都没有意义。

- --prefix-symbols=string
在输出文件中的所有符号前加上 string 前缀。

- --–prefix-sections=string
在输出文件中的所有section名称前加上 string 前缀。

- --prefix-alloc-sections=string
在输出文件中所有已分配section的名称前加上 string 前缀。

- --add-gnu-debuglink=path-to-file
创建一个 .gnu_debuglink section，其中包含对 path-to-file 的引用，并将其添加到输出文件中。注意：path-to-file 中的文件必须存在。添加 .gnu_debuglink section的过程包括将调试信息文件内容的校验和嵌入到该部分中。
如果调试信息文件在一个位置构建，但稍后将安装到不同的位置，则不要使用安装位置的路径。--add-gnu-debuglink 选项将失败，因为安装文件尚不存在。相反，将调试信息文件放在当前目录中，并使用不带任何目录组件的 --add-gnu-debuglink 选项，如下所示：
`objcopy --add-gnu-debuglink=foo.debug`
在调试时，调试器将尝试在一组已知位置查找单独的调试信息文件。这些位置的确切集合因所使用的发行版而异，但通常包括：
与可执行文件相同的目录。
包含可执行文件的目录的子目录，称为 .debug。
全局调试目录，如 /usr/lib/debug。
只要在运行调试器之前将调试信息文件安装到这些位置之一，一切都应该正常工作。

- --keep-section-symbols
在剥离文件时（例如使用 --strip-debug 或 --strip-unneeded），保留指定section名称的任何符号，否则这些符号将被剥离。

- --keep-file-symbols
在剥离文件时（例如使用 --strip-debug 或 --strip-unneeded），保留指定源文件名称的任何符号，否则这些符号将被剥离。

- --only-keep-debug
剥离文件，移除任何不会被 --strip-debug 剥离的section内容，并保留调试section。在 ELF 文件中，这会保留输出中的所有注释section。
注意：被剥离的section的section头被保留，包括它们的大小，但section内容被丢弃。保留section头是为了让其他工具能够将调试信息文件与真实可执行文件匹配，即使该可执行文件已被重新定位到不同的地址空间。 此选项的目的是与 --add-gnu-debuglink 一起使用，以创建一个两部分的可执行文件。一个是剥离的二进制文件，占用较少的 RAM 和分发空间，另一个是仅在需要调试功能时才需要的调试信息文件。创建这些文件的建议步骤如下：
1：正常链接可执行文件。假设它被称为 foo
2：objcopy --only-keep-debug foo foo.dbg  创建包含调试信息的文件
3：objcopy --strip-debug foo 创建一个剥离调试信息的可执行文件
4：objcopy --add-gnu-debuglink=foo.dbg foo  将调试信息链接添加到剥离的可执行文件中
注意：选择 .dbg 作为调试信息文件的扩展名是任意的。--only-keep-debug 步骤是可选的。也可以这样做：
1：正常链接可执行文件
2：cp foo foo.full
3：objcopy --strip-debug foo
3：objcopy --add-gnu-debuglink=foo.full foo
即，--add-gnu-debuglink 指向的文件可以是完整的可执行文件。它不必是由 --only-keep-debug 选项创建的文件。
注意：此选项仅用于完全链接的文件。对调试信息可能不完整的目标文件使用它没有意义。此外，gnu_debuglink特性目前只支持包含调试信息的一个文件名，而不是每个对象文件一个的多个文件名。

- --strip-dwo
移除所有 DWARF .dwo section的内容，保留剩余的调试部分和所有符号。此选项旨在由编译器作为 -gsplit-dwarf 选项的一部分使用，该选项将调试信息分割在 .o 文件和单独的 .dwo 文件之间。编译器在同一个文件中生成所有调试信息，然后使用 --extract-dwo 选项将 .dwo section复制到 .dwo 文件中，然后使用 --strip-dwo 选项从原始 .o 文件中移除这些部分。

- --extract-dwo
提取所有 DWARF .dwo 部分的内容。有关更多信息，请参见 --strip-dwo 选项。

- --file-alignment num
指定文件对齐。文件中的section将始终从该数字的倍数的文件偏移开始。默认值为 512。[此选项仅适用于 PE 目标。]

- --heap reserve / --heap reserve,commit
指定要保留（和可选提交）用于此程序的堆的内存字节数。[此选项仅适用于 PE 目标。]

- --image-base value
使用 value 作为程序或 DLL 的基地址。这是加载程序或 DLL 时将使用的最低内存位置。为了减少重定位的需要并提高 DLL 的性能，每个 DLL 应具有唯一的基地址，并且不应与其他 DLL 重叠。默认值为 0x400000（用于可执行文件），0x10000000（用于 DLL）。[此选项仅适用于 PE 目标。]

- --section-alignment num
[此选项仅适用于 PE 目标。]
设置 PE 头中的section对齐字段（如果二进制文件中存在）。内存中的section将始终从该数字的倍数的地址开始。默认值为 0x1000。
注意：此选项还将设置每个section标志中的对齐字段。
注意：如果section的 LMA 或 VMA 地址不再对齐，并且这些地址未通过 --set-section-lma 或 --set-section-vma 选项设置，并且文件已完全重定位，则会发出警告消息。然后由用户决定是否需要更新 LMA 和 VMA。

- --stack reserve / --stack reserve,commit
指定要保留（和可选提交）用于此程序的堆栈的内存字节数。[此选项仅适用于 PE 目标。]

- --subsystem which / --subsystem which:major / --subsystem which:major.minor
指定程序将在哪个子系统下执行。which 的合法值包括 native、windows、console、posix、efi-app、efi-bsd、efi-rtd、sal-rtd 和 xbox。你还可以选择设置子系统版本。which 也接受数值。[此选项仅适用于 PE 目标。]

- --extract-symbol
保留文件的section标志和符号，但移除所有section数据。具体来说，此选项：
移除所有section的内容；
将每个section的大小设置为零；
将文件的起始地址设置为零。
此选项用于为 VxWorks 内核构建 .sym 文件。它也可以是减少 --just-symbols 链接器输入文件大小的有用方法。

- --compress-debug-sections
使用 zlib 和 ELF ABI 的 SHF_COMPRESSED 压缩 DWARF 调试部分。注意：如果压缩实际上会使 section 变大，则不会压缩该部分。

- --compress-debug-sections=none / --compress-debug-sections=zlib / --compress-debug-sections=zlib-gnu / --compress-debug-sections=zlib-gabi / --compress-debug-sections=zstd
对于 ELF 文件，这些选项控制 DWARF 调试部分的压缩方式。--compress-debug-sections=none 等同于 --decompress-debug-sections。--compress-debug-sections=zlib 和 --compress-debug-sections=zlib-gabi 等同于 --compress-debug-sections。--compress-debug-sections=zlib-gnu 使用已废弃的 zlib-gnu 格式压缩 DWARF 调试部分。调试部分将重命名为以 .zdebug 开头。--compress-debug-sections=zstd 使用 zstd 压缩 DWARF 调试部分。注意：如果压缩实际上会使部分变大，则不会压缩或重命名该部分。

- --decompress-debug-sections
解压缩 DWARF 调试部分。对于 .zdebug 部分，恢复原始名称。

- --elf-stt-common=yes / --elf-stt-common=no
对于 ELF 文件，这些选项控制是否应将公共符号转换为 STT_COMMON 或 STT_OBJECT 类型。--elf-stt-common=yes 将公共符号类型转换为 STT_COMMON。--elf-stt-common=no 将公共符号类型转换为 STT_OBJECT。

- --merge-notes / --no-merge-notes
对于 ELF 文件，尝试（或不尝试）通过删除重复的注释来减少任何 SHT_NOTE 类型部分的大小。

- -V / --version
显示 objcopy 的版本号。

- --verilog-data-width=bytes
对于 Verilog 输出，此选项控制每个输出数据元素转换的字节数。输入目标控制转换的字节顺序。

- -v / --verbose
详细输出：列出所有修改的对象文件。对于档案文件，objcopy -V 列出档案的所有成员。

- --help
显示 objcopy 选项的摘要。

- --info
显示所有可用架构和对象格式的列表。


#### 参考连接：
【1】[https://sourceware.org/binutils/docs/binutils/objcopy.html](https://sourceware.org/binutils/docs/binutils/objcopy.html)