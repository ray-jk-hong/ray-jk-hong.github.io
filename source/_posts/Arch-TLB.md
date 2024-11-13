---
title: TLB
categories: 
- Arch
tags:
- Arch
---

## TLB
### TLB介绍
TLB是一个cache、是保存MMU最近转换过的内容(TLB hit)。在每次MMU地址转换的时候、都会先
查看TLB中是否存在、如果有的话就可以直接访问。如果TLB中没有(TLB miss)、就会通过MMU做
转换并保存在TLB中。TLB中不仅保存虚拟地址和对应的物理地址、也保存
1)	attributes such as memory type (见memory ordering)
2)	cache policies
3)	access permissions 
4)	the Address Space ID (ASID), and the Virtual Machine ID (VMID)。

如果MMU页表在建立之后、中途需要调整变化、在修改完MMU对应的页表之后、就要刷新(invalidate)TLB。

### TLB刷新
TLBI命令就是负责刷新TLB。

TLBI <type><level>{IS} {, <Xt>}

type字段：
ALL：刷新所有TLB entry
VMALL: 刷新所有stage 1TLB entry


https://aijishu.com/a/1060000000381352
