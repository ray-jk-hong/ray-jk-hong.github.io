---
title: Linux Dma
categories: 
- Linux Driver
tags:
- Linux Driver
---

## dts配置注意
1. dma-coherent
在相应设备的设备树节点先，添加dma-coherent之后，在调用dma_alloc_coherent的时候?(在4.19内核版本中看，后面变成IOMMU_CACHE属性了)


## 使用接口注意
1. dma_map_single
dma_map_single接口传入的地址，需要是线性地址范围地址，且物理地址需要是连续的。
因为接口内部会使用virt_to_page，如果不是线性地址，则转出来的page结构体地址不合法。

2. dma_alloc_coherent

