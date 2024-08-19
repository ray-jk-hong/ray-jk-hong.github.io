---
title: Workqueue
categories: 
- Linux
tags:
- Linux Workqueue
---

## WQ_UNBOUND类型的Workqueue创建流程
在调用alloc_workqueue的时候创建的结构体如下图：
![Workqueue初始化结构体创建](/images/Workqueue/Workqueue创建流程.drawio.svg)

### 创建进程所属的结构体
1. 创建struct workqueue_struct和struct workqueue_attrs
2. 创建struct pool_workqueue
    此结构体如果ordered为true则只有一个，如果ordered是

### 创建全局的结构体
1. 创建struct worker_pool。
    此结构体是全局的，根据wqattrs_equal函数的对比结果，可能创建新的，可能会沿用旧的。
2. 在创建worker_pool的时候，对应的会创建一个新的struct worker。
    在调用完alloc_workqueue("xxx")之后，会生成[xxx]线程(5.xx版本)或者kworker/R-xxx线程，这个就是一个rescue线程。
    worker_thread线程不一定会被创建出来新的。因为unbound类型的attr已有的话，就会沿用以前的。
    如果创建新的，就可以在内核线程中新创建[kworker/%d:%d%d, pool->cpu, id]这样的新的内核线程。

## WQ_UNBOUND类型的Wokqueue指定执行的cpu
1. 可以在调用alloc_workqueue的时候传入WQ_SYSFS
    例如：alloc_workqueue(xxx, WQ_SYSFS)，xxx是workqueue的名字
2. 修改/sys/devices/virtual/workqueue/xxx/cpumask。
    例如cpu个数是10个则/sys/devices/virtual/workqueue/xxx/cpumask是0x3FF。
    如果想把cpu0给mask掉不让workqueue执行到cpu0，就写入0x3FE即可。

原理是写入之后，workqueue创建worker的流程会重新执行，并将worker_thread线程bind到对应的cpu上。

## Workqueue Trace
代码在[include/trace/events/workqueue.h]

### Trace节点
在sys trace目录/sys/kernel/debug/tracing/events/workqueue下，可以看到几个event节点。

```bash
/sys/kernel/debug/tracing/events/workqueue/
workqueue_activate_work  workqueue_execute_end  workqueue_execute_start  workqueue_queue_work
```
所有的都打开就可以看到所有work执行的过程

### Work消耗太多CPU Cycles(Top命令能看到)

Worker线程通过ps命令打印如下：
```bash
root      5671  0.0  0.0      0     0 ?        S    12:07   0:00 [kworker/0:1]
root      5672  0.0  0.0      0     0 ?        S    12:07   0:00 [kworker/1:2]
root      5673  0.0  0.0      0     0 ?        S    12:12   0:00 [kworker/0:0]
root      5674  0.0  0.0      0     0 ?        S    12:13   0:00 [kworker/1:0]
```

以下几种可能
1.Work切换频繁
```bash
$echo workqueue:workqueue_queue_work > /sys/kernel/tracing/set_event
$cat /sys/kernel/tracing/trace_pipe > out.txt                    
(wait a few secs)                                                 
^C
```
这样可以看到所有的Work执行的情况。

```bash
<idle>-0       [005] dNs.. 25598.686997: workqueue_queue_work: work struct=00000000b8691ef7 function=nf_flow_offload_work_gc workqueue=events_power_efficient req_cpu=24 cpu=-1
```
function=‘Work函数名’就是Work的函数。

2.Work一次执行消耗太多
使用如下方式查看kwork的调用栈。
```bash
$cat /proc/THE_OFFENDING_KWORKER/stack
```
THE_OFFENDING_KWORKER就是Worker线程的pid。

## WQ flag的意义
```c
/*
  1  * Workqueue flags and constants.  For details, please refer to
  2  * Documentation/core-api/workqueue.rst.
  3  */
  4 enum {
  5     WQ_UNBOUND      = 1 << 1, /* not bound to any cpu */
  6     WQ_FREEZABLE        = 1 << 2, /* freeze during suspend */
  7     WQ_MEM_RECLAIM      = 1 << 3, /* may be used for memory reclaim */
  8     WQ_HIGHPRI      = 1 << 4, /* high priority */
  9     WQ_CPU_INTENSIVE    = 1 << 5, /* cpu intensive workqueue */
 10     WQ_SYSFS        = 1 << 6, /* visible in sysfs, see workqueue_sysfs_register() */
 11 
 12     /*
 13      * Per-cpu workqueues are generally preferred because they tend to
 14      * show better performance thanks to cache locality.  Per-cpu
 15      * workqueues exclude the scheduler from choosing the CPU to
 16      * execute the worker threads, which has an unfortunate side effect
 17      * of increasing power consumption.
 18      *
 19      * The scheduler considers a CPU idle if it doesn't have any task
 20      * to execute and tries to keep idle cores idle to conserve power;
 21      * however, for example, a per-cpu work item scheduled from an
 22      * interrupt handler on an idle CPU will force the scheduler to
 23      * execute the work item on that CPU breaking the idleness, which in
 24      * turn may lead to more scheduling choices which are sub-optimal
 25      * in terms of power consumption.
 26      *
 27      * Workqueues marked with WQ_POWER_EFFICIENT are per-cpu by default
 28      * but become unbound if workqueue.power_efficient kernel param is
 29      * specified.  Per-cpu workqueues which are identified to
 30      * contribute significantly to power-consumption are identified and
 31      * marked with this flag and enabling the power_efficient mode
 32      * leads to noticeable power saving at the cost of small
 33      * performance disadvantage.
 34      *
 35      * http://thread.gmane.org/gmane.linux.kernel/1480396
 36      */
 37     WQ_POWER_EFFICIENT  = 1 << 7,
 38 
 39     __WQ_DESTROYING     = 1 << 15, /* internal: workqueue is destroying */
 40     __WQ_DRAINING       = 1 << 16, /* internal: workqueue is draining */
 41     __WQ_ORDERED        = 1 << 17, /* internal: workqueue is ordered */
 42     __WQ_LEGACY     = 1 << 18, /* internal: create*_workqueue() */
 43     __WQ_ORDERED_EXPLICIT   = 1 << 19, /* internal: alloc_ordered_workqueue() */
 44 
 45     WQ_MAX_ACTIVE       = 512,    /* I like 512, better ideas? */
 46     WQ_UNBOUND_MAX_ACTIVE   = WQ_MAX_ACTIVE,
 47     WQ_DFL_ACTIVE       = WQ_MAX_ACTIVE / 2,
 48 };
```

1. WQ_UNBOUND
工作队列的设计目的是在提交任务的 CPU 上运行这些任务，以期获得更好的内存缓存行为。这个标志关闭了这种行为，允许提交的任务在系统中的任何 CPU 上运行。它适用于任务可以运行很长时间的情况，这样让调度程序管理它们的位置会更好。
设置这个标记之后，每次queue_work，如果输入的cpu是-1的时候，所有unbound类型的workqueue会主动使用rr方式找到下一个cpu并找到对应cpu的struct pool_workqueue->struct worker_pool挂接work上去。

## 非WQ_UNBOUND类型
per-CPU的Worker 早在 CPU prepare 阶段就通过以下步骤创建完毕：
1. workqueue_init_early初始化cpu_worker_pools
2. workqueue_prepare_cpu->create_worker 创建。
所以在create_workqueue的时候，不再需要创建worker。


参考：
https://lwn.net/Articles/403891/
https://lwn.net/Articles/403918/

## 接口使用注意
#### cancle_work_sync
如果work的回调函数中有等待信号量等操作的时候，直接调用destroy_workqueue是会有报错的。
正确的做法是:

1.唤醒work回调函数的信号量等待

2.调用cancel_work_sync等待work结束

3.调用destroy_workqueue销毁workqueue

## 参考
Linux/Documentation/core-api/workqueue.rst
https://www.kernel.org/doc/html/v5.14/translations/zh_CN/core-api/workqueue.html
https://events.static.linuxfound.org/sites/events/files/slides/Async%20execution%20with%20wqs.pdf

https://www.kernel.org/doc/Documentation/core-api/workqueue.rst

https://lwn.net/Articles/932431/

https://docs.kernel.org/core-api/workqueue.html

https://www.binss.me/blog/analysis-of-linux-workqueue/