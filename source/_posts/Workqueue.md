---
title: Workqueue
categories: 
- Linux
tags:
- Linux Workqueue
---

## alloc_queue创建流程
在调用alloc_workqueue的时候创建的结构体如下图：
![TCR寄存器](/images/Workqueue/Workqueue创建流程.drawio.svg)

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

## Workqueue指定执行的cpu
1. 可以在调用alloc_workqueue的时候传入WQ_SYSFS
    例如：alloc_workqueue(xxx, WQ_SYSFS)，xxx是workqueue的名字
2. 修改/sys/devices/virtual/workqueue/xxx/cpumask。
    例如cpu个数是10个则/sys/devices/virtual/workqueue/xxx/cpumask是0x3FF。
    如果想把cpu0给mask掉不让workqueue执行到cpu0，就写入0x3FE即可。

原理是写入之后，workqueue创建worker的流程会重新执行，并将worker_thread线程bind到对应的cpu上。

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

