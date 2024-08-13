---
title: Spinlock
categories: 
- Linux
tags:
- Linux Spinlock
---

## Spinlock嵌套使用
spinlock锁的类型有以下几种：
- spin_lock/spin_unlock
- spin_lock_bh/spin_unlock_bh
- spin_lock_irq/spin_unlock_irq
- spin_lock_irqsave/spin_unlock_irqrestore
spinlock锁的严格程度是由弱到强。弱的spinlock类型不能被强的spinlock类型给嵌套。

### spin_lock_irq套spin_lock_bh
如下图所示，spin_lock_irq套了spin_lock_bh来保护临界区的函数，分别在进程上下文和tasklet中都被调用。

![TTBR地址范围](/images/Spinlock/spinlock嵌套-1.svg)

因为在spin_unlock_bh函数中，会查看当前是否有堆积的tasklet并直接调用do_softirq进行调用，导致spin_lock_irq死锁。
