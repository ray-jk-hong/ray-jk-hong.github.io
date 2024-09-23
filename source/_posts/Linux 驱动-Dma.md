---
title: Linux Dma
categories: 
- Linux Driver
tags:
- Linux Driver
---

## dts配置注意
1. dma-coherent
在相应设备的设备树节点先，添加dma-coherent之后，在调用dma_alloc_coherent的时候，申请内存建页表的时候，会按照cacheable的方式建页表。？？
(在4.19内核版本中看，后面变成IOMMU_CACHE属性了)


## 使用接口注意
1. dma_map_single
  dma_map_single接口传入的地址，需要是线性地址范围地址，且物理地址需要是连续的。
  因为接口内部会使用virt_to_page，如果不是线性地址，则转出来的page结构体地址不合法。

2. dma_alloc_coherent
设备的dts中，根据有没有定义dma-coherent属性，dma_alloc_coherent的行为有所区别。
__dma_alloc函数里边is_device_dma_coherent()就是返回dts里边有没有dma-coherent的。
(1) 有定义dma-coherent：dma_alloc_coherent直接返回申请的cma地址。这个是在线性地址范围内的的、所以dma_alloc_coherent返回的地址可以直接做virt_to_phys就得到正确的物理地址。
(2) 没有定义dma-coherent：会分配一些pages, 然后使用get_vm_area_caller分配vm_struct，并使用map_kernel_range将pages重新映射到vm_struct起始地址中。
  重新映射的时候会使用wc属性（这是因为你没有使能dma-coherent属性表示DMA与CPU是没有办法使用硬件手段保持cache一致的，所以就需要使用wc属性将数据直接写入到DDR中以便DMA读写）。
  这种场景下拿到的虚拟地址是重新映射过的的地址，不能使用virt_to_phys方式转成物理地址，没有办法直接拿到物理地址的。

## 参考
https://www.byteisland.com/dma-%E4%B8%8E-scatterlist-%E6%8A%80%E6%9C%AF%E7%AE%80%E4%BB%8B/

