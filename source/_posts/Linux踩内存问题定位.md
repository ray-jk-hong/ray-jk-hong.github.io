---
title: Prctl
categories: 
- Linux Debug
tags:
- Linux Debug
---

## 打开Kasan选项

## 打开page alloc/free相关宏
```c
CONFIG_PAGE_EXTENSION=y
CONFIG_DEBUG_PAGEALLOC=y
CONFIG_DEBUG_PAGEALLOC_ENABLE_DEFAULT=y
# CONFIG_PAGE_OWNER is not set
CONFIG_PAGE_POISONING=y
CONFIG_PAGE_POISONING_NO_SANITY=y
原理：内核空间通过线性映射可以访问所有的内存（页表提前被创建），上述debug版本在内存释放后会释放线性映射，这样free的内存就不能在访问了
```
