---
title: Workqueue
---

## 查看workqueue在干什么
1) 通过cat查看workqueue的调用栈
  cat /proc/<kworker_pid>/stack
2) trace-cmd方式查看
  trace-cmd record -e workqueue:workqueue_queue_work
  trace-cmd report > trace.log
  You can get list of the most frequently queued to least frequently queued workqueue functions with the following:
    grep -o -e "function=[_a-zA-Z_][_a-zA-Z0-9]*" trace.log|sort|uniq -c |sort -rn

## 接口使用注意
#### cancle_work_sync
如果work的回调函数中有等待信号量等操作的时候，直接调用destroy_workqueue是会有报错的。
正确的做法是
1) 唤醒work回调函数的信号量等待
2) 调用cancel_work_sync等待work结束
3) 调用destroy_workqueue销毁workqueue
