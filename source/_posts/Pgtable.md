---
title: Linux页表
categories: 
- Linux
tags:
- Linux 页表
---

## 页表基地址
疑问：TTBR1_EL1保存内核态页表基地址，可以有一份，那用户态呢？（在context switch的时候重新设置TTBR0_EL1寄存器地址？）
可以看[arch/arm64/mm/context.c]文件中的cpu_do_switch_mm函数，确实是在重新设置ttbr0_el1基地址。

## 地址范围
虚拟地址范围需要设置到TCR.TxSZ寄存器中，Linux下会根据CONFIG_ARM64_VA_BITS的值设置上去。可以看mmu寄存器篇。
```c
CONFIG_ARM64_VA_BITS_48=y
CONFIG_ARM64_VA_BITS=48
CONFIG_ARM64_PA_BITS_48=y
CONFIG_ARM64_PA_BITS=48
```

## 页表级数
在Linux中通过CONFIG_PGTABLE_LEVELS设置，CONFIG_PGTABLE_LEVELS=4表示页表级数就是4级，后面也是按照这个来说明

## 页的大小
页的大小在TCR.TG0/TCR.TG1分别表示TTBR_EL0和TTBR_EL1的页大小。
- 00：16Kbyte
- 10：4KByte
- 11：64KByte

## 页表设置

保存stage2的页表基地址
PTE_SHARED
pgprot_val（PAGE_KERNEL）
pteval_t
这些都有什么不同

## 页表walk

## 参考

https://www.cnblogs.com/LoyenWang/p/11406693.html

http://www.wowotech.net/armv8a_arch/__cpu_setup.html
http://www.wowotech.net/armv8a_arch/turn-on-mmu.html
http://www.wowotech.net/armv8a_arch/create_page_tables.html
http://www.wowotech.net/armv8a_arch/__cpu_setup.html

http://www.wowotech.net/armv8a_arch/create_page_tables.html
http://www.wowotech.net/armv8a_arch/turn-on-mmu.html

https://www.cnblogs.com/LoyenWang/p/11406693.html
