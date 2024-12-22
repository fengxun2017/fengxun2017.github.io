---
title: GNU binary utilities —— strip
date: 2024/11/25
categories: 
- GNU-binary-utilities
---

<center>
GNU binary utilities (GNU Binutils) version 2.42: —— strip
</center>

<!--more-->

***
**个人笔记，仅供参考！！**
strip [-F bfdname |--target=bfdname]
      [-I bfdname |--input-target=bfdname]
      [-O bfdname |--output-target=bfdname]
      [-s|--strip-all]
      [-S|-g|-d|--strip-debug]
      [--strip-dwo]
      [-K symbolname|--keep-symbol=symbolname]
      [-M|--merge-notes][--no-merge-notes]
      [-N symbolname |--strip-symbol=symbolname]
      [-w|--wildcard]
      [-x|--discard-all] [-X |--discard-locals]
      [-R sectionname |--remove-section=sectionname]
      [--keep-section=sectionpattern]
      [--remove-relocations=sectionpattern]
      [--strip-section-headers]
      [-o file] [-p|--preserve-dates]
      [-D|--enable-deterministic-archives]
      [-U|--disable-deterministic-archives]
      [--keep-section-symbols]
      [--keep-file-symbols]
      [--only-keep-debug]
      [-v |--verbose] [-V|--version]
      [--help] [--info]
      objfile…

GNU strip 删除目标文件 objfile 中的所有符号。目标文件列表可能包括归档文件。必须至少给定一个目标文件。
strip 会修改其参数中指定的文件，而不是写入带有不同名称的修改副本。

选项：

##### -F bfdname --target=bfdname
将原始 objfile 作为具有对象代码格式 bfdname 的文件处理，并以相同格式重写它。有关详细信息，请参阅target selection。

##### --help 
显示 strip 的选项摘要并退出。

##### --info 
显示一个列表，列出所有可用的架构和对象格式。

##### -I bfdname --input-target=bfdname 
将原始 objfile 作为具有对象代码格式 bfdname 的文件处理。有关详细信息，请参阅target select。

##### -O bfdname --output-target=bfdname 
将 objfile 替换为 bfdname 格式的文件。有关详细信息，请参阅target select。

##### -R sectionname --remove-section=sectionname 
从输出文件中移除任何名为 sectionname 的段，除了会被删除的其他段外。此选项可以多次给出。请注意，不当使用此选项可能会使输出文件无法使用。通配符 * 可以在 sectionname 的末尾给出。如果是这样，则任何以 sectionname 开头的段都将被移除。

如果 sectionpattern 的第一个字符是感叹号（!），则即使命令行中较早使用的 --remove-section 会删除它，匹配的段也不会被删除。例如：

--remove-section=.text.* --remove-section=!.text.foo 会删除所有匹配模式 .text.* 的段，但不会删除段 .text.foo。

##### --keep-section=sectionpattern
从输出文件中移除段时，保留与 sectionpattern 匹配的段。

##### --remove-relocations=sectionpattern 
删除输出文件中任何匹配 sectionpattern 的段的重定位。此选项可以多次给出。请注意，不当使用此选项可能会使输出文件无法使用。sectionpattern 中接受通配符。例如：

--remove-relocations=.text.* 会删除所有匹配模式 .text.* 的段的重定位。

如果 sectionpattern 的第一个字符是感叹号（!），则即使命令行中较早使用的 --remove-relocations 会删除重定位，匹配的段也不会删除其重定位。例如：

--remove-relocations=.text.* --remove-relocations=!.text.foo 会删除所有匹配模式 .text.* 的段的重定位，但不会删除段 .text.foo 的重定位。

##### --strip-section-headers 
剥离段标头。此选项仅对 ELF 文件有效。隐含 --strip-all 和 --merge-notes。

##### -s --strip-all 
删除所有符号。

##### -g -S -d --strip-debug 
仅删除调试符号。

##### --strip-dwo 
删除所有 DWARF .dwo 段的内容，保留其余的调试段和所有符号。有关详细信息，请参阅 objcopy 部分中的说明。

##### --strip-unneeded 
除了通过 --strip-debug 删除的调试符号和段外，还删除所有不需要重定位处理的符号。

##### -K symbolname --keep-symbol=symbolname 
删除符号时，保留符号 symbolname，即使它通常会被删除。此选项可以多次给出。

##### -M --merge-notes --no-merge-notes 
对于 ELF 文件，尝试（或不尝试）通过删除重复的注释来减小任何 SHT_NOTE 类型段的大小。默认情况下，除非剥离调试或 DWO 信息，否则尝试进行此简化。

##### -N symbolname --strip-symbol=symbolname 
从源文件中删除符号 symbolname。此选项可以多次给出，并且可以与其他剥离选项（除了 -K）结合使用。

##### -o file 
将剥离的输出放入 file 中，而不是替换现有文件。使用此参数时，只能指定一个 objfile 参数。

##### -p --preserve-dates 
保留文件的访问和修改日期。

##### -D --enable-deterministic-archives 
以确定性模式操作。在复制归档成员和写入归档索引时，使用零 UID、GID、时间戳，并为所有文件使用一致的文件模式。

如果 binutils 使用 --enable-deterministic-archives 配置，则此模式默认启用。可以使用下面的 -U 选项禁用它。

##### -U --disable-deterministic-archives 
不以确定性模式操作。这是上述 -D 选项的反义词：在复制归档成员和写入归档索引时，使用它们的实际 UID、GID、时间戳和文件模式值。

除非 binutils 使用 --enable-deterministic-archives 配置，否则这是默认值。

##### -w --wildcard 
允许在其他命令行选项中使用 symbolnames 中的正则表达式。问号（?）、星号（*）、反斜杠（\）和方括号（[]）运算符可以在符号名称的任何位置使用。如果符号名称的第一个字符是感叹号（!），则对该符号的开关意义被反转。例如：

-w -K !foo -K fo* 将导致 strip 只保留以“fo”开头的符号，但丢弃符号“foo”。

##### -x --discard-all 
删除非全局符号。

##### -X --discard-locals 
删除编译器生成的本地符号。（这些通常以“L”或“.”开头。）

##### --keep-section-symbols 
剥离文件时，保留任何指定段名称的符号，否则这些符号将被剥离，可能与 --strip-debug 或 --strip-unneeded 一起使用。

##### --keep-file-symbols 
剥离文件时，保留任何指定源文件名称的符号，否则这些符号将被剥离，可能与 --strip-debug 或 --strip-unneeded 一起使用。

##### --only-keep-debug 
剥离文件，清空 --strip-debug 不会剥离的任何段的内容，并保留调试段。在 ELF 文件中，这也保留了输出中的所有注释段。

注意：剥离段的段标头会被保留，包括它们的大小，但段的内容会被丢弃。保留段标头是为了让其他工具可以将调试信息文件与实际可执行文件匹配，即使该可执行文件已重新定位到不同的地址空间。

此选项的意图是与 --add-gnu-debuglink 一起使用，以创建一个两部分的可执行文件。一个是剥离的二进制文件，它占用的 RAM 和分发空间较少，另一个是调试信息文件，仅在需要调试功能时才需要。创建这些文件的建议程序如下：

正常链接可执行文件，假设它被称为 foo，然后…… 运行 objcopy --only-keep-debug foo foo.dbg 以创建包含调试信息的文件。 运行 objcopy --strip-debug foo 创建一个剥离的可执行文件。 运行 objcopy --add-gnu-debuglink=foo.dbg foo 将调试信息链接添加到剥离的可执行文件中。 注意：.dbg 作为调试信息文件的扩展名是任意的。--only-keep-debug 步骤是可选的。你可以这样做：

正常链接可执行文件。将 foo 复制为 foo.full 运行 strip --strip-debug foo 运行 objcopy --add-gnu-debuglink=foo.full foo 即，由 --add-gnu-debuglink 指向的文件可以是完整的可执行文件。它不必是通过 --only-keep-debug 选项创建的文件。

注意：此选项仅适用于完全链接的文件。它在调试信息可能不完整的目标文件上使用是没有意义的。此外，gnu_debuglink 功能目前仅支持存在一个包含调试信息的文件，而不是每个目标文件一个文件名。

##### -V --version 
显示 strip 的版本号。

##### -v --verbose 
详细输出：列出所有修改的目标文件。在归档的情况下，strip -v 列出归档的所有成员。

#### 参考连接：
【1】[https://sourceware.org/binutils/docs-2.42/binutils/strip.html](https://sourceware.org/binutils/docs-2.42/binutils/strip.html)