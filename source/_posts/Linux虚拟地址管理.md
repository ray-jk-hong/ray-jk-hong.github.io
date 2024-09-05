---
title: Linux虚拟地址管理
categories: 
- Linux MM
tags:
- Linux MM
---

## 软件架构
### mm_struct
疑问：mmget, mmgrab的区别
答案：
1. mmget增加mm_struct->mm_users计数，即使进程退了，也不会将mm_struct的vma之类的都释放掉
2. mmgrap增加mm_count计数，防止mm_struct被释放（即使是进程退了）

使用方法：

```c
1. 获取struct mm_struct *get_task_mm(struct task_struct *task) // 这里就是调用mmget，但里边用了task_lock锁
2. mmap_lock锁需要抓？down_write?down_read锁怎么区分  
3. mmgrab()
4. 做vma分配等事情
5. mmdrop()
6. mmput()
```

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

## 参考
https://www.kernel.org/doc/gorman/html/understand/
