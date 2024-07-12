---
title: GNU linker ld —— Linker Script Format
date: 2024/6/21
categories: 
- GUN-linker
---

<center>
GNU linker ld (GNU Binutils) version 2.42 —— Linker Scripts：Linker Script Format
</center>

<!--more-->

***

**个人笔记，仅供参考！！**

链接脚本是文本文件，其内容为一系列命令。每个命令要么是一个关键字，后面可能跟着参数，要么是对一个符号的赋值。
可以使用分号来分隔命令，空白字符通常会被忽略。

链接脚本中可以直接使用文件或格式名称这样的字符串。如果文件名包含逗号字符，可以将文件名放在双引号中来处理这种情况（逗号字符本来会用来分隔文件名）。不允许在文件名中使用双引号字符。

在链接脚本中可以使用 C 风格的注释，注释由`/*`和`*/`界定。


#### 参考连接：
【1】[https://sourceware.org/binutils/docs/ld/Script-Format.html](https://sourceware.org/binutils/docs/ld/Script-Format.html)