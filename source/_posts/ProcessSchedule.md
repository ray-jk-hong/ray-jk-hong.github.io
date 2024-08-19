---
title: Process Schedule
categories: 
- Linux
tags:
- Linux Process
---

## 进程调度算法
CFS/FIFO/RR

## 进程sched事件追踪
```[bash] [sched事件追踪]
cd /sys/kernel/debug/tracing/events/sched
echo 1 > sched_switch/enable
echo 1 > sched_wakeup/enable
echo 1 > sched_wakeup_new/enable
echo 1 > sched_waking/enable
echo 1 > sched_process_fork/enable
echo 1 > sched_stat_runtime/enable
echo 1 > /sys/kernel/debug/tracing/events/irq/enable
echo 1 > /sys/kernel/debug/tracing/tracing_on
```

## 参考
https://blog.csdn.net/weixin_51760563/article/details/122789480



