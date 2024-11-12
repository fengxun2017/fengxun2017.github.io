---
title: GNU binary utilities —— objdump
date: 2024/8/19
categories: 
- GNU-binary-utilities
---

<center>
GNU binary utilities (GNU Binutils) version 2.42: —— objdump
</center>

<!--more-->

***
**个人笔记，仅供参考！！**
```
objdump [-a|--archive-headers]
        [-b bfdname|--target=bfdname]
        [-C|--demangle[=style] ]
        [-d|--disassemble[=symbol]]
        [-D|--disassemble-all]
        [-z|--disassemble-zeroes]
        [-EB|-EL|--endian={big | little }]
        [-f|--file-headers]
        [-F|--file-offsets]
        [--file-start-context]
        [-g|--debugging]
        [-e|--debugging-tags]
        [-h|--section-headers|--headers]
        [-i|--info]
        [-j section|--section=section]
        [-l|--line-numbers]
        [-S|--source]
        [--source-comment[=text]]
        [-m machine|--architecture=machine]
        [-M options|--disassembler-options=options]
        [-p|--private-headers]
        [-P options|--private=options]
        [-r|--reloc]
        [-R|--dynamic-reloc]
        [-s|--full-contents]
        [-Z|--decompress]
        [-W[lLiaprmfFsoORtUuTgAck]|
         --dwarf[=rawline,=decodedline,=info,=abbrev,=pubnames,=aranges,=macro,=frames,=frames-interp,=str,=str-offsets,=loc,=Ranges,=pubtypes,=trace_info,=trace_abbrev,=trace_aranges,=gdb_index,=addr,=cu_index,=links]]
        [-WK|--dwarf=follow-links]
        [-WN|--dwarf=no-follow-links]
        [-wD|--dwarf=use-debuginfod]
        [-wE|--dwarf=do-not-use-debuginfod]
        [-L|--process-links]
        [--ctf=section]
        [--sframe=section]
        [-G|--stabs]
        [-t|--syms]
        [-T|--dynamic-syms]
        [-x|--all-headers]
        [-w|--wide]
        [--start-address=address]
        [--stop-address=address]
        [--no-addresses]
        [--prefix-addresses]
        [--[no-]show-raw-insn]
        [--adjust-vma=offset]
        [--show-all-symbols]
        [--dwarf-depth=n]
        [--dwarf-start=n]
        [--ctf-parent=section]
        [--no-recurse-limit|--recurse-limit]
        [--special-syms]
        [--prefix=prefix]
        [--prefix-strip=level]
        [--insn-width=width]
        [--visualize-jumps[=color|=extended-color|=off]
        [--disassembler-color=[off|terminal|on|extended]
        [-U method] [--unicode=method]
        [-V|--version]
        [-H|--help]
        objfile…
```

objdump 显示一个或多个目标文件的信息。选项控制显示哪些特定信息。

`objfile...` 是要检查的目标文件。当指定归档文件（archives）时，objdump 会显示每个成员目标文件的信息。

这里显示的选项的长短形式是等效的。必须从列表 -a, -d, -D, -e, -f, -g, -G, -h, -H, -p, -P, -r, -R, -s, -S, -t, -T, -V, -x 中至少选择一个选项。

#### -a, --archive-header：
如果任何 objfile 文件是归档文件（archive），显示归档文件头信息（格式类似于 ls -l）。除了可以用 `ar tv` 列出的信息外，objdump -a 还显示每个归档文件中成员的目标文件格式。


#### –adjust-vma=offset：
在转储信息时，首先将 offset 添加到所有的section地址。这在section地址与符号表不对应时很有用，例如在使用不能表示section地址的格式（如 a.out）时将section放置在特定地址。

#### -b bfdname, --target=bfdname：
指定目标文件的目标代码格式为 bfdname。这个选项可能不是必须的；objdump 可以自动识别许多格式。例如：
`objdump -b oasys -m vax -h fu.o`
显示 fu.o 的section头信息（-h），该文件被明确标识（-m）为由 Oasys 编译器生成的 VAX 目标文件。可以使用 -i 选项列出可用的格式。

#### -C, --demangle[=style]：
将低级符号名解码（解混淆）为用户级别的名称。除了移除系统预置的初始下划线外，这还使 C++ 函数名可读。不同的编译器有不同的混淆风格。可选的解混淆风格参数可用于为你的编译器选择合适的解混淆风格。更多信息请参见 c++filt。

#### –recurse-limit, --no-recurse-limit, --recursion-limit, --no-recursion-limit：
启用或禁用在解混淆字符串时执行的递归量限制。由于名称混淆格式允许无限级别的递归，可能会创建解码时耗尽主机机器上可用堆栈空间的字符串，从而触发内存故障。限制通过将递归限制为 2048 级嵌套来防止这种情况发生。默认情况下启用此限制，但在解混淆非常复杂的名称时可能需要禁用它。然而，请注意，如果禁用递归限制，则可能会耗尽堆栈空间。

#### -g, --debugging：
显示调试信息。这会尝试解析存储在文件中的 STABS 调试格式信息，并使用类似 C 的语法打印出来。如果没有找到 STABS 调试信息，此选项会回退到 -W 选项以打印文件中的任何 DWARF 信息。

#### -e, --debugging-tags：
类似于 -g，但信息以与 ctags 工具兼容的格式生成。

#### -d, --disassemble, --disassemble=symbol：
显示输入文件中机器指令的汇编助记符。此选项仅反汇编预期包含指令的section。如果给定了可选的 symbol 参数，则从 symbol 开始显示汇编助记符。如果 symbol 是函数名，则反汇编将在函数结束时停止，否则将在遇到下一个符号时停止。如果没有匹配的 symbol，则不会显示任何内容。请注意，如果启用了 --dwarf=follow-links 选项，则在反汇编时将读取并使用链接的调试信息文件中的任何符号表。


#### -D / --disassemble-all：
类似于 -d，但反汇编所有非空的非 bss 段的内容，而不仅仅是那些预期包含指令的段。可以使用 -j 选择特定的段。
该选项还对代码段中指令的反汇编有微妙的影响。当 -d 选项生效时，objdump 会假设代码段中的任何符号都出现在指令之间的边界上，并且会拒绝跨越这样的边界进行反汇编。然而，当 -D 选项生效时，这种假设会被抑制。这意味着如果在代码段中存储了数据，-d 和 -D 的输出可能会有所不同。
如果目标是 ARM 架构，该选项还会强制反汇编器将代码段中的数据片段解码为指令。
注意，如果启用了 --dwarf=follow-links 选项，则在反汇编时会读取并使用链接的调试信息文件中的符号表。

#### --no-addresses：
反汇编时，不在每行或符号和重定位偏移量上打印地址。与 --no-show-raw-insn 结合使用时，这对于比较编译器输出可能很有用。

#### --prefix-addresses
反汇编时，在每行打印完整地址。这是较旧的反汇编格式。

#### -EB、-EL、--endian=big/little
指定目标文件的字节序。这只影响反汇编。当反汇编不描述字节序信息的文件格式（如 S-records）时，这可能很有用。

#### -f/--file-headers
显示每个目标文件的整体头部摘要信息。

#### -F/--file-offsets
反汇编段时，每当显示符号时，也显示即将转储的数据区域的文件偏移量。如果跳过了零，则在反汇编恢复时，告诉用户跳过了多少个零以及从哪里恢复反汇编。当转储段时，显示转储开始位置的文件偏移量。

#### --file-start-context
指定在显示未显示过的文件的交错源代码/反汇编（假设 -S）时，将上下文扩展到文件的开头。

#### -h/--section-headers/--headers
显示目标文件的段头摘要信息。
文件段可能会被重定位到非标准地址，例如使用 -Ttext、-Tdata 或 -Tbss 选项。然而，一些目标文件格式（如 a.out）不存储文件段的起始地址。在这种情况下，尽管 ld 正确地重定位了段，但使用 objdump -h 列出文件段头时无法显示正确的地址。相反，它显示的是目标隐含的通常地址。
注意，在某些情况下，一个段可能同时具有 READONLY 和 NOREAD 属性。在这种情况下，NOREAD 属性优先，但 objdump 会报告两者，因为标志位的确切设置可能很重要。

#### -H / --help
打印 objdump 选项的摘要并退出。

#### -i / --info
显示所有可用于 -b 或 -m 指定的架构和目标格式的列表。

#### -j name / --section=name
显示指定段的信息。此选项可以多次指定。

#### -L / --process-links
显示链接到主文件的单独调试信息文件中非调试段的内容。此选项自动隐含 -WK 选项，并且只有其它命令行选项请求的段会被显示。

#### -l / --line-numbers 
使用与所显示的目标代码或位置对应的文件名和源行号标记显示(使用调试信息)。仅用于-d、-d或-r。

#### -m machine / --architecture=machine
指定反汇编目标文件时使用的架构。当反汇编不包含架构信息的目标文件（如 S-records）时，这个选项非常有用。你可以使用 -i 选项列出可用的架构。
对于大多数架构，可以提供一个架构名称和一个机器名称，用冒号分隔。例如，‘foo:bar’ 表示 ‘foo’ 架构中的 ‘bar’ 机器类型。如果 objdump 被配置为支持多种架构，这将非常有用。
如果目标是 ARM 架构，那么这个选项还有一个额外的效果。它将反汇编限制为仅支持由 machine 指定的架构的指令。如果因为输入文件不包含任何架构信息而需要使用此选项，但又希望反汇编所有指令，可以使用 -marm。

-M options
–disassembler-options=options 将特定于目标的信息传递给反汇编器。仅在某些目标上支持。如果需要指定多个反汇编选项，可以使用多个 -M 选项，或者将它们放在一个逗号分隔的列表中。

对于 ARC（Argonaut RISC Core，一种由 Synopsys 开发的可配置处理器架构）：
dsp: 控制 DSP 指令的打印，
spfp: 选择打印 FPX 单精度 FP 指令，
dpfp: 选择打印 FPX 双精度 FP 指令，
quarkse_em: 选择打印特殊的 QuarkSE-EM 指令，
fpuda: 选择打印双精度辅助指令，
fpus: 选择打印 FPU 单精度 FP 指令，
fpud: 选择打印 FPU 双精度 FP 指令。
此外，可以选择使用 hex 将所有立即数打印为十六进制。默认情况下，短立即数使用十进制表示，而长立即数使用十六进制表示。

cpu=... 允许在反汇编指令时强制使用特定的 ISA（Instruction Set Architecture），覆盖 -m 值或 ELF 文件中的任何内容。这可能对选择 ARC EM（Embedded Microprocessor） 或 HS（ High Speed） ISA 有用，因为它们的架构相同，反汇编器依赖于私有 ELF 头数据来决定代码是针对 EM 还是 HS。此选项可以多次指定——仅使用最新的值。有效值与汇编器的 -mcpu=... 选项相同。

如果目标是 ARM 架构，则此开关可用于选择在反汇编期间使用的寄存器名称集。指定 -M reg-names-std（默认）将选择 ARM 指令集文档中使用的寄存器名称，但寄存器 13 称为 sp，寄存器 14 称为 lr，寄存器 15 称为 pc。指定 -M reg-names-apcs 将选择 ARM 过程调用标准使用的名称集，而指定 -M reg-names-raw 将仅使用 r 后跟寄存器编号。

还有两个 APCS 寄存器命名方案的变体，分别由 -M reg-names-atpcs 和 -M reg-names-special-atpcs 启用，使用 ARM/Thumb 过程调用标准命名约定。（使用正常寄存器名称或特殊寄存器名称）。

此选项还可用于 ARM 架构，通过使用开关 --disassembler-options=force-thumb 强制反汇编器将所有指令解释为 Thumb 指令。这在尝试反汇编其他编译器生成的 Thumb 代码时可能很有用。

对于 AArch64 目标，此开关可用于设置是否使用 -M no-aliases 选项将指令反汇编为最通用的指令，或者是否使用 -M notes 在反汇编中生成指令注释。

对于 x86，一些选项重复了 -m 开关的功能，但允许更细粒度的控制。
x86-64/i386/i8086: 选择给定架构的反汇编。
intel/att: 在 Intel 语法模式和 AT&T 语法模式之间选择。
amd64/intel64: 在 AMD64 ISA 和 Intel64 ISA 之间选择。
intel-mnemonic/att-mnemonic: 在 Intel 助记符模式和 AT&T 助记符模式之间选择。注意：intel-mnemonic 隐含 intel，att-mnemonic 隐含 att。
addr64/addr32/addr16/data32/data16: 指定默认地址大小和操作数大小。如果 x86-64、i386 或 i8086 出现在选项字符串中，这五个选项将被覆盖。
suffix: 在 AT&T 模式下以及在 Intel 模式下的有限指令集中，指示反汇编器打印助记符后缀，即使可以通过操作数或某些指令的执行模式的默认值推断出后缀。

对于 PowerPC，-M 参数 raw 选择硬件指令的反汇编，而不是别名。例如，你会看到 rlwinm 而不是 clrlwi，addi 而不是 li。所有选择 CPU 的 gas 的 -m 参数都受支持。这些参数包括：403、405、440、464、476、601、603、604、620、7400、7410、7450、7455、750cl、821、850、860、a2、booke、booke32、cell、com、e200z2、e200z4、e300、e500、e500mc、e500mc64、e500x2、e5500、e6500、efs、power4、power5、power6、power7、power8、power9、power10、power11、ppc、ppc32、ppc64、ppc64bridge、ppcps、pwr、pwr2、pwr4、pwr5、pwr5x、pwr6、pwr7、pwr8、pwr9、pwr10、pwr11、pwrx、titan、vle 和 future。32 和 64 修改默认或先前的 CPU 选择，分别禁用和启用 64 位指令。此外，altivec、any、lsp、htm、vsx、spe 和 spe2 为先前或后来的 CPU 选择添加功能。any 将反汇编 binutils 知道的任何操作码，但在操作码有两种不同含义或不同参数的情况下，你可能不会看到预期的反汇编结果。如果在没有给出 CPU 选择的情况下进行反汇编，将从 BFD 从目标文件头中获取的信息中选择一个默认值，但结果可能仍然不如预期。

对于 MIPS，此选项控制反汇编指令中的指令助记符名称和寄存器名称的打印。可以将以下多个选择指定为逗号分隔的字符串，无效选项将被忽略：
- no-aliases：打印“原始”指令助记符，而不是某些伪指令助记符。例如，打印 daddu 或 or 而不是 move，打印 sll 而不是 nop。
- msa：反汇编 MSA 指令。
- virt：反汇编虚拟化 ASE 指令。
- xpa：反汇编扩展物理地址（XPA）ASE 指令。
- gpr-names=ABI：根据指定的 ABI 打印 GPR（通用寄存器）名称。默认情况下，GPR 名称根据正在反汇编的二进制文件的 ABI 选择。
- fpr-names=ABI：根据指定的 ABI 打印 FPR（浮点寄存器）名称。默认情况下，打印 FPR 编号而不是名称。
- cp0-names=ARCH：根据 ARCH 指定的 CPU 或架构打印 CP0（系统控制协处理器；协处理器 0）寄存器名称。默认情况下，CP0 寄存器名称根据正在反汇编的二进制文件的架构和 CPU 选择。
- hwr-names=ARCH：根据 ARCH 指定的 CPU 或架构打印 HWR（硬件寄存器，用于 rdhwr 指令）名称。默认情况下，HWR 名称根据正在反汇编的二进制文件的架构和 CPU 选择。
- reg-names=ABI：根据选定的 ABI 打印 GPR 和 FPR 名称。
- reg-names=ARCH：根据选定的 CPU 或架构打印特定于 CPU 的寄存器名称（CP0 寄存器和 HWR 名称）。
对于上述任何选项，可以将 ABI 或 ARCH 指定为“numeric”，以便为选定类型的寄存器打印数字而不是名称。你可以使用 --help 选项列出 ABI 和 ARCH 的可用值。

对于 VAX，你可以使用 -M entry:0xf00ba 指定函数入口地址。你可以多次使用此选项，以正确反汇编不包含符号表的 VAX 二进制文件（如 ROM 转储）。在这些情况下，函数入口掩码将被解码为 VAX 指令，这可能会导致函数的其余部分被错误反汇编。


#### -p / --private-headers 
打印特定于目标文件格式的信息。打印的确切信息取决于目标文件格式。对于某些目标文件格式，不会打印任何附加信息。

#### -P options / --private=options
打印特定于目标文件格式的信息。参数 options 是一个逗号分隔的列表，具体取决于格式（选项列表可以通过帮助命令显示）。

对于 XCOFF，可用选项包括：
        header
        aout
        sections
        syms
        relocs
        lineno
        loader
        except
        typchk
        traceback
        toc
        ldinfo
对于 PE，可用选项包括：
header
sections
并非所有目标格式都支持此选项，特别是 ELF 格式不使用它。

#### -r / --reloc:
打印文件的重定位条目。如果与 -d 或 -D 一起使用，重定位条目将与反汇编内容交错打印。

#### -R --dynamic-reloc:
打印文件的动态重定位条目。这仅对动态对象（如某些类型的共享库）有意义。与 -r 一样，如果与 -d 或 -D 一起使用，重定位条目将与反汇编内容交错打印。

#### -s / --full-contents:
显示节的全部内容，通常与 -j 一起使用以请求特定节。默认情况下，显示所有非空的非 bss 节。默认情况下，任何压缩节将以压缩形式显示。要查看解压缩形式的内容，请在命令行中添加 -Z 选项。

#### -S / --source
如果可能，显示与反汇编内容交错的源代码。隐含 -d 选项。

#### --show-all-symbols
反汇编时，显示与给定地址匹配的所有符号，而不仅仅是第一个符号。

#### --source-comment[=txt] 
类似于 -S 选项，但所有源代码行都以 txt 为前缀显示。通常，txt 是一个注释字符串，用于区分汇编代码和源代码。如果未提供 txt，则使用默认字符串 “# ”（井号后跟一个空格）。

#### --prefix=prefix 
指定在使用 -S 时添加到绝对路径的前缀。

#### --prefix-strip=level 
指示从硬编码的绝对路径中剥离多少个初始目录名称。如果没有 --prefix=prefix，则此选项无效。

#### --show-raw-insn 
反汇编指令时，以十六进制和符号形式打印指令。这是默认设置，除非使用了 --prefix-addresses。

##### --no-show-raw-insn 
反汇编指令时，不打印指令字节。这是使用 --prefix-addresses 时的默认设置。

#### --insn-width=width 
反汇编指令时，在一行上显示 width 字节。

#### --visualize-jumps[=color|=extended-color|=off] 
通过在起始地址和目标地址之间绘制 ASCII 艺术来可视化函数内的跳转。可选的 =color 参数使用简单的终端颜色为输出添加颜色。或者，=extended-color 参数将使用 8 位颜色添加颜色，但这些颜色可能不适用于所有终端。

如果需要在启用 visualize-jumps 选项后禁用它，请使用 visualize-jumps=off。

#### --disassembler-color=off / --disassembler-color=terminal / --disassembler-color=on|color|colour / --disassembler-color=extended|extended-color|extended-colour 
启用或禁用反汇编输出中的彩色语法高亮显示。默认行为由配置时的选项确定。请注意，并非所有架构都支持彩色语法高亮显示，并且根据使用的终端，彩色输出可能实际上不可读。
on 参数使用简单的终端颜色添加颜色。
terminal 参数执行相同操作，但仅当输出设备是终端时。
extended-color 参数类似于 on 参数，但使用 8 位颜色。这些颜色可能不适用于所有终端。
off 参数禁用彩色反汇编。

#### -W[lLiaprmfFsoORtUuTgAckK] / --dwarf[=rawline,=decodedline,=info,=abbrev,=pubnames,=aranges,=macro,=frames,=frames-interp,=str,=str-offsets,=loc,=Ranges,=pubtypes,=trace_info,=trace_abbrev,=trace_aranges,=gdb_index,=addr,=cu_index,=links,=follow-links] 
显示文件中 DWARF 调试节的内容（如果存在）。压缩的调试节在显示之前会自动解压缩（临时）。如果开关后面跟有一个或多个可选字母或单词，则仅转储这些类型的数据。字母和单词对应以下信息：

- a 或 =abbrev：显示 .debug_abbrev 节的内容。
- A 或 =addr：显示 .debug_addr 节的内容。
- c 或 =cu_index：显示 .debug_cu_index 和/或 .debug_tu_index 节的内容。
- f 或 =frames：显示 .debug_frame 节的原始内容。
- F 或 =frames-interp：显示 .debug_frame 节的解释内容。
- g 或 =gdb_index：显示 .gdb_index 和/或 .debug_names 节的内容。
- i 或 =info：显示 .debug_info 节的内容。注意：此选项的输出也可以通过使用 --dwarf-depth 和 --dwarf-start 选项来限制。
- k 或 =links：显示 .gnu_debuglink、.gnu_debugaltlink 和 .debug_sup 节的内容（如果存在）。还显示任何链接到单独的 DWARF 对象文件（dwo）的链接，如果它们由 .debug_info 节中的 DW_AT_GNU_dwo_name 或 DW_AT_dwo_name 属性指定。
- K 或 =follow-links 显示在链接的单独调试信息文件中找到的任何选定调试节的内容。如果同一调试节存在于多个文件中，这可能会导致显示多个版本的相同调试节。
此外，当显示 DWARF 属性时，如果发现引用单独调试信息文件的形式，则也会显示引用的内容。
注意：在某些发行版中，此选项默认启用。可以通过 N 调试选项禁用它。可以在配置 binutils 时通过 --enable-follow-debug-links=yes 或 --enable-follow-debug-links=no 选项选择默认值。如果不使用这些选项，则默认启用调试链接的跟随。
注意：如果在构建 binutils 时启用了 debuginfod 协议支持，则此选项还将尝试联系 DEBUGINFOD_URLS 环境变量中提到的任何 debuginfod 服务器。这可能需要一些时间来解析。可以通过 =do-not-use-debuginfod 调试选项禁用此行为。

- N 或 =no-follow-links 禁用跟随单独调试信息文件的链接。

- D 或 =use-debuginfod 如果需要跟随调试链接，则启用联系 debuginfod 服务器。这是默认行为。

- E 或 =do-not-use-debuginfod 在需要跟随调试链接时禁用联系 debuginfod 服务器。

- l 或 =rawline 以原始格式显示 .debug_line 节的内容。

- L 或 =decodedline 显示 .debug_line 节的解释内容。

- m 或 =macro 显示 .debug_macro 和/或 .debug_macinfo 节的内容。

- o 或 =loc 显示 .debug_loc 和/或 .debug_loclists 节的内容。

- O 或 =str-offsets 显示 .debug_str_offsets 节的内容。

- p 或 =pubnames 显示 .debug_pubnames 和/或 .debug_gnu_pubnames 节的内容。

- r 或 =aranges 显示 .debug_aranges 节的内容。

- R 或 =Ranges 显示 .debug_ranges 和/或 .debug_rnglists 节的内容。

- s 或 =str 显示 .debug_str、.debug_line_str 和/或 .debug_str_offsets 节的内容。

- t 或 =pubtype 显示 .debug_pubtypes 和/或 .debug_gnu_pubtypes 节的内容。

- T 或 =trace_aranges 显示 .trace_aranges 节的内容。

- u 或 =trace_abbrev 显示 .trace_abbrev 节的内容。

- U 或 =trace_info 显示 .trace_info 节的内容。


#### --dwarf-depth=n
限制.debug_info section的转储深度为n层。这只在使用--debug-dump=info 选项时有用。默认情况下会打印所有的DIEs；n的特殊值0也会有同样的效果。当n为非零值时，n层或更深层的DIEs将不会被打印。n的范围是从0开始。

#### --dwarf-start=n
仅打印从编号为n的DIE开始的DIE。这只在使用--debug-dump=info选项时有用。如果指定了这个选项，将会抑制任何头信息的打印以及编号为n之前的所有DIEs的打印。只打印指定DIE的兄弟和子代。

可以与--dwarf-depth选项结合使用。

#### --dwarf-check 
启用对Dwarf信息的一致性进行额外检查。

#### --ctf[=section] 
显示指定CTF section的内容。CTF section本身包含许多子section，所有子section将按顺序显示。默认情况下，显示名为.ctf的部分，这是由ld生成的名称。

#### --ctf-parent=member
如果CTF section包含模糊定义的类型，它将由许多CTF字典的归档组成，所有这些字典都继承自一个包含明确类型的字典。默认情况下，这个成员命名为.ctf，就像包含它的部分一样，但可以在链接时使用ctf_link_set_memb_name_changer 函数更改这个名称。在查看由使用名称更改器重命名父归档成员的链接器创建的CTF归档时，可以使用--ctf-parent 指定用于父项的名称。

#### --sframe[=section]
显示指定SFrame section的内容。默认情况下，显示名为.sframe的部分，这是由ld生成的名称。

#### -G, --stabs
显示请求的任何section的全部内容。显示ELF文件中.stab、.stab.index和.stab.excl部分的内容。这只在系统（如Solaris 2.0）中使用，其中.stab调试符号表条目位于ELF部分中。在大多数其他文件格式中，调试符号表条目与链接符号交错可见，并在--syms输出中可见。

#### --start-address=address
从指定地址开始显示数据。这会影响-d、-r和-s选项的输出。

#### --stop-address=address
在指定地址处停止显示数据。这会影响-d、-r和-s选项的输出。

#### -t / --syms 
打印文件的符号表条目。虽然显示格式不同，但这类似于nm程序提供的信息。输出格式取决于被转储文件的格式，主要有两种类型。一个看起来像这样：
[  4](sec  3)(fl 0x00)(ty   0)(scl   3) (nx 1) 0x00000000 .bss
[  6](sec  1)(fl 0x00)(ty   0)(scl   2) (nx 0) 0x00000000 fred
其中，方括号内的数字是符号表条目的编号，sec数字是节号，fl值是符号的标志位，ty数字是符号的类型，scl数字是符号的存储类别，nx值是与符号关联的辅助条目数。最后两个字段是符号的值和名称。

另一种常见的输出格式，通常用于基于ELF的文件，像这样：
00000000 l    d  .bss   00000000 .bss
00000000 g       .text  00000000 fred
这里，第一个数字是符号的值（有时称为地址）。下一个字段实际上是一组字符和空格，表示设置在符号上的标志位。这些字符如下所述。接下来是符号所属的节名，如果节是绝对的（即不连接任何节），则为*ABS*；如果节在被转储的文件中被引用但未定义，则为*UND*。
节名之后是另一个字段，一个数字，对于常见符号是对齐，对其他符号是大小。最后是符号的名称。
标志字符分为7组如下：
l g u ! 符号是本地符号(l)，全局符号(g)，唯一全局符号(u)，既不是全局也不是本地符号（一个空格）或既是全局又是本地符号(!)。一个符号可能既不是本地也不是全局符号出于多种原因，例如因为它用于调试，但如果它既是本地符号又是全局符号，可能是一个错误的指示。唯一全局符号是对ELF符号绑定的GNU扩展。对于这样的符号，动态链接器将确保在整个进程中仅使用一个具有该名称和类型的符号。
w 符号是弱符号(w)或强符号（一个空格）。
C 符号表示构造函数符号(C)或普通符号（一个空格）。
W 符号是警告符号(W)或正常符号（一个空格）。警告符号的名称是一个消息，如果引用了警告符号后面的符号，就会显示此消息。
I i 符号是对另一个符号的间接引用(I)，在重定位处理期间要评估的函数(i)或普通符号（一个空格）。
d D 符号是调试符号(d)或动态符号(D)或普通符号（一个空格）。
F f O 符号是函数名称(F)，文件名称(f)，对象名称(O)或只是普通符号（一个空格）。

#### -T / --dynamic-syms
打印文件的动态符号表条目。这仅对动态对象有意义，例如某些类型的共享库。这类似于使用nm程序时带有-D（--dynamic）选项所提供的信息。
输出格式类似于--syms选项生成的格式，不同之处在于在符号名称之前插入了一个额外字段，给出了与符号关联的版本信息。如果版本是用于解析未版本化引用的默认版本，则按原样显示，否则将其放在括号中。

#### --special-syms
在显示符号时包括目标认为特殊的符号，这些符号通常不会引起用户的兴趣。

#### -U [d|i|l|e|x|h] /--unicode=[default|invalid|locale|escape|hex|highlight] 
控制字符串中UTF-8编码的多字节字符的显示。默认（--unicode=default）是对它们不做特殊处理。--unicode=locale选项会在当前区域设置下显示序列，当前区域设置可能支持也可能不支持它们。--unicode=hex和--unicode=invalid选项将它们显示为用尖括号或大括号括起来的十六进制字节序列。
--unicode=escape选项将它们显示为转义序列（\uxxxx），--unicode=highlight选项将它们显示为高亮红色的转义序列（如果输出设备支持）。着色旨在引起人们对可能意料之外的Unicode序列的注意。



#### 参考连接：
【1】[https://sourceware.org/binutils/docs/binutils/objdump.html](https://sourceware.org/binutils/docs/binutils/objdump.html)