---
title: GNU linker ld —— Special Sections
date: 2024/7/24
categories: 
- GUN-linker
---

<center>
GNU linker ld (GNU Binutils) version 2.42 —— Linker Scripts：Special Sections
</center>

<!--more-->

***

**个人笔记，仅供参考！！**
在链接ELF格式的目标文件时，ld 会以一种特殊的、非标准的方式处理某些 section。

**.gnu.warning**：
`.gnu.warning` section 的内容被假定为ASCII格式的警告消息。
如果在任何输入文件中出现这个部分，其内容将显示给用户，但不会复制到输出映像中。如果启用了--fatal-warnings选项，那么如果遇到警告，链接过程也会停止。
注意：`.gnu.warning` section 不受链接器垃圾回收或孤儿段处理的影响。

**.gnu.warning.SYM**：
以前缀`.gnu.warning.`开始并以符号名称结尾的section，以类似于`.gnu.warning` section 的方式处理。但只有在指定的符号被引用时，才会显示这个部分的内容。例如，如果存在一个名为`.gnu.warning.foo`的 section，只有当一个或多个输入文件引用了符号 foo 时，其内容才会作为警告消息显示。这包括从静态库中提取的目标文件、完成链接所需的共享对象等。

注意：因为这些警告消息是在链接器执行垃圾收集(如果启用)之前生成的，所以可能会显示一个符号的警告，该符号后来被删除，然后永远不会出现在最终输出中。

**.note.gnu.property：**
当链接器合并这个名称的各个section时，它会根据注释本身编码的各种规则将它们合并在一起。因此，输出的`.note.gnu.property` section 的内容可能不对应于输入部分的简单连接。如果使用了-Map选项来请求链接器映射，那么属性合并的详细信息将包含在映射中。


#### 参考连接：
【1】[https://sourceware.org/binutils/docs/ld/Special-Sections.html](https://sourceware.org/binutils/docs/ld/Special-Sections.html)