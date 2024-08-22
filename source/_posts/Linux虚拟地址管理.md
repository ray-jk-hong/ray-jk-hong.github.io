---
title: Linux虚拟地址管理
categories: 
- Linux
tags:
- Linux MM
---

## 内核虚拟地址管理[include/linux/mm.h]

### VM Flag
VM_LOCKED

#### VM_PFNMAP
这个是映射没有page结构体管理的保留内存或者外设物理地址。据说在映射过程中会设置写直通。
remap_pfn_range会直接赋值VM_PFNMAP属性。
如果是直接映射kmalloc等出来的线性地址的话，
最好使用vm_insert_page来映射地址到vma

## Mmap的flag

MAP_SHARED

https://zhuanlan.zhihu.com/p/566614796

http://www.wowotech.net/linux_kenrel/516.html

CONFIG_ARM64_HW_AFDBM


