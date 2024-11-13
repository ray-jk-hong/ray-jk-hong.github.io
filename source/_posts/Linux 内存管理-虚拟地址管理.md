---
title: Linux虚拟地址管理
categories: 
- Linux
tags:
- Linux
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
#### VM_LOCKED

#### VM_PFNMAP
这个是映射没有page结构体管理的保留内存或者外设物理地址。据说在映射过程中会设置写直通。
remap_pfn_range会直接赋值VM_PFNMAP属性。
如果是直接映射kmalloc等出来的线性地址的话，
最好使用vm_insert_page来映射地址到vma

#### VM_IO
VM_IO将该VMA标记为内存映射的IO区域，VM_IO会阻止系统将该区域包含在进程的存放转存(core dump)中

#### VM_DONTCOPY
#### VM_DONTEXPAND
#### VM_DONTDUMP

#### VM_MIXEDMAP
#### VM_PFNMAP

https://cloud.tencent.com/developer/ask/sof/708258/answer/1042209

## Mmap的flag
MAP_SHARED：
MAP_PRIVATE：表示此地址是本进程独享的，所有对该地址的读写，不会马上写入。例如你在用户态mmap(传入MAP_PRIVATE)，然后在内核态通过在mmap回调中remap_pfn_xx建好了页表。
当用户态写的时候，会发现写的内容不会马上被写到内存中。
MAP_ANONYMOUS：

## Mremap的flag
MREMAP_MAYMOVE：
MREMAP_FIXED：


https://zhuanlan.zhihu.com/p/566614796

http://www.wowotech.net/linux_kenrel/516.html

CONFIG_ARM64_HW_AFDBM



## mmap内存映射时，需要增加的VMA Flag（安全防护）
### 防止mremap
1. A进程remap_pfn_xx方式映射了一段page
2. A进程fork B进程
3. B进程使用mremap方式将A进程的地址从新映射到硬外的虚拟地址
4. 进程A释放这段page之后，B进程仍然可以访问

解决：
1. 需要在remap_pfn_x的时候设置如下几个属性。
VM_DONTCOPY
VM_DONTEXPAND
VM_DONTDUMP
VM_IO
2. vm_struct_ops的mremap回调函数中，挂接函数并直接返回错误。

## 内核态地址映射

### remap_pfn_range, io_remap_pfn_range
1. remap_pfn_range
remap_pfn_range() 将物理内存(通过内核逻辑地址)映射到用户空间进程。它对于实现mmap()系统调用特别有用。

在对一个文件(无论它是否是一个设备文件)调用mmap()系统调用之后，CPU将切换到特权模式并运行相应的 file_operations.mmap() 内核函数，该内核函数将依次调用 remap_pfn_range()。映射区域的内核PTE将被导出并赋予进程，当然使用不同的保护标志。使用一个新的VMA条目(具有适当的属性)更新进程的VMA列表，该进程将使用PTE访问相同的内存。

因此，内核只是复制PTEs，而不是通过复制来浪费内存。但是，内核和用户空间PTEs具有不同的属性。remap_pfn_range() 的原型如下:

一个成功的调用将返回0，如果失败将返回一个否定的错误代码。remap_pfn_range()的大多数参数是在mmap()方法被调用时提供的:

vma: 这是内核在调用 file_operations.mmap() 时提供的虚拟内存区域。它对应于应该进行映射的用户进程vma。
addr: 这是VMA应当开始的用户虚拟地址(vma->vm_start)，这将完成 addr 和 addr + size 之间的虚拟地址范围的映射。
pfn: 表示要映射的内核内存区域的PFN。它对应于 PAGE_SHIFT 位右移的物理地址。应该考虑vma偏移量(映射必须开始的对象偏移量)以产生PFN。因为VMA结构的 vm_pgoff 字段包含页面数形式的偏移值，所以它正是您需要(使用PAGE_SHIFT左移)以字节形式提取偏移值的:offset = VMA ->vm_pgoff << PAGE_SHIFT)。最后，pfn = virt_to_phys(buffer + offset) >> PAGE_SHIFT。
size: 这是以字节为单位的被重映射的区域的大小。
flags: 表示新VMA请求的保护。驱动程序可以破坏默认值，但是应该使用在vma->vm_page_prot中找到的值作为使用OR操作符的框架，因为它的一些位已经由用户空间设置了。其中一些flags是:
VM_IO，它指定设备的内存映射I/O
VM_DONTCOPY，它告诉内核不要在fork上复制这个vma
VM_DONTEXPAND，它阻止vma使用mremap(2)展开
VM_DONTDUMP，它阻止vma包含在核心转储中
你可能需要修改这个值，以便禁用缓存，如果使用这个I/O内存(vma->vm_page_prot = pgprot_noncached(vma->vm_page_prot);)。

2. io_remap_pfn_range
当涉及到将I/O内存映射到用户空间时，remap_pfn_range()函数将不再适用。合适的函数是 io_remap_pfn_range()，它们的参数相同。唯一改变的是PFN的来源。它的原型如下:
当尝试将I/O内存映射到用户空间时，不需要使用 ioremap()。ioremap()用于内核目的映射(将I/O内存映射到内核地址空间)，而 io_remap_pfn_range 用于用户空间目的。

只需将实际的物理I/O地址(通过PAGE_SHIFT向下移动以产生PFN)直接传递给io_remap_pfn_range()。即使在某些体系结构中io_remap_pfn_range()被定义为remap_pfn_range()，但在其他体系结构中却不是这样。出于可移植性的原因，您应该只在PFN参数指向RAM的情况下使用remap_pfn_range()，而在phys_addr指向I/O内存的情况下使用io_remap_pfn_range()。

https://www.cnblogs.com/wanglouxiaozi/p/15036469.html

### vm_insert_page()/vm_insert_pfn()
映射内核page到用户态

## 参考
https://www.kernel.org/doc/gorman/html/understand/
