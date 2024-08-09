---
title: Workqueue
---

## Workqueue问题定位
### Work消耗太多CPU Cycles(Top命令能看到)

Worker线程通过ps命令打印如下：
‘’‘
root      5671  0.0  0.0      0     0 ?        S    12:07   0:00 [kworker/0:1]
root      5672  0.0  0.0      0     0 ?        S    12:07   0:00 [kworker/1:2]
root      5673  0.0  0.0      0     0 ?        S    12:12   0:00 [kworker/0:0]
root      5674  0.0  0.0      0     0 ?        S    12:13   0:00 [kworker/1:0]
’‘’

以下几种可能
1) Work切换频繁
‘’‘bash
$echo workqueue:workqueue_queue_work > /sys/kernel/tracing/set_event
$cat /sys/kernel/tracing/trace_pipe > out.txt                    
(wait a few secs)                                                 
^C
’‘’
这样可以看到所有的Work执行的情况。

'''bash
<idle>-0       [005] dNs.. 25598.686997: workqueue_queue_work: work struct=00000000b8691ef7 function=nf_flow_offload_work_gc workqueue=events_power_efficient req_cpu=24 cpu=-1
'''
function=‘Work函数名’就是Work的函数。

2) Work一次执行消耗太多
使用如下方式查看kwork的调用栈。
'''bash
$cat /proc/THE_OFFENDING_KWORKER/stack
'''bash
THE_OFFENDING_KWORKER就是Worker线程的pid。


## 接口使用注意
#### cancle_work_sync
如果work的回调函数中有等待信号量等操作的时候，直接调用destroy_workqueue是会有报错的。
正确的做法是
1) 唤醒work回调函数的信号量等待
2) 调用cancel_work_sync等待work结束
3) 调用destroy_workqueue销毁workqueue
