---
title: GNU linker ld —— Assigning Values to Symbols
date: 2024/7/1
categories: 
- GUN-linker
---

<center>
GNU linker ld (GNU Binutils) version 2.42 —— Linker Scripts：Assigning Values to Symbols
</center>

<!--more-->

***

**个人笔记，仅供参考！！**

#### 1 Simple Assignments
在链接脚本中，可以使用以下赋值运算符，来为符号赋值：

```
symbol = expression;
 symbol += expression;
 symbol -= expression;
 symbol *= expression;
 symbol /= expression;
 symbol <<= expression;
 symbol >>= expression;
 symbol &= expression;
 symbol |= expression;
```
**注意：在表达式之后，分号是必需的**。

第一种情况下，将会将symbol定义为expression的值。其他情况下，symbol必须已经被定义，其值将根据表达式进行调整。

特殊的符号名称`.`表示位置计数器（location counter）。只能在SECTIONS命令中使用它。


可以将符号赋值作为单独的命令，或者作为SECTIONS命令中的语句，或者作为SECTIONS命令中输出段描述中的一部分。

以下是一个示例，展示了三种不同的符号赋值的用法：
```
 
floating_point = 0;
SECTIONS
{
  .text :
    {
      *(.text)
      _etext = .;
    }
  _bdata = (. + 3) & ~3;
  .data : { *(.data) }
}
```

在这个示例中，符号`floating_point`被定义为零。符号`_etext`被定义为最后一个输入段`.text`之后的地址。符号`_bdata`将被定义为`text`输出段之后的地址，向上对齐到4字节边界。

#### 2 HIDDEN
对于 ELF 输出目标，可以使用 `HIDDEN` 关键字来定义一个不会被导出的隐藏符号。
其语法为：HIDDEN(symbol = expression)

使用 HIDDEN 重写前面的例子，如下：
```
 
HIDDEN(floating_point = 0);
SECTIONS
{
  .text :
    {
      *(.text)
      HIDDEN(_etext = .);
    }
  HIDDEN(_bdata = (. + 3) & ~3);
  .data : { *(.data) }
}
```
在这个例子中，这三个符号都不会在此模块之外可见。


#### 3 PROVIDE

在某些情况下，我们希望仅在符号被引用且未被任何链接中包含的对象文件定义时才定义该符号。
例如，传统的链接器定义了符号`etext`。然而，ANSI C要求用户能够将`etext`用作函数名而不会遇到错误。PROVIDE关键字可用于仅在符号被引用但未定义时定义符号，例如`etext`。其语法如下：PROVIDE(symbol = expression)。

以下是使用PROVIDE定义“etext”的示例：
```
SECTIONS
{
  .text :
    {
      *(.text)
      _etext = .;
      PROVIDE(etext = .);
    }
}
```
在这个例中，如果程序中定义了`_etext`，链接器将会给出多重定义的报错。如果程序定义了`etext`，链接器将会在程序中默认使用程序中的定义。如果程序引用了“etext”但未定义它，链接器将使用链接脚本中的定义。


#### 4 PROVIDE_HIDDEN
类似于 `PROVIDE`。对于ELF输出目标，符号将被隐藏并且不会被导出。

#### 5 Source Code Reference

链接脚本中的符号并不等同于高级语言中的变量声明，而是一个没有值的符号。

首先，有一点需要注意：编译器通常会将源代码中的名称转换为存储在符号表中的不同名称。例如，Fortran 编译器通常会在变量名前后添加下划线，而 C++ 则会执行“名称修饰”。因此，在源代码中使用的变量名称与在链接脚本中定义的相同变量名称之间可能存在差异。

在下面的示例中，我们假设没有进行名称转换。

当在高级语言（如 C 语言）中声明符号时，会发生两件事情。首先，编译器会在程序的内存中保留足够的空间来存储该符号的值。其次，编译器会在程序的符号表中创建一个条目，该条目保存符号的地址。
例如，以下 C 声明：
```
int foo = 1000;
```
会在符号表中创建一个名为`foo`的条目。该条目保存了一个`int`大小的内存块的地址，初始值为 1000。

相比之下，链接脚本中的符号声明会在符号表中创建一个条目，但不会为它们分配任何内存。因此，它们只是一个没有值的地址。
例如，链接脚本中的定义：
```
foo = 1000;
```
会在符号表中创建一个名为`foo`的条目，该条目保存了内存位置`1000`的地址，但在地址`1000`处并没有存储任何特殊的内容。
**这意味着无法访问链接脚本定义的符号的值——它没有值。只能访问链接脚本定义的符号的地址。**

因此，在源代码中使用链接脚本定义的符号时，应该始终获取该符号的地址，而不要尝试使用其值。
例如，假设需要将名为 .ROM 的段的内容复制到名为 .FLASH 的段中，链接脚本包含以下声明：
```
 
start_of_ROM = .ROM;
end_of_ROM = .ROM + sizeof(.ROM);
start_of_FLASH = .FLASH;
```
那么，执行复制的 C 源代码将如下所示：
```
 
extern char start_of_ROM, end_of_ROM, start_of_FLASH;
memcpy(&start_of_FLASH, &start_of_ROM, &end_of_ROM - &start_of_ROM);
```

注意这里需要使用“&”运算符。或者，也可以将这些符号视为向量或数组的名称，然后按如下方式编写：
```
 
extern char start_of_ROM[], end_of_ROM[], start_of_FLASH[];
memcpy(start_of_FLASH, start_of_ROM, end_of_ROM - start_of_ROM);
```


#### 参考连接：
【1】[https://sourceware.org/binutils/docs/ld/Source-Code-Reference.html](https://sourceware.org/binutils/docs/ld/Source-Code-Reference.html)
【2】[https://sourceware.org/binutils/docs/ld/PROVIDE_005fHIDDEN.html](https://sourceware.org/binutils/docs/ld/PROVIDE_005fHIDDEN.html)
【3】[https://sourceware.org/binutils/docs/ld/PROVIDE.html](https://sourceware.org/binutils/docs/ld/PROVIDE.html)
【4】[https://sourceware.org/binutils/docs/ld/Simple-Assignments.html](https://sourceware.org/binutils/docs/ld/Simple-Assignments.html)