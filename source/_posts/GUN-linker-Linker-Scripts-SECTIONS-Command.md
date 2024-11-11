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
- 输出段描述（参见[Output Section Description](https://fengxun2017.github.io/2024/07/12/GUN-linker-Linker-Scripts-Output-Section-Description/)）
- 覆盖描述(overlay description)

如果没有在链接器脚本中使用 **SECTIONS** 命令，链接器将按照输入文件中段出现的顺序，将每个输入段放入同名的输出段中。


#### 参考连接：
【1】[https://sourceware.org/binutils/docs/ld/SECTIONS.html](https://sourceware.org/binutils/docs/ld/SECTIONS.html)