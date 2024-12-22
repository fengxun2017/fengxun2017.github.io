---
title: GNU binary utilities —— size
date: 2024/11/20
categories: 
- GNU-binary-utilities
---

<center>
GNU binary utilities (GNU Binutils) version 2.42: —— size
</center>

<!--more-->

***
**个人笔记，仅供参考！！**

size [-A|-B|-G|--format=compatibility] [--help] [-d|-o|-x|--radix=number] [--common] [-t|--totals] [--target=bfdname] [-V|--version] [-f] [objfile...]

GNU size 工具列出每个二进制文件的节大小和总大小。如果未指定文件，则默认使用 a.out。
命令行选项的含义如下：

#### -A -B -G --format=compatibility 
使用这些选项之一，可以选择输出格式：

-A 或 --format=sysv：使用 System V size 格式

-B 或 --format=berkeley：使用 Berkeley size 格式（默认）

-G 或 --format=gnu：使用 GNU 格式
```
$ size --format=Berkeley ranlib size
   text    data     bss     dec     hex filename
 294880   81920   11592  388392   5ed28 ranlib
 294880   81920   11888  388688   5ee50 size
```
Berkeley 输出格式将只读数据计入 text 列，而非 data 列；dec 和 hex 列分别以十进制和十六进制显示 text、data 和 bss 列的总和。

```
$ size --format=GNU ranlib size
      text       data        bss      total filename
    279880      96920      11592     388392 ranlib
    279880      96920      11888     388688 size
```
GNU 格式将只读数据计入 data 列，而非 text 列，仅在 total 列显示总和。
```
$ size --format=SysV ranlib size
ranlib  :
section         size         addr
.text         294880         8192
.data          81920       303104
.bss           11592       385024
Total         388392

size  :
section         size         addr
.text         294880         8192
.data          81920       303104
.bss           11888       385024
Total         388688
```

#### --help -h -H -? 
显示可接受参数和选项的摘要。

#### -d -o -x --radix=number 
控制每个节的大小显示格式：
- -d 或 --radix=10：十进制

- -o 或 --radix=8：八进制

- -x 或 --radix=16：十六进制

总大小始终以两种基数显示；对于 -d 或 -x，显示十进制和十六进制；对于 -o，显示八进制和十六进制。

#### --common 
显示每个文件中的公共符号总大小。使用 Berkeley 或 GNU 格式时，这些符号包含在 bss 大小中。

#### -t --totals 
显示所有列出的对象的总数（仅适用于 Berkeley 或 GNU 格式）。

#### --target=bfdname 
指定 objfile 的目标代码格式为 bfdname。通常不需要此选项，因为 size 可以自动识别许多格式。

#### -v -V --version 
显示 size 的版本号。

#### -f
被忽略。此选项由其他版本的 size 程序使用，但不受 GNU Binutils 版本支持。

#### 参考连接：
【1】[https://sourceware.org/binutils/docs-2.42/binutils/size.html](https://sourceware.org/binutils/docs-2.42/binutils/size.html)