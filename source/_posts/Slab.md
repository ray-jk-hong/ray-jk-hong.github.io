---
title: Slab
categories: 
- Linux
tags:
- Linux MM
---

## Feature
CONFIG_SLAB_MERGE_DEFAULT 
https://lore.kernel.org/lkml/20230627132131.214475-1-julian.pidancet@oracle.com/T/

## Flag
kmem_cache_create创建的时候传的flag
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
2. SLAB_ACCOUNT: 多numa且打开cgroup的场景，是否一定要定义这个？
