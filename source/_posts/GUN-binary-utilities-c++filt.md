---
title: GNU binary utilities —— c++filt
date: 2024/11/27
categories: 
- GNU-binary-utilities
---

<center>
GNU binary utilities (GNU Binutils) version 2.42: —— c++filt
</center>

<!--more-->

***
**个人笔记，仅供参考！！**
c++filt [-_|--strip-underscore]
        [-n|--no-strip-underscore]
        [-p|--no-params]
        [-t|--types]
        [-i|--no-verbose]
        [-r|--no-recurse-limit]
        [-R|--recurse-limit]
        [-s format|--format=format]
        [--help]  [--version]  [symbol…]


C++ 和 Java 语言提供了函数重载的功能，这意味着你可以编写许多具有相同名称的函数，只要每个函数的参数类型不同即可。为了能够区分这些同名的函数，C++ 和 Java 将它们编码为低级汇编名称，这些名称唯一标识每个不同的版本。这一过程称为“符号重整”。c++filt 程序进行相反的映射：它将低级名称解码（去除重整），使其成为用户级别的名称，以便于阅读。

输入中的每个字母数字单词（由字母、数字、下划线、美元符号或句点组成）都是一个潜在的重整名称。如果名称解码为 C++ 名称，那么 C++ 名称将替换输出中的低级名称，否则输出原始单词。通过这种方式，你可以将包含重整名称的整个汇编源文件传递给 c++filt，并看到包含解码名称的相同源文件。

你还可以通过命令行传递符号来使用 c++filt 解码单个符号:`c++filt symbol`

如果未提供符号参数，则 c++filt 从标准输入读取符号名称。所有结果都会打印在标准输出上。从命令行读取名称与从标准输入读取名称的区别在于，命令行参数预期只是重整名称，不会进行检查以将它们与周围文本分开。例如：`c++filt -n _Z1fv`，将正确解码名称为 f()，而：`c++filt -n _Z1fv,`,将无法工作。（注意，重整名称末尾的额外逗号使其无效）。然而，以下命令是有效的：`echo _Z1fv, | c++filt -n`
并将显示 `f(),`，即解码后的名称后跟一个尾随逗号。这种行为是因为当从标准输入读取名称时，假设它们可能是汇编源文件的一部分，其中可能包含在重整名称后面的额外字符。例如：
`.type   _Z1fv, @function`


选项：
##### -_ / --strip-underscore
在某些系统上，C 和 C++ 编译器会在每个名称前加一个下划线。例如，C 名称 foo 得到低级名称 _foo。此选项删除初始下划线。c++filt 是否默认删除下划线取决于目标。

##### -n / --no-strip-underscore
不要删除初始下划线。

##### -p --no-params
在解码函数名称时，不显示函数参数的类型。

##### -t --types 
尝试解码类型和函数名称。默认情况下此选项被禁用，因为重整类型通常仅在编译器内部使用，并且它们可能与未重整的名称混淆。例如，一个名为 a 的函数被视为重整类型名称时，会解码为 signed char。

##### -i --no-verbose
不要在解码输出中包含实现细节（如果有）。

##### -r / -R / --recurse-limit / --no-recurse-limit / --recursion-limit / --no-recursion-limit 
启用或禁用解码字符串时执行的递归量限制。由于名称重整格式允许无限递归级别，可能会创建一些字符串，其解码会耗尽主机机器上的堆栈空间，从而触发内存故障。该限制试图通过将递归限制为2048级嵌套来防止这种情况发生。

默认情况下启用此限制，但在解码非常复杂的名称时，可能需要禁用它。然而请注意，如果禁用递归限制，则可能发生堆栈耗尽，并且任何有关此类事件的错误报告都会被拒绝。

-r 选项是 --no-recurse-limit 选项的同义词。-R 选项是 --recurse-limit 选项的同义词。

##### -s format / --format=format 
c++filt 可以解码由不同编译器使用的各种重整方法。此选项的参数选择它使用的方法：

- auto：基于可执行文件的自动选择（默认方法）

- gnu：GNU C++ 编译器（g++）使用的方法

- lucid：Lucid 编译器（lcc）使用的方法

- arm：C++ 注释参考手册指定的方法

- hp：HP 编译器（aCC）使用的方法

- edg：EDG 编译器使用的方法

- gnu-v3：GNU C++ 编译器（g++）使用的 V3 ABI 方法

- java：GNU Java 编译器（gcj）使用的方法

- gnat：GNU Ada 编译器（GNAT）使用的方法

##### --help 
打印 c++filt 选项的摘要并退出。

##### --version 
打印 c++filt 的版本号并退出。

警告：c++filt 是一个新的工具，其用户界面的细节可能会在未来版本中发生变化。特别是，将来可能需要一个命令行选项来解码作为命令行参数传递的名称；换句话说，`c++filt symbol`在未来的版本中可能会变成：`c++filt option symbol`


#### 参考连接：
【1】[https://sourceware.org/binutils/docs-2.42/binutils/c_002b_002bfilt.html](https://sourceware.org/binutils/docs-2.42/binutils/c_002b_002bfilt.html)