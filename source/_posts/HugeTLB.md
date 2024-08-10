---
title: HugeTLB
---
## 参考
https://blog.csdn.net/wangquan1992/article/details/103963108
https://blog.csdn.net/hbuxiaofei/article/details/128402495
https://blog.csdn.net/tony_vip/article/details/113791585

https://students.mimuw.edu.pl/ZSO/Wyklady/11_extXfs/TransparentHugePages.pdf

## 为什么使用大页   
#### 大页提高性能
大页能提高性能的原理：MMU翻译页表，按照2MB翻译，到PMD这层就可以了，不用翻译到PTE阶段。

## 大页类型
### HugeTLB机制： 
   hstate管理大页，从伙伴系统申请，由order值决定大小。小于order的由伙伴系统申请，大于order的由memblock预留内存中申请或者调用alloc_cont_range申请。
   HugeTLB机制：hugetlbfs文件系统
   (1) HugeTLB就是透过hugetlbfs方式向文件系统提供使用HugeTLB大页机制
   (2) hugetlbfs创建的文件可以被读系统调用操作，但不允许写系统调用操作，可以mmap映射

### 复合大页（Compound pages）：多个page组合起来管理连续内存空间
### 透明大页（Transparent Huge Pages）：伙伴系统直接动态分配
   透明大页机制介绍：khugepaged线程
   1) Hash表是为了便于通过mm_struct指针地址，来找到对应的mm_slot结构
   2) Khugepaged_scan管理的链表是透明大页遍历扫描的链表，透明大页遍历每个mm_slot 的mm_struct
   3) 通过mm_struct，遍历每个vma数据结构，扫描vma的地址空间，每次按2M大小扫描对应的pte内容

透明大页由于性能抖动以及挂死等问题，被禁用：
https://www.pingcap.com/blog/transparent-huge-pages-why-we-disable-it-for-databases/

## 如何使用大页
### 用户态使用大页
   用户态使用大页有以下几种方法：
   - mount一个特殊的hugetlbfs文件系统，在上面创建文件，然后用mmap()进行访问, 但文件是只读的。也可以使用libhugetlbfs。
   - shmget/shmat，调用shmget申请共享内存加上SHM_HUGETLB标志。
   - mmap()时指定MAAP_HUGETLB标志。
   - memfd的memfd_create传MFD_HUGETLB标记
     
#### mmap方式使用示例
1) cat /proc/meminfo | grep -i huge查看大页预留情况
   AnonHugePages:      2048 kB
   HugePages_Total:     200
   HugePages_Free:      200
   HugePages_Rsvd:        0
   HugePages_Surp:        0
   Hugepagesize:       2048 kB
2) 预留大页（200个大页）
   echo 200 > /proc/sys/vm/nr_huagepages
   或者
   sysctl vm.nr_hugepages=200
3) mmap + memset的时候，在mmap参数中添加MAP_HUGETLB申请使用大页
   例如：
   (1) 申请使用大页
      size = 2 * 1024 * 1024;
      addr = mmap(NULL, size, PROT_READ | PROT_WRITE, MAP_PRIVATE | MAP_ANONYMOUS | 0x40000 /*MAP_HUGETLB*/, -1, 0);
      memset(addr, 0, size);
   (2) 重新查看大页使用情况
      AnonHugePages:      2048 kB
      HugePages_Total:     200
      HugePages_Free:      199
      HugePages_Rsvd:        0
      HugePages_Surp:        0
      Hugepagesize:       2048 kB
#### mount hugetlbfs方式示例
   libhugetlbfs：使用大页，将用户态程序的text/data/BSS保存到大页功能，提高性能

#### shmemget方式示例
   shmem大页：https://stackoverflow.com/questions/40777684/create-huge-page-shared-memory-for-ipc-in-linux

### 内核态申请大页


