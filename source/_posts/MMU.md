---
title: MMU
categories: 
- Aarch64
tags:
- MMU
---

## TTBR(Translation Table Base Register)寄存器
### TTBR寄存器介绍
Stage1的页表翻译时，MMU页表的基地址由TTBR0_EL1, TTBR1_EL1指定。TTBR0_EL1指定用户态页表基地址，TTBR1_EL1指定内核态的页表基地址。
EL2和EL3有TTBR0但没有TTBR1(就是说EL2有TTBR0_EL2, EL3有TTBR_EL3，但没有TTBR1_EL2和TTRB1_EL3)。
- EL2/EL3如果是aarch64, 也只能使用0x0-0x0000FFFF_FFFFFFFF范围的地址（对地址范围有疑问看下一节TTBR地址范围确定一节）。
用户态不能直接访问MMU，当然也没有所谓的TTBR0_EL0，TTBR1_EL0之类的寄存器了。

### TTBR地址范围确定

![TTBR地址范围](/images/MMU/TTBR表示的地址范围.drawio.svg)

### 如何选择（D5-1736）

### TTBR位分配
1) ASID是做什么？

### VTTBR_EL2

## TCR(Translation Control Register)寄存器
包含TCR_EL1/TCR_EL2/TCR_EL3这几种。
决定EL0/EL1在做地址翻译的时候选择哪个TTBR寄存器。
例如：0x0000开始的地址，比如0x0000_ABCD_ABCD_ABCD这样的地址就是通过TTBR0_EL1开始地址翻译，而0xFFFF开始的地址，比如0xFFFF_ABCD_ABCD_ABCD这个地址就是通过TTBR1_EL1开始地址翻译。

### TCR寄存器位分配
![TCR寄存器](/images/MMU/TCR寄存器位图.png)
以下是对TCR寄存器中的各位进行解释

### TB （Top Byte ignored）（D8-2038）
表示top addr是ignore，还是用于MTE的计算
看代码应该是kasa在用这个。
'''[arch/arm64/mm/proc.S]
#ifdef CONFIG_KASAN_SW_TAGS
#define TCR_KASAN_FLAGS TCR_TBI1
#else
#define TCR_KASAN_FLAGS 0
#endif
'''

参考文件：Linux目录下tools/testing/selftests/arm64/tags/tags_test.c 可以查看TB是否使能。
TB ignore使能的情况下，可以将高位用来做计数，实际地址访问的时候是会把高位给忽略掉的

./Documentation/arm64/memory-tagging-extension.rst
prctl(PR_SET_TAGGED_ADDR_CTRL, flags, 0, 0, 0)

### A1
ASID的选择，是使用TTBR_EL1中的，还是使用TTBR_EL0中的。

### AS
ASID是使用8bit，还是使用16bit。

### EPD
包含EPD1、EPD0，表示TTBR_EL1/TTBR_EL0是enable还是disable

### T1SZ/T0SZ
T0SZ: 表示TTBR0_EL1能表示的地址范围，地址范围的计算公式就是2^(64-T1SZ) Bytest:
例如：如果用户态地址范围是48位虚拟地址，那这里应该配置T0SZ=64-48=16, 虚拟地址的范围是 2^(64-16) = 0 ~ 0x0000_FFFF_FFFF_FFFF
T1SZ: 和T0SZ一样，就是表示的是TTBR1_EL1的
这样设置之后，0x0000_开头的地址都走TTBR0_EL1进行地址翻译，0xFFFFF_开头的地址就都走TTBR1_EL1进行地址翻译。
Aarch64的tcr相关的定义都在arch/arm64/include/asm/pgtable-hwdef.h
虚拟地址的位宽在linux用CONFIG_ARM64_VA_BITS定义

### IPS（Intermediate Physical Address Size）
表示物理地址的范围：
000 32 bits, 4 GB.
001 36 bits, 64 GB.
010 40 bits, 1 TB.
011 42 bits, 4 TB.
100 44 bits, 16 TB.
101 48 bits, 256 TB.

### TG（Granule size）
寻址的地址粒度（其实就是Page size）
01 16KByte
10 4KByte
11 64KByte

## MAIR（Memory Attribute Indirection Register)
表示内存的属性。

## 页表设置

保存stage2的页表基地址
PTE_SHARED
pgprot_val（PAGE_KERNEL）
pteval_t
这些都有什么不同

## 页表walk

https://github.com/rcore-os/rCore/blob/master/docs/2_OSLab/g2/memory.md

https://blog.csdn.net/2301_79143213/article/details/137247214?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-0-137247214-blog-109057232.235^v43^pc_blog_bottom_relevance_base1&spm=1001.2101.3001.4242.1&utm_relevant_index=3

## 参考

https://github.com/rcore-os/rCore/blob/master/docs/2_OSLab/g2/memory.md

https://blog.csdn.net/weixin_42135087/article/details/109057232
