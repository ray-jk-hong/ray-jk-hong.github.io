---
title: Smmu
categories: 
- Linux
tags:
- Linux Smmu
---

## VMID/ASID/PASID
### 概念
VMID：每个虚拟机都被赋予一个VMID。VMID用于标识TLB项，区分每个TLB是属于哪个虚拟机。这允许多个不同的虚拟机转换在同一时间内在TLB中存在。

ASID：在多进程的情况下，每次切换进程都需要进行TLB清理。这样会导致切换的效率变低。为了解决问题，TLB 引入了 ASID(Address Space ID) 。ASID 的范围是 0-255。ASID 由操作系统分配，当前进程的ASID值被写在 ASID 寄存器 (使用CP15 c3访问)。TLB 在更新页表项时也会将 ASID 写入 TLB。

PASID(Process Address Space ID) ，地址空间ID，是EP的本地ID，每个function都有一组不同的PASID，不同function间的PASID互不相关。带有PASID的TLP Prefix是一种End-End的TLP前缀，PASID与Requester ID一起共同作为请求TLP地址空间的唯一标识。同一PASID在同一系统中可以重复使用。PASID用来对多个进程进行区分。

### PASID获取
iommu_sva_bind_device
iommu_sva_get_pasid
获取pasid

PASID的意义
https://blog.csdn.net/yiyeguzhou100/article/details/128069086
https://www.kernel.org/doc/html/next/x86/sva.html

## SVA的概念
https://www.kernel.org/doc/html/next/x86/sva.html



