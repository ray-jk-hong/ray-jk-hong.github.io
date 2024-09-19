---
title: Linux进程调度
categories: 
- Linux进程调度
tags:
- Linux进程调度
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


## 内容
1、代码路径
主线代码：https://github.com/torvalds/linux

2、重点学习内容：
调度类 – CFS， 调度类 – IDLE，调度类 – RT， 调测工具；

3、CPU体系架构，CPU调频，CPU hotplug,  CPU isolate， EAS，NOHZ等
当前了解即可，后面深入到项目中慢慢掌握。


CPU体系架构：

1、代码路径
主线代码：https://github.com/torvalds/linux

2、    重点学习内容：
调度类 – CFS， 调度类 – IDLE，调度类 – RT， 调测工具；

3、    CPU体系架构，CPU调频，CPU hotplug,  CPU isolate， EAS，NOHZ等
当前了解即可，后面深入到项目中慢慢掌握。


CPU体系架构:
http://www.wowotech.net/pm_subsystem/cpu_topology.html

CPU调度发展史:
https://github.com/gatieme/LDD-LinuxDeviceDrivers/blob/master/study/kernel/00-DESCRIPTION/SCHEDULER.md

调度类 -- RT（负载跟踪，负载均衡，选核，选任务，抢占机制, 组调度，CPU bandwidth）:
https://www.cnblogs.com/LoyenWang/p/12584345.html

调度类 -- CFS（负载跟踪，负载均衡，选核，选任务，抢占机制, 组调度，CPU bandwidth）:
http://www.wowotech.net/sort/process_management
https://www.cnblogs.com/LoyenWang/p/12316660.html
https://www.cnblogs.com/LoyenWang/p/12249106.html
https://www.cnblogs.com/LoyenWang/p/12386281.html
https://www.cnblogs.com/LoyenWang/p/12459000.html
https://www.cnblogs.com/LoyenWang/p/12495319.html

调度类 -- IDLE（menu governor,driver框架，CPU idle polling）:
https://www.cnblogs.com/LoyenWang/p/11379937.html

EAS(energy aware schedule):
https://www.kernel.org/doc/html/latest/scheduler/sched-energy.html

cpu 调频:
http://www.wowotech.net/sort/pm_subsystem
https://www.cnblogs.com/LoyenWang/p/11385811.html

cpu hotplug,  CPU isolate:
https://www.cnblogs.com/LoyenWang/p/11397084.html

调试工具（ftrace, perf, top-down，火焰图， schedstat）:
https://www.brendangregg.com/bpf-performance-tools-book.html
https://zhuanlan.zhihu.com/p/60940902
https://blog.csdn.net/wudongxu/article/details/8574755
https://blog.csdn.net/zhangskd/article/details/37902159





