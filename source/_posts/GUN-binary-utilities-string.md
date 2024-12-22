---
title: GNU binary utilities —— strings
date: 2024/11/21
categories: 
- GNU-binary-utilities
---

<center>
GNU binary utilities (GNU Binutils) version 2.42: —— strings
</center>

<!--more-->

***
**个人笔记，仅供参考！！**

strings [-afovV] [-min-len] [-n min-len] [--bytes=min-len] [-t radix] [--radix=radix] [-e encoding] [--encoding=encoding] [-U method] [--unicode=method] [-] [--all] [--print-file-name] [-T bfdname] [--target=bfdname] [-w] [--include-all-whitespace] [-s] [--output-separator sep_string] [--help] [--version] file…

对于给定的每个文件，GNU strings 会打印至少包含4个字符的可打印字符序列（或者根据下面的选项指定的字符数）且后面跟有一个不可打印字符。

根据strings程序的配置，它默认要么显示每个文件中能找到的所有可打印序列，要么仅显示可加载的已初始化数据段中的序列。如果文件类型无法识别，或者strings从标准输入读取，它将始终显示能找到的所有可打印序列。

为了向后兼容，任何仅有一个 - 命令行选项之后的文件也将被完整扫描，无论是否存在任何 -d 选项。

strings 主要用于确定非文本文件的内容。

选项：
#### -a / --all / -
扫描整个文件，无论其包含哪些段，或这些段是否已加载或初始化。通常这是默认行为，但strings可以配置为 -d 为默认行为。

#### -d / --data 
只打印文件中已初始化的加载数据段中的字符串。这可能减少输出中的垃圾，但也可能暴露strings程序使用的BFD库中的任何安全漏洞。strings可以配置为此选项为默认行为，在这种情况下，可以使用 -a 选项来避免使用BFD库，只打印文件中的所有字符串。

#### -f / --print-file-name
在每个字符串前打印文件名。

#### --help
打印程序使用的摘要到标准输出并退出。

#### -*min-len* / -n *min-len* / --bytes=*min-len*
打印至少包含*min-len*个字符的可显示字符序列。如果未指定，默认最小长度为4。可显示和不可显示字符的区别取决于-e和-U选项的设置。序列总是以控制字符（如换行和回车）为终止符，但不包括制表符。

#### -o 
类似于 `-t o`。一些其它版本的strings将 -o 作用于 `-t d`。

#### -t radix / --radix=radix
在每个字符串前打印文件中的偏移量。单个字符参数指定偏移量的基数——‘o’ 表示八进制，‘x’ 表示十六进制，‘d’ 表示十进制。

#### -e *encoding* / --encoding=*encoding*
选择要查找的字符串的字符编码。*encoding* 的可能值包括：‘s’ = 单字节7位字符（默认），‘S’ = 单字节8位字符，‘b’ = 16位大端序，‘l’ = 16位小端序，‘B’ = 32位大端序，‘L’ = 32位小端序。对于查找宽字符字符串非常有用。（‘l’和‘b’适用于例如Unicode UTF-16/UCS-2编码）。

#### -U [d|i|l|e|x|h] / --unicode=[default|invalid|locale|escape|hex|highlight]
控制strings中 UTF-8 编码多字节字符的显示。默认（--unicode=default）是不作特殊处理，而依赖于--encoding选项的设置。其他选项值自动启用--encoding=S。
--unicode=invalid 选项将它们视为非图形字符，因此不属于有效字符串。其余选项将它们视为有效字符串字符。
--unicode=locale 选项在当前语言环境中显示它们，这可能支持也可能不支持UTF-8编码。--unicode=hex 选项将它们显示为括在<>字符之间的十六进制字节序列。--unicode=escape 选项将它们显示为转义序列（\uxxxx），--unicode=highlight 选项将它们显示为红色高亮的转义序列（如果输出设备支持）。着色旨在引起注意，以便在不期望存在unicode序列的地方发现它们。

#### -T bfdname / --target=bfdname
指定一种不同于系统默认格式的目标代码格式。详见 Target Selection。

#### -v / -V / --version
打印程序版本号到标准输出并退出。

#### -w / --include-all-whitespace
默认情况下，制表符和空格字符包含在显示的字符串中，但其他空白字符（如换行和回车）不包括在内。-w 选项将其更改为所有空白字符都视为字符串的一部分。

#### -s / --output-separator
默认情况下，输出字符串由换行符分隔。此选项允许你提供任意字符串作为输出记录分隔符。对于--include-all-whitespace非常有用，字符串内部可能包含换行符。

#### 参考连接：
【1】[https://sourceware.org/binutils/docs-2.42/binutils/strings.html](https://sourceware.org/binutils/docs-2.42/binutils/strings.html)