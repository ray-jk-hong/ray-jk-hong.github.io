---
title: Mmu寄存器
categories: 
- Aarch64
tags:
- Mmu
---

## TTBR(Translation Table Base Register)寄存器
### TTBR寄存器介绍
Stage1页表翻译时的页表基地址。
TTBR0_EL1指定用户态页表基地址，TTBR1_EL1指定内核态的页表基地址。
EL2和EL3有TTBR0但没有TTBR1(就是说EL2有TTBR0_EL2, EL3有TTBR_EL3，但没有TTBR1_EL2和TTRB1_EL3)。
- 所以EL2/EL3模式下，只能使用0x0-0x0000FFFF_FFFFFFFF范围的地址

用户态不能直接访问MMU，当然也没有所谓的TTBR0_EL0，TTBR1_EL0之类的寄存器了。

### TTBR地址范围确定
![TTBR地址范围](/images/MMU/TTBR表示的地址范围.drawio.svg)
在Linux系统中，0x0000_0000_0000_0000到0x0000_FFFF_FFFF_FFFF范围是被指定为用户态内存，0xFFFF_0000_0000_0000到0xFFFF_FFFF_FFFF_FFFF被指定为内核态地址。
具体前面几个0表示走TTBR0_EL1或者几个F走TTBR1_EL1则是由TCR.TxSZ指定。

### 如何选择（D5-1736）
在TCR.T0SZ, TCR.T1SZ都为16的场景下：
- 0x0000_开头的地址选择TTBR0_EL1为基地址开始页表翻译
- 0xFFFF_开头的地址选择TTBR1_EL1为基地址开始页表翻译

### TTBR位分配
1) ASID是做什么？

### VTTBR_EL2

## TCR(Translation Control Register)寄存器
TCR寄存器的初始化在__cpu_setup函数[arch/arm64/mm/proc.S]中。
```c 
mov_q   tcr, TCR_TxSZ(VA_BITS) | TCR_CACHE_FLAGS | TCR_SMP_FLAGS | \
            TCR_TG_FLAGS | TCR_KASLR_FLAGS | TCR_ASID16 | \       
                TCR_TBI0 | TCR_A1 | TCR_KASAN_SW_FLAGS | TCR_MTE_FLAGS      
```

### TCR寄存器位分配
![TCR寄存器](/images/MMU/TCR寄存器位图.png)
以下是对TCR寄存器中的各位进行解释

### TB （Top Byte ignored）（D8-2038）(MTE)
表示top addr是ignore，还是用于MTE的计算
看代码应该是kasa在用这个。
```c [arch/arm64/mm/proc.S]
#ifdef CONFIG_KASAN_SW_TAGS
#define TCR_KASAN_FLAGS TCR_TBI1
#else
#define TCR_KASAN_FLAGS 0
#endif
```

参考文件：Linux目录下tools/testing/selftests/arm64/tags/tags_test.c 可以查看TB是否使能。
TB ignore使能的情况下，可以将高位用来做计数，实际地址访问的时候是会把高位给忽略掉的

./Documentation/arm64/memory-tagging-extension.rst
prctl(PR_SET_TAGGED_ADDR_CTRL, flags, 0, 0, 0)

TB的使用可以具体参考MTE使用篇。

### A1
ASID的选择，是使用TTBR_EL1中的，还是使用TTBR_EL0中的。

### AS
ASID是使用8bit，还是使用16bit。
AS的初始化在__cpu_setup[arch/arm64/mm/proc.S]，默认使用16位的asid的（TCR_ASID16）。
```c
/*
* Set/prepare TCR and TTBR. We use 512GB (39-bit) address range for
* both user and kernel.
*/
mov_q	x10, TCR_TxSZ(VA_BITS) | TCR_CACHE_FLAGS | TCR_SMP_FLAGS | \
            TCR_TG_FLAGS | TCR_KASLR_FLAGS | TCR_ASID16 | \
            TCR_TBI0 | TCR_A1 | TCR_KASAN_FLAGS
```
但linux代码中asid_bits初始化是在[arch/arm64/mm/context.c]文件中，读的又是ID_AA64MMFR0_EL1寄存器写入的，不知道会不会冲突。
真正asid的分配逻辑也在[arch/arm64/mm/context.c]文件的arm64_mm_context_get函数，保存在mm_struct.context.id里边。
asid在进程切换的时候使用。
TTBRx_EL1中使用ASID的原因：https://blog.csdn.net/WANGYONGZIXUE/article/details/132996049

与asid对应的还有一个pasid。pasid应该是sva使用的，例如某个物理地址在cpu或者多个dma设备之间共享。
创建应该是在[drivers/iommu/iommu-sva.c]文件的iommu_sva_bind_device->iommu_alloc_mm_data

### EPD
包含EPD1、EPD0，表示TTBR_EL1/TTBR_EL0是使能还是去使能。

### T1SZ/T0SZ：表示虚拟地址的范围
T0SZ: 表示TTBR0_EL1能表示的虚拟地址范围，范围的计算公式就是2^(64-T1SZ) Bytest:
例如：如果用户态虚拟地址范围是48位虚拟地址，那这里应该配置T0SZ=64-48=16, 虚拟地址的范围是 2^(64-16) = 0 ~ 0x0000_FFFF_FFFF_FFFF
T1SZ: 和T0SZ一样，就是表示的是TTBR1_EL1的
这样设置之后，0x0000_开头的地址都走TTBR0_EL1进行地址翻译，0xFFFFF_开头的地址就都走TTBR1_EL1进行地址翻译。
Aarch64的tcr相关的定义都在arch/arm64/include/asm/pgtable-hwdef.h

虚拟地址的位宽在linux用CONFIG_ARM64_VA_BITS定义。CONFIG_ARM64_VA_BITS宏的值和TCR.T1SZ的值是否要保持一致？答案是linux内核会根据CONFIG_ARM64_VA_BITS去配置TCR.T1SZ的值

### IPS（Intermediate Physical Address Size）
中间级物理地址范围
- 000 32 bits, 4 GB.
- 001 36 bits, 64 GB.
- 010 40 bits, 1 TB.
- 011 42 bits, 4 TB.
- 100 44 bits, 16 TB.
- 101 48 bits, 256 TB.
应该是和TxSZ表示的大小设置一样就可以了。

### TG（Granule size）
寻址的地址粒度（在Linux场景下，就是page_size对应的大小）
01 16KByte
10 4KByte
11 64KByte

这个寄存器在__cpu_setup函数[arch/arm64/mm/proc.S]中根据
CONFIG_ARM64_4K_PAGES/CONFIG_ARM64_16K_PAGES/CONFIG_ARM64_64K_PAGES的使能情况进行设定。

## MAIR (Memory Attribute Indirection Register)
表示内存的属性。
![TCR寄存器](/images/MMU/MAIR_EL1-1.png)
MAIR_ELx可以定义8种不同的属性。每种属性占8bit。8个bit分为前4个bit和后4个bit分别定义了不同的属性。
![TCR寄存器](/images/MMU/MAIR_EL1-2.png)
![TCR寄存器](/images/MMU/MAIR_EL1-3.png)

上面显示的RW分别别是读写属性。如果读写属性都有，就填1就可以了。

在Linux中，占了5种类型。MAIR_ELx属性在启动阶段就会设置好。设置的代码在
arch/arm64/mm/proc.S的__cpu_setup函数中。
```c
mov_q   mair, MAIR_EL1_SET
```

MAIR_EL1_SET定义如下：
```c 
#define MAIR_ATTRIDX(attr, idx)     ((attr) << ((idx) * 8))

#define MAIR_ATTR_DEVICE_nGnRnE     UL(0x00)
#define MAIR_ATTR_DEVICE_nGnRE      UL(0x04)
#define MAIR_ATTR_NORMAL_NC         UL(0x44)
#define MAIR_ATTR_NORMAL_TAGGED     UL(0xf0)
#define MAIR_ATTR_NORMAL            UL(0xff)
#define MAIR_ATTR_MASK              UL(0xff)

#define MT_NORMAL           0 
#define MT_NORMAL_TAGGED    1                    
#define MT_NORMAL_NC        2                
#define MT_DEVICE_nGnRnE    3
#define MT_DEVICE_nGnRE     4                               

#define MAIR_EL1_SET                            \
    (MAIR_ATTRIDX(MAIR_ATTR_DEVICE_nGnRnE, MT_DEVICE_nGnRnE) |  \ 
    MAIR_ATTRIDX(MAIR_ATTR_DEVICE_nGnRE, MT_DEVICE_nGnRE) |  \    
    MAIR_ATTRIDX(MAIR_ATTR_NORMAL_NC, MT_NORMAL_NC) |     \           
    MAIR_ATTRIDX(MAIR_ATTR_NORMAL, MT_NORMAL) |            \  
    MAIR_ATTRIDX(MAIR_ATTR_NORMAL, MT_NORMAL_TAGGED)) 
```

MAIR_ELx设置完成之后，在寻址的时候，就只需要通过lower attribute中的Attrindex中的index就可以知道这段页的内存是哪种属性了。例如如果Attrindex的值是0就是MT_NORMAL，如果是3就是MT_DEVICE_nGnRnE这种类型的。
![页表描述](/images/MMU/PTE-1.png)
![页表描述](/images/MMU/PTE-2.png)

## 启动时配置mmu
__cpu_setup
__enable_mmu

## 参考

TTBR寄存器中asid的作用
https://blog.csdn.net/weixin_42135087/article/details/123369119


