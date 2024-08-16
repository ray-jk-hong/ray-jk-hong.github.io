---
title: Workqueue
categories: 
- Linux
tags:
- Linux Workqueue
---

## alloc_workqueue WQ_UNBOUND的时候创建线程
在内核态ps -ef可以看到alloc_workqueue调用的时候创建的线程。例如名字是xxx_wq的时候, 5.15版本是[xxx_wq]，6.6x版本是显示[kworker/R-xxx_wq]。
这个线程是在init_rescuer的时候创建的。什么时候在这个worker里边执行，后面再看一下。
在queue_work的时候，真正执行的并不是上面的线程，一般都是在新创建的kworkerxxx执行。因为在alloc_workqueue的时候会选择条件一致的
struct worker_pool并在这个上面执行。


## work被中断抢占
1. work每次都执行在cpu0的时候被中断抢占，可以设置work的cpumask不让work在cpu0上执行
```bash
/sys/devices/virtual/workqueue# echo ffe >cpumask
```

## Workqueue Trace
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

