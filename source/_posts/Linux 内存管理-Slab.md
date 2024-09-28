---
title: Linux Slab
categories: 
- Linux
tags:
- Linux
---

## Feature
CONFIG_SLAB_MERGE_DEFAULT 
https://lore.kernel.org/lkml/20230627132131.214475-1-julian.pidancet@oracle.com/T/

## Flag
### slab.h里的flag
kmem_cache_create创建的时候传的flag. 定义在include/linux/slab.h文件下。
1. SLAB_NO_MERGE
```c [include/linux/slab.h]
/*
 * Prevent merging with compatible kmem caches. This flag should be used
 * cautiously. Valid use cases:
 *
 * - caches created for self-tests (e.g. kunit)
 * - general caches created and used by a subsystem, only when a
 *   (subsystem-specific) debug option is enabled
 * - performance critical caches, should be very rare and consulted with slab
 *   maintainers, and not used together with CONFIG_SLUB_TINY
 */
```
出现Wrong slab cache. "lsm_file_cache"，是否跟Slab的merge有关系？

2. SLAB_ACCOUNT: 多numa且打开cgroup的场景，是否一定要定义这个？

### slab_comon.c里组合flag
#define SLAB_NEVER_MERGE (SLAB_RED_ZONE | SLAB_POISON | SLAB_STORE_USER | \
		SLAB_TRACE | SLAB_TYPESAFE_BY_RCU | SLAB_NOLEAKTRACE | \
		SLAB_FAILSLAB | SLAB_NO_MERGE)

#define SLAB_MERGE_SAME (SLAB_RECLAIM_ACCOUNT | SLAB_CACHE_DMA | \
			 SLAB_CACHE_DMA32 | SLAB_ACCOUNT)

## Debug
1. slabtop
2. /proc/slabinfo

## 参考

https://blog.csdn.net/Breeze_CAT/article/details/130015522
https://hiroyukichishiro.com/linux-kernel-vol10/

