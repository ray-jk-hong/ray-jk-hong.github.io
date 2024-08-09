---
title: 大页机制
---

https://blog.csdn.net/wangquan1992/article/details/103963108
大页能提高性能的原理：MMU翻译页表，按照2MB翻译，到PMD这层就可以了，不用翻译到PTE阶段。
大页管理机制有以下几种：
1. HugeTLB机制： 
   hstate管理大页，从伙伴系统申请，由order值决定大小。小于order的由伙伴系统申请，大于order的由memblock预留内存中申请或者调用alloc_cont_range申请。
   HugeTLB机制：hugetlbfs文件系统
   (1) HugeTLB就是透过hugetlbfs方式向文件系统提供使用HugeTLB大页机制
   (2) hugetlbfs创建的文件可以被读系统调用操作，但不允许写系统调用操作，可以mmap映射
   如何使用：
   (1) mmap通过传MAAP_HUGETLB标记
   (2) shmget接口传SHM_HUGETLB标记
   (3) memfd的memfd_create接口传MFD_HUGETLB标记
   (4) mount挂载hugetlbfs文件系统，在文件系统里边创建文件并mmap对应文件
2. 组合大页：多个page组合起来管理连续内存空间
3. 透明大页：伙伴系统直接动态分配
   透明大页机制介绍：khugepaged线程
   (1) Hash表是为了便于通过mm_struct指针地址，来找到对应的mm_slot结构
   (2) Khugepaged_scan管理的链表是透明大页遍历扫描的链表，透明大页遍历每个mm_slot 的mm_struct
   (3) 通过mm_struct，遍历每个vma数据结构，扫描vma的地址空间，每次按2M大小扫描对应的pte内容

4. libhugetlbfs：使用大页，将用户态程序的text/data/BSS保存到大页功能，提高性能
5. shmem大页：https://stackoverflow.com/questions/40777684/create-huge-page-shared-memory-for-ipc-in-linux

6. 配置大页内存
   echo 200 > /proc/sys/vm/nr_huagepages
