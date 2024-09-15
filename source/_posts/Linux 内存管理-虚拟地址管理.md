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
1. 获取newmm = struct mm_struct *get_task_mm(struct task_struct *task), oldmm = current->mm // get_task_mm就是调用mmget，里边用了task_lock锁
2. mmap_lock锁需要抓？down_write?down_read锁怎么区分  
3. mmgrab()
4. current->mm = newmm;
5. 做vma分配等事情 // 在流程中需要使用current->mm
6. current->mm = oldmm;
7. mmdrop()
8. mmput()
```
上面的操作就是A进程拿了B进程的mm并做操作的流程，但仅限于A进程是用户态进程的场景，如果A进程是内核态线程，需要做如下操作。

```c
1. 获取newmm = struct mm_struct *get_task_mm(struct task_struct *task), oldmm = current->mm // get_task_mm就是调用mmget，里边用了task_lock锁
2. mmap_lock锁需要抓？down_write?down_read锁怎么区分  
3. kthread_use_mm
4. 做vma分配等事情 // 在流程中需要使用current->mm
5. kthread_unuse_mm;
6. mmput()
```
如果A进程是内核的线程，拿mmgrap要替换成kthread_use_mm()【这个很重要】

## mm notifier
mm notifier可以提交注册，观测mm_struct是否在调用exit_mmap函数销毁所有的vma。
(__mmput()->exit_mmap()->mmu_notifier_release()函数中会调用mm notifier的回调函数)

## 内核虚拟地址管理[include/linux/mm.h]

### VM Flag
VM_LOCKED

#### VM_PFNMAP
这个是映射没有page结构体管理的保留内存或者外设物理地址。据说在映射过程中会设置写直通。
remap_pfn_range会直接赋值VM_PFNMAP属性。
如果是直接映射kmalloc等出来的线性地址的话，
最好使用vm_insert_page来映射地址到vma

## Mmap的flag
MAP_SHARED：
MAP_PRIVATE：表示此地址是本进程独享的，所有对该地址的读写，不会马上写入。例如你在用户态mmap(传入MAP_PRIVATE)，然后在内核态通过在mmap回调中remap_pfn_xx建好了页表。
当用户态写的时候，会发现写的内容不会马上被写到内存中。
MAP_ANONYMOUS：
MREMAP_MAYMOVE：
MREMAP_FIXED：


https://zhuanlan.zhihu.com/p/566614796

http://www.wowotech.net/linux_kenrel/516.html

CONFIG_ARM64_HW_AFDBM



## 防止mremap
1. A进程remap_pfn_xx方式映射了一段page
2. A进程fork B进程
3. B进程使用mremap方式将A进程的地址从新映射到硬外的虚拟地址
4. 进程A释放这段page之后，B进程仍然可以访问

解决：
1. 需要在remap_pfn_x的时候设置如下几个属性。
VM_DONTCOPY
VM_DONTEXPAND
VM_DONTDUMP
2. vm_struct_ops的mremap回调函数中，挂接函数并直接返回错误。


## 参考
https://www.kernel.org/doc/gorman/html/understand/
