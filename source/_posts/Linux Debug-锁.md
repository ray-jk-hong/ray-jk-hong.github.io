---
title: Linux 锁-Debug
categories: 
- Linux
tags:
- Linux
---

CONFIG_LOCKUP_DETECTOR=y
CONFIG_PROVE_LOCKING=y
CONFIG_DEBUG_LOCK_ALLOCK=y
CONFIG_DEBUG_LOCKDEP=y
CONFIG_LOCK_STAT=y
CONFIG_DEBUG_LOCKING_API_SELFTESTS=y

CONFIG_DEBUG_MUTEXES: 可以用来调试mutex死锁。
CONFIG_DEBUG_SPINLOCK: 可以用来调试spinlock死锁。
CONFIG_DEBUG_LOCK_ALLOC: 可以用来检查锁是否被异常释放(包含mutex spin rwlock rwsem)。
CONFIG_DEBUG_ATOMIC_SLEEP: 可以用来检查是否在原子上下文中使用might_sleep相关操作。
CONFIG_SCHED_STACK_END_CHECK: 可以用来检查栈是否溢出。
CONFIG_DETECT_HUNG_TASK: 可有用来检查是否有进程长时间处于阻塞状态。
