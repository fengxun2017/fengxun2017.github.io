---
title: GNU linker ld —— Simple Linker Script Commands
date: 2024/6/28
categories: 
- GUN-linker
---

<center>
GNU linker ld (GNU Binutils) version 2.42 —— Linker Scripts：Simple Linker Script Commands
</center>

<!--more-->

***

**个人笔记，仅供参考！！**

#### 1 Setting the Entry Point
```
ENTRY(symbol)
```
该命令用于设置程序入口点 程序中要执行的第一条指令称为入口点。

有几种方法可以设置入口点。链接器将按顺序尝试以下每种方法来设置入口点，直到其中一种成功：
- 命令行选项 `-e`；
- 链接器脚本中的 `ENTRY(synbol)` 命令；
- 如果定义了目标特定的符号的值（对于许多目标来说，start符号的值为程序入口）。
- 如果正在创建可执行文件，则代码段的第一个字节的地址即为程序入口。
- 地址 0。

#### 2 Commands Dealing with Files
如下是一些链接脚本中处理文件的命令：

##### 2.1 INCLUDE filename
在当前位置包含链接脚本文件名。文件将在当前目录和使用 -L 选项指定的任何目录中进行搜索。可以嵌套调用 INCLUDE，最深可达 10 层。

可以在顶层、MEMORY 或 SECTIONS 命令中，或在输出段描述中放置 INCLUDE 指令。

##### 2.2 INPUT(file, file, …) 或 INPUT(file file …) 
INPUT 命令指示链接器在链接中包含所指定文件。

例如，如果总是需要在链接时包含 subr.o，但不想每次都在链接命令行上输入它，就可以在链接脚本中放置 INPUT (subr.o)。

如果配置了 sysroot 前缀，并且文件名以 / 字符开头，并且正在处理的脚本位于 sysroot 前缀内，那么将在 sysroot 前缀中查找文件名。通过在文件名路径中指定 = 作为第一个字符，或在文件名路径前加上 $SYSROOT，也可以强制使用 sysroot 前缀。另见命令行选项中的 -L 描述。

如果没有使用 sysroot 前缀，那么链接器将尝试在包含链接脚本的目录中打开文件。如果找不到，链接器将搜索当前目录。如果仍然找不到，链接器将搜索库搜索路径。

如果 INPUT (-lfile)，ld 将名称转换为 libfile.a，效果与命令行参数 -l 一样。

需要注意的是，在链接脚本中使用 INPUT 命令时，文件将在包含链接脚本文件的点被包含在链接中。这可能会影响库搜索。

##### 2.3 GROUP(file, file, …) 或 GROUP(file file …) 
GROUP 命令类似于 INPUT，不同之处在于输入的文件都应该是库文件，并且它们可以被重复搜索另见命令行选项中的`-(`描述。

##### 2.4 AS_NEEDED(file, file, …) 或 AS_NEEDED(file file …) 
这个结构只能出现在 INPUT 或 GROUP 命令中。
这个结构本质上为其中列出的所有文件启用了 --as-needed 选项，并在之后恢复之前的 --as-needed 或 --no-as-needed 设置。
例如：
```
GROUP(
  libmain.a
  AS_NEEDED(liboptional.so libanother.so)
  libsupport.a
)

```

##### 2.5 OUTPUT(filename)
OUTPUT 可以设置输出文件。在链接脚本中使用 OUTPUT(filename) 与在命令行上使用 -o filename 效果一样。如果两者都使用了，命令行选项优先。

可以使用 OUTPUT 命令为输出文件定义一个默认名称。

##### 2.6 SEARCH_DIR(path) 
SEARCH_DIR 命令用于将路径添加到 ld 查找库的路径列表中。
使用 SEARCH_DIR(path) 与在命令行上使用 -L path 效果一样（参见命令行选项）。如果两者都使用了，那么链接器将搜索两个路径。首先搜索使用命令行选项指定的路径。

##### 2.7 STARTUP(filename)
STARTUP 命令指定的 filename 将成为要链接的第一个输入文件。如果当前系统的入口点总是第一个文件的开始，该命令可以用来指定入口。



#### 3 Commands Dealing with Object File Formats
如下是一些用于处理对象文件格式的命令。

##### 3.1 OUTPUT_FORMAT(bfdname) 或 OUTPUT_FORMAT(default, big, little) 
OUTPUT_FORMAT 命令指定输出文件使用的 BFD 格式。作用与在命令行上使用 --oformat bfdname 一样。如果两者都使用了，命令行选项优先。

如果使用带有三个参数的 OUTPUT_FORMAT，会基于命令行选项 -EB 和 -EL 而使用不同的格式。这允许链接脚本根据所需的字节序设置输出格式。
具体地：如果链接命令行中，既没有使用 -EB 选项，也没有使用 -EL 选项，那么输出格式将是第一个参数，default。如果使用了 -EB，输出格式将是第二个参数 big。如果使用了 -EL，输出格式将是第三个参数 little。

例如，MIPS ELF 目标的默认链接脚本使用此命令：
```
OUTPUT_FORMAT(elf32-bigmips, elf32-bigmips, elf32-littlemips)
```
这表明输出文件的默认格式是 elf32-bigmips，但如果用户使用了 -EL 命令行选项，输出文件将以 elf32-littlemips 格式创建。

##### 3.2 TARGET(bfdname) 
TARGET 命令指定读取输入文件时使用的 BFD 格式。它影响随后的 INPUT 和 GROUP 命令。其作用与在命令行上使用 -b bfdname 一样。如果使用了 TARGET 命令但没有使用 OUTPUT_FORMAT，那么最后一个 TARGET 命令也将用于设置输出文件的格式。

#### 4 Assign alias names to memory regions
链接脚本中的 **REGION_ALIAS** 用于为内存区域创建别名，使输出节能够灵活地映射到内存区域。示例场景：

假设我们有一个嵌入式系统的应用程序，该程序包含四个输出节：
- .text：程序代码段
- .rodata：只读数据段
- .data：可读写的，已初始化数据段
- .bss：零初始化数据

该系统中存在一个通用的、易失性的 RAM 内存，用于代码执行或数据存储。还有一个只读的、非易失性的 ROM 内存，用于代码执行和只读数据访问。以及一种变体是只读的、非易失性的 ROM2 内存，用于只读数据访问，但不支持代码执行。
要求：程序代码加载到 ROM 中，只读数据加载到 ROM2 中，读写数据加载到 RAM 中。

链接器脚本示例：
```
MEMORY
  {
    ROM : ORIGIN = 0, LENGTH = 2M
    ROM2 : ORIGIN = 0x10000000, LENGTH = 1M
    RAM : ORIGIN = 0x20000000, LENGTH = 1M
  }

REGION_ALIAS("REGION_TEXT", ROM);
REGION_ALIAS("REGION_RODATA", ROM2);
REGION_ALIAS("REGION_DATA", RAM);
REGION_ALIAS("REGION_BSS", RAM);

SECTIONS
  {
    .text :
      {
        *(.text)
      } > REGION_TEXT
    .rodata :
      {
        *(.rodata)
        rodata_end = .;
      } > REGION_RODATA
    .data : AT (rodata_end)
      {
        data_start = .;
        *(.data)
      } > REGION_DATA
    data_size = SIZEOF(.data);
    data_load_start = LOADADDR(.data);
    .bss :
      {
        *(.bss)
      } > REGION_BSS
  }
```

最后，如果需要，可以编写一个通用的系统初始化例程，将 .data 节从 ROM 或 ROM2 复制到 RAM 中。
```
#include <string.h>

extern char data_start [];
extern char data_size [];
extern char data_load_start [];

void copy_data(void)
{
  if (data_start != data_load_start)
    {
      memcpy(data_start, data_load_start, (size_t) data_size);
    }
}

```

#### 5 Other Linker Script Commands

##### 5.1 ASSERT(exp, message)：
**ASSERT** 命令用于确保表达式 exp 的值非零。如果 exp 为零，则链接器会退出并打印错误消息。
需要注意的是，链接器在最终的链接阶段之前会检查断言。因此在链接的早期阶段，链接脚本中的断言会被验证。但另一方面，在段定义内部提供的符号，它们的值在链接的最终阶段之前不会被设置，因此在断言中使用这些符号可能会失败。唯一的例外是仅引用点号提供的符号。

例如：下面的例子使用了节内部定义的__stack_size，因此会失败。（__stack使用“.”没有问题）
```
.stack :
{
  PROVIDE (__stack = .);
  PROVIDE (__stack_size = 0x100);
  ASSERT ((__stack > (_end + __stack_size)), "Error: No room left for the stack");
}
```

在段定义之外提供的符号会更早地进行求值，因此它们可以在断言中使用。例如：
```
PROVIDE (__stack_size = 0x100);
.stack :
{
  PROVIDE (__stack = .);
  ASSERT ((__stack > (_end + __stack_size)), "Error: No room left for the stack");
}
```

##### 5.2 EXTERN(symbol symbol …)：

**EXTERN** 命令用于将符号作为未定义符号输入到输出文件中。这可能会触发从标准库链接其他模块的操作。

可以为每个 **EXTERN** 列出多个符号，并且可以多次使用 **EXTERN**。此命令的效果与命令行选项 -u 相同。

##### 5.3 FORCE_COMMON_ALLOCATION：
此命令的效果与命令行选项 -d 相同：即使指定了可重定位输出文件（-r），也会使链接器为 common 符号分配空间。

#### 5.4 INHIBIT_COMMON_ALLOCATION：
此命令的效果与命令行选项 --no-define-common 相同：即使对于非重定位输出文件，也会使链接器忽略对 common 符号的地址分配。

##### 5.5FORCE_GROUP_ALLOCATION：
此命令的效果与命令行选项 --force-group-allocation 相同：使链接器将段组成员放置在普通输入节中，并且即使指定了可重定位输出文件（-r），也会删除段组。

##### 5.6 INSERT [ AFTER | BEFORE ] output_section：
**INSERT** 命令允许在指定的输出段之后或之前插入其他链接脚本语句。

##### 5.7 NOCROSSREFS(section section …)：
**NOCROSSREFS** 命令接受一个**输出节名称的列表**。如果链接器检测到这些节之间存在交叉引用，它将报告错误并返回非零的退出状态。NOCROSSREFS 命令在确保两个或多个输出节完全独立的情况下非常有用。

例如，在使用覆盖层（overlays）的嵌入式系统中，当一个节被加载到内存中时，另一个节不会被加载，如果一个节中的代码调用了另一个节中定义的函数，那么这将是一个错误。

##### 5.8 NOCROSSREFS_TO(tosection fromsection …)：
NOCROSSREFS_TO 命令接受一个**输出节名称的列表**。第一个节不能从其他节中引用。如果链接器检测到其他节中对第一个节的引用，它将报告错误并返回非零的退出状态。


##### 5.9 OUTPUT_ARCH(bfdarch)：
**OUTPUT_ARCH** 命令用于指定特定的输出机器体系结构。参数是 BFD 库使用的名称之一。你可以使用 objdump 程序的 -f 选项查看目标文件的体系结构。

##### 5.10 LD_FEATURE(string)：
**LD_FEATURE** 命令用于修改链接器的行为。
如果 string 是 “SANE_EXPR”，则链接脚本中的绝对符号和数字将在所有地方都被视为数字。请参阅[GUN-linker-Linker-Scripts-The-Section-of-an-Expression](https://fengxun2017.github.io/2024/06/30/GUN-linker-Linker-Scripts-The-Section-of-an-Expression/)。



#### 参考连接：
【1】[https://sourceware.org/binutils/docs/ld/Entry-Point.html](https://sourceware.org/binutils/docs/ld/Entry-Point.html)
【2】[https://sourceware.org/binutils/docs/ld/Format-Commands.html](https://sourceware.org/binutils/docs/ld/Format-Commands.html)
【3】[https://sourceware.org/binutils/docs/ld/File-Commands.html](https://sourceware.org/binutils/docs/ld/File-Commands.html)
【3】[https://sourceware.org/binutils/docs/ld/Miscellaneous-Commands.html](https://sourceware.org/binutils/docs/ld/Miscellaneous-Commands.html)