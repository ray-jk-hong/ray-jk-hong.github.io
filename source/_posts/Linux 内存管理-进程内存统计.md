---
title: Linux 进程内存统计
categories: 
- Linux 
tags:
- Linux 
---

## 用户态角度
ps -aux等方式打印进程信息时，能看到内存的使用情况，但显示都是VSS/RSS/PSS/USS等。
这几种意义如下

### USS（Unique Set size）
是进程实际独自占用的物理内存（不包含共享库占用的内存）。USS 揭示了单个进程运行中真实的内存增量大小。如果单个进程终止，USS 就是实际返还给系统的内存大小。
当怀疑某个进程中内存泄漏时，可以查看 USS 的数值。

### PSS（Proportional Set Size）
是单个进程运行时实际占用的物理内存（包含比例分配共享库占用的内存）。
对比 RSS 来说，PSS 中的共享库内存是按照比例计算的。一个共享库有 N 个进程使用，那么该库比例分配给 PSS 的大小为：1/N；
PSS 明确的表示了单个进程在系统总内存中的实际使用量。

### RSS（ Resident Set Size）
是进程在 RAM 中实际保存的总内存（包含共享库占用的共享内存总数）。
即单个进程实际占用内存大小，RSS 可能会产生误导，因为包含了共享库占用的共享内存总数。然而实际上一个共享库仅会被加载到内存中一次，无论被多少个进程使用。
所以，RSS 不能准确的表示单个进程的内存占用情况（因为RSS把所有共享库大小都算进了当前进程使用的物理内存大小中）。

### VSS
是进程向系统申请的虚拟内存（包含共享库内存总数），即单个进程全部可访问的虚拟地址空间，其大小可能包括还尚未访问的虚拟地址，所以VSS大小会大于等于实际进程占用的物理地址大小。

VSS >= RSS >= PSS >= USS
### 参考
https://segmentfault.com/a/1190000040077427

疑问：进程通过ioctl进入到内核中，通过kmalloc等方式申请的内存，能否统计到USS/PSS/RSS中？？

## 内核态角度
OOM时打印的内容
dump_tasks

## 内核态/用户态看到的统计信息关联

## 参考
https://www.lineo.co.jp/blog/linux/sol01-processmemory.html

https://www.cnblogs.com/sunsky303/p/13494977.html

https://stackoverflow.com/questions/1353283/what-does-the-linux-proc-meminfo-mapped-topic-mean
