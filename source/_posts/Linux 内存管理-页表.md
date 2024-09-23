---
title: Linux页表
categories: 
- Linux MM
tags:
- Linux MM
---

## 页表基地址
疑问：TTBR1_EL1保存内核态页表基地址，可以有一份，那用户态呢？（在context switch的时候重新设置TTBR0_EL1寄存器地址？）
可以看[arch/arm64/mm/context.c]文件中的cpu_do_switch_mm函数，确实是在重新设置ttbr0_el1基地址。
疑问：多个用户态进程在不同的cpu上并发，但只有一个ttbr0_el1寄存器？

内核的页表基地址：swapper_pg_dir

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

## 页的属性 [G3-3596]
### PXN: 特权模式下不能执行。阻止内核态执行用户态代码
https://blog.csdn.net/andlee/article/details/121227038

### XN: 表示在任何模式下都不能执行
### UXN：表示在用户模式下不能执行
### AP[2:1]: 表示访问权限
AP[1]：
- 0：EL0不可访问，EL1可访问
- 1：EL0可访问，EL1可访问
AP[2]：
- 0：可读写
- 1：只读

AP[2:1]可以组织个表
[AP]        [EL0]              [EL1/2/3]
[00]        [No access]        [Read and write]
[01]        [Read and write]   [Read and write]
[10]        [No access]        [Read Only]
[11]        [Read Only]        [Read Only]

### AF
表示页表是否是第一次被访问。其实就是在建页表的时候，这个比特位就需要被软件设置为1。
AF为位0表示从来没有被访问过，表示在第一次访问这个地址的时候，TLB就不需要针对此地址做TLB Flush，因为这个地址从来没有在TLB中存在过。
ARMv8是要求软件做AF的设置的，pte_mkyoung函数，可以看到在很多缺页和主动建页表的函数中存在。

### nG
- 0：表示此页是可以被所有的进程访问的
- 1：此页只有当前进程访问。
nG的设置会影响TLB Flush的策略。

### DBM（Dirt Bit Modifier）
脏比特位(Dirty Bit Modifier)。Linux内核使用PTE_DBM宏来表示该比特位

https://lkml.indiana.edu/hypermail/linux/kernel/2307.2/00102.html

https://cloud.tencent.com/developer/article/2117804

### Contiguous
表示当前页表项处在一个连续物理页面集合中，可以使用一个单一的TLB表项进行优化。
Linux内核使用PTE_CONT宏来表示该比特位

PTE_CONT

### AttrIndx[2:0]
配合MAIR寄存器表示内存属性。

### PBHA ?

### Reserved for Software use ?

## 页表设置

保存stage2的页表基地址
PTE_SHARED
pgprot_val（PAGE_KERNEL）
pteval_t
这些都有什么不同

## 页表walk

## 上下文切换(Context Switching)

## 参考

https://www.cnblogs.com/LoyenWang/p/11406693.html

http://www.wowotech.net/armv8a_arch/__cpu_setup.html
http://www.wowotech.net/armv8a_arch/turn-on-mmu.html
http://www.wowotech.net/armv8a_arch/create_page_tables.html
http://www.wowotech.net/armv8a_arch/__cpu_setup.html

http://www.wowotech.net/armv8a_arch/create_page_tables.html
http://www.wowotech.net/armv8a_arch/turn-on-mmu.html

https://www.cnblogs.com/LoyenWang/p/11406693.html
