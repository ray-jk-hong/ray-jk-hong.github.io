---
title: 页表管理
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

## MMU
### MMU寄存器
#### TCR(Translation Control Register)寄存器（包含TCR_EL1/TCR_EL2/TCR_EL3这几种）(D8-2038)

##### TCR寄存器位分配

###### T1SZ/T0SZ
表示TTBR0_ELx能表示的地址范围，地址范围的计算公式就是2^(64-T1SZ) Bytest:
例如：如果用户态地址范围是48位虚拟地址，那这里应该配置T0SZ=64-48=16, 虚拟地址的范围是 2^(64-16) = 0 ~ 0x0000_FFFF_FFFF_FFFF

#### TTBR(Translation Table Base Register)寄存器
##### TTBR0_EL1, TTBR1_EL1寄存器
###### TTBR寄存器介绍
Stage1的页表翻译时，MMU页表的基地址由TTBR0_EL1, TTBR1_EL1指定。TTBR0_EL1指定用户态页表基地址，TTBR1_EL1指定内核态的页表基地址。
EL2和EL3有TTBR0但没有TTBR1(就是说EL2有TTBR0_EL2, EL3有TTBR_EL3，但没有TTBR1_EL2和TTRB1_EL3)。
- EL2/EL3如果是aarch64, 也只能使用0x0-0x0000FFFF_FFFFFFFF范围的地址（对地址范围有疑问看下一节TTBR地址范围确定一节）。
用户态不能直接访问MMU，当然也没有所谓的TTBR0_EL0，TTBR1_EL0之类的寄存器了。

###### TTBR地址范围确定

![Alt text](./TTBR0-TTBR1地址范围和选择.drawio.svg)
<img src="./TTBR0-TTBR1地址范围和选择.drawio.svg">

###### 如何选择（D5-1736）

###### TTBR位分配

1) ASID是做什么？


##### VTTBR_EL2

## 页表设置


保存stage2的页表基地址

PTE_SHARED

pgprot_val（PAGE_KERNEL）

pteval_t

这些都有什么不同



## 页表walk

https://github.com/rcore-os/rCore/blob/master/docs/2_OSLab/g2/memory.md

https://blog.csdn.net/2301_79143213/article/details/137247214?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-0-137247214-blog-109057232.235^v43^pc_blog_bottom_relevance_base1&spm=1001.2101.3001.4242.1&utm_relevant_index=3

https://blog.csdn.net/weixin_42135087/article/details/109057232

