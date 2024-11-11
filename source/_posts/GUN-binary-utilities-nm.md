---
title: GNU binary utilities —— nm
date: 2024/7/30
categories: 
- GNU-binary-utilities
---

<center>
GNU binary utilities (GNU Binutils) version 2.42: —— nm
</center>

<!--more-->

***
**个人笔记，仅供参考！！**
```
nm [-A|-o|--print-file-name]
   [-a|--debug-syms]
   [-B|--format=bsd]
   [-C|--demangle[=style]]
   [-D|--dynamic]
   [-fformat|--format=format]
   [-g|--extern-only]
   [-h|--help]
   [--ifunc-chars=CHARS]
   [-j|--format=just-symbols]
   [-l|--line-numbers] [--inlines]
   [-n|-v|--numeric-sort]
   [-P|--portability]
   [-p|--no-sort]
   [-r|--reverse-sort]
   [-S|--print-size]
   [-s|--print-armap]
   [-t radix|--radix=radix]
   [-u|--undefined-only]
   [-U|--defined-only]
   [-V|--version]
   [-W|--no-weak]
   [-X 32_64]
   [--no-demangle]
   [--no-recurse-limit|--recurse-limit]]
   [--plugin name]
   [--size-sort]
   [--special-syms]
   [--synthetic]
   [--target=bfdname]
   [--unicode=method]
   [--with-symbol-versions]
   [--without-symbol-versions]
   [objfile…]
```

nm 是 GNU 工具，用于列出目标文件（objfile）中的符号。如果没有列出目标文件作为参数，nm 将假定文件为 a.out。

对于每个符号，nm 会显示：
：**符号的值**（默认为十六进制）。
：**符号类型**。如果是小写字母，则符号通常是局部的；如果是大写字母，则符号是全局的（外部的）。但是，有一些小写字母符号用于显示特殊的全局符号（u、v 和 w）。
：**符号名**。如果符号具有与之关联的版本信息，则也会显示版本信息。如果版本化的符号未定义或对链接器隐藏，则版本字符串将显示为符号名称的后缀，前面是@字符。例如' foo@VER_1 '。如果该版本是解析对该符号的未版本引用时使用的默认版本，那么它将显示为前面有两个@字符的后缀。例如'foo@@VER_2'

符号类型至少使用以下类型，其它类型也可能根据目标文件格式而异
- A：符号的值是绝对的，不会被进一步链接更改。

- B、b：符号位于 BSS 数据段。该段通常包含零初始化或未初始化的数据。

- C、c：符号是 common 符号。Common 符号是未初始化的数据。在链接时，多个 common 符号可能具有相同的名称。如果符号在任何地方定义，common 符号将被视为未定义的引用。有关 common 符号的更多详细信息，请参阅 GNU 链接器中的链接器选项中的 `--warn-common` 。当符号位于用于小 common 符号的特殊段中时，使用小写字母 c。

- D、d：符号位于已初始化的数据段。

- G、g：符号位于已初始化的数据段，用于小对象。某些目标文件格式允许更有效地访问小数据对象，例如全局 int 变量，而不是大型全局数组。

- i：对于 PE 格式文件，表示符号位于专用于 DLL 实现的段中。对于 ELF 格式文件，表示符号是一个间接函数。这是 GNU 对 ELF 符号类型的标准集的扩展。它表示一个符号，如果由重定位引用，则不会计算其地址，而必须在运行时调用。运行时执行将返回用于重定位的值。
注意：GNU 间接符号的实际显示由 --ifunc-chars 命令行选项控制。如果提供了此选项，则字符串中的第一个字符将用于全局间接函数符号。如果字符串包含第二个字符，则将用于本地间接函数符号。

- I：符号是对另一个符号的间接引用。
  
- N：符号是调试符号。

- n：符号位于非数据、非代码、非调试的只读段中。

- p：符号位于堆栈展开段中。

- R、r：符号位于只读数据段中。
  
- S、s：符号位于未初始化或零初始化的数据段，用于小对象。

- T、t：符号位于文本（代码）段中。

- U：符号未定义。

- u：符号是唯一的全局符号。这是对 ELF 符号绑定标准集的 GNU 扩展。对于这样的符号，动态链接器将确保在整个进程中只有一个具有此名称和类型的符号在使用。

- V、v：符号是弱对象。当弱定义的符号与正常定义的符号链接时，正常定义的符号将被使用，不会出错。当链接弱未定义的符号且该符号未定义时，弱符号的值变为零，也不会出错。在某些系统上，大写表示已指定默认值。

- W、w：符号是未明确标记为弱对象符号的弱符号。当弱定义的符号与正常定义的符号链接时，正常定义的符号将被使用，不会出错。当链接弱未定义的符号且该符号未定义时，符号的值将以系统特定的方式确定，也不会出错。在某些系统上，大写表示已指定默认值。

- -：符号是 a.out 对象文件中的 stabs 符号。在这种情况下，接下来打印的值是 stabs 的其他字段、stabs 的描述字段和 stab 类型。Stabs 符号用于保存调试信息。
  
- ?：符号类型未知，或者是特定于对象文件格式的。


`nm`的命令行参数有：
- -A、o、--print-file-name：在每个符号之前显示它所在的输入文件（或存档成员）的名称，而不是仅在所有符号之前一次性标识输入文件。

- -a、--debug-syms：显示所有符号，即使是仅供调试器使用的符号；通常这些不会列出。

- -B：与 --format=bsd 相同（用于与 MIPS nm 的兼容性）。

- -C、--demangle[=style]：将低级别符号名称解码为用户级别的名称。除了移除系统添加的任何初始下划线外，这还使得 C++ 函数名称可读。不同的编译器具有不同的名称修饰风格。可选的解码风格参数可用于选择适合于当前使用编译器的解码风格。有关反编译的更多信息，请参阅 c++filt。
  
- --no-demangle：不要解码低级别符号名称。这是默认设置。

- --recurse-limit、--no-recurse-limit、--recursion-limit、--no-recursion-limit：启用或禁用在解码字符串时执行的递归量的限制。由于名称修饰格式允许无限级别的递归，因此可能会创建解码将耗尽主机机器上可用的堆栈空间的字符串，从而触发内存故障。该限制试图通过将递归限制为 2048 层嵌套来防止这种情况发生。
默认情况下，此限制处于启用状态，但在解码真正复杂的名称时可能需要禁用它。但请注意，如果禁用了递归限制，则可能会出现堆栈耗尽。

- -D、--dynamic：显示动态符号而不是普通符号。这仅对动态对象（例如某些类型的共享库）有意义。

- -f format、--format=format：使用输出格式 format，可以是 bsd、sysv、posix 或 just-symbols。默认为 bsd。只有 format 的第一个字符是有效的；它可以是大写或小写。

- -g、--extern-only：仅显示外部符号。

- -h、--help：显示 nm 的选项摘要并退出。

- --ifunc-chars=CHARS：在显示 GNU 间接函数符号时，nm 将默认使用 i 字符同时用于本地间接函数和全局间接函数。--ifunc-chars 选项允许用户指定一个包含一个或两个字符的字符串。第一个字符将用于全局间接函数符号，如果存在第二个字符，则将用于本地间接函数符号。

- j：与 --format=just-symbols 相同。

- -l、--line-numbers：对于每个符号，使用调试信息尝试查找文件名和行号。对于已定义的符号，查找符号地址的行号。对于未定义的符号，查找引用该符号的重定位条目的行号。如果可以找到行号信息，则在其他符号信息之后打印它。

- --inlines：当 -l 选项处于活动状态时，如果地址属于已内联的函数，则此选项会导致打印出所有封闭作用域的源信息，直到第一个非内联函数为止。例如，如果 main 内联了 callee1，而 callee1 又内联了 callee2，并且地址来自 callee2，则还将打印出 callee1 和 main 的源信息。

- -n、-v、--numeric-sort：按照符号的地址而不是名称按数字顺序排序。

- -p、--no-sort：不要对符号进行排序；按照遇到的顺序打印它们。

- -P、--portability：使用 POSIX.2 标准输出格式，而不是默认格式。相当于 -f posix。

- -r、--reverse-sort：反转排序顺序（无论是数字还是字母）；让最后一个先出现。

- -S、--print-size：对于 bsd 输出样式，打印已定义符号的值和大小。对于不记录符号大小的对象格式，此选项无效，除非还使用了 --size-sort，在这种情况下将显示计算出的大小。

- -s、--print-armap：在列出存档成员的符号时，包括索引：一个映射（由 ar 或 ranlib 存储在存档中）指示哪些模块包含哪些名称的定义。

- -t radix、--radix=radix：使用基数 radix 打印符号值。它必须是十进制的 d、八进制的 o 或十六进制的 x。

- -u、--undefined-only：仅显示未定义的符号（每个对象文件外部的符号）。默认情况下，已定义和未定义的符号都会显示。

- -U、--defined-only：仅为每个对象文件显示已定义的符号。默认情况下，已定义和未定义的符号都会显示。

- -V、--version：显示 nm 的版本号并退出。

- --plugin name：加载名为 name 的插件，以添加对额外目标类型的支持。仅当工具链启用了插件支持时，此选项才可用。
如果未提供 --plugin，但已启用插件支持，则 nm 会按字母顺序迭代 `${libdir}/bfd-plugins` 中的文件，并使用第一个适用于所讨论的对象的插件。注意，此插件搜索目录不是 ld 的 -plugin 选项使用的目录。为了使 nm 使用链接器插件，必须将其复制到`${libdir}/bfd-plugins` 目录中。对于基于 GCC 的编译，链接器插件称为 `liblto_plugin.so.0.0.0`。对于基于 Clang 的编译，它称为 `LLVMgold.so`。GCC 插件始终向后兼容早期版本，因此只需复制最新版本即可。

- --size-sort：按大小对符号进行排序。对于 ELF 对象，从 ELF 中读取符号大小；对于其他对象类型，符号大小计算为该符号的值与具有更高值的符号的值之间的差异。如果使用 bsd 输出格式，则打印符号的大小而不是值，必须使用 -S 选项以同时打印大小和值。 注意：如果启用了 --undefined-only，则此选项无效，因为未定义的符号没有大小。

- --special-syms：显示具有特定目标特殊含义的符号。这些符号通常由目标用于某些特殊处理，通常不会在正常符号列表中显示。例如，对于 ARM 目标，此选项将跳过用于标记 ARM 代码、THUMB 代码和数据之间转换的映射符号。

- --synthetic：在输出中包含合成符号。这些是链接器为各种目的创建的特殊符号。默认情况下不显示它们，因为它们不是二进制原始源代码的一部分。

- --unicode=[default|invalid|locale|escape|hex|highlight]：控制字符串中 UTF-8 编码的多字节字符的显示方式。
默认值（--unicode=default）是不对它们进行特殊处理。
--unicode=locale 选项以当前区域设置中的序列显示它们，该区域设置可能支持或不支持它们。
--unicode=hex 和 --unicode=invalid 选项将它们显示为由尖括号或花括号括起的十六进制字节序列。
--unicode=escape 选项将它们显示为转义序列（\uxxxx），而 --unicode=highlight 选项将它们显示为红色高亮的转义序列（如果输出设备支持）。着色旨在引起对可能不希望出现的 Unicode 序列的注意。

- -W、--no-weak：不显示弱符号。

- --with-symbol-versions、--without-symbol-versions：启用或禁用符号版本信息的显示。版本字符串作为后缀显示在符号名称之后，前面带有 @ 字符。例如，“foo@VER_1”。如果版本是解析未版本化引用到该符号时要使用的默认版本，则它将作为后缀显示，前面带有两个 @ 字符。例如，“foo@@VER_2”。默认情况下，显示符号版本信息。

- --target=bfdname：指定除系统默认格式之外的对象代码格式。

#### 参考连接：
【1】[https://sourceware.org/binutils/docs/binutils/nm.html](https://sourceware.org/binutils/docs/binutils/nm.html)