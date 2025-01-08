---
title: Linux IO-应用
categories: 
- Linux
tags:
- Linux
---

## Select

## Epoll

https://rammuking.tistory.com/entry/Epoll%EC%9D%98-%EA%B8%B0%EC%B4%88-%EA%B0%9C%EB%85%90-%EB%B0%8F-%EC%82%AC%EC%9A%A9-%EB%B0%A9%EB%B2%95


https://fd3kyt.github.io/posts/implementation-of-epoll/

https://zhuanlan.zhihu.com/p/721189872

https://github.com/libevent/libevent/blob/master/epoll.c

epoll和poll的行为在内核态差不多。POLLIN, EPOLLIN这些是否兼容

https://stackoverflow.com/questions/18103093/how-does-a-socket-event-get-propagated-converted-to-epoll/27143672#27143672


## Poll

https://www.cnblogs.com/gulan-zmc/p/12229159.html

https://www.makelinux.net/ldd3/chp-6-sect-3.shtml

## IoUring

## 内核态实现
1. 进程wait初始化
   1) 进程结构体中定义一个wakt_queue_head_t inq, wait_queue_head_t outq
   2) 在进程初始化时inti_waitqueue_head(&intq), inti_waitqueue_head(&outq)初始化
2. 实现file_operations->poll函数
```
   static unsigned int scull_p_poll(struct file *filp, poll_table *wait)
  {
      struct scull_pipe *dev = filp->private_data;
      unsigned int mask = 0;
      /*
      * The buffer is circular; it is considered full
      * if "wp" is right behind "rp" and empty if the
      * two are equal.
      */
      down(&dev->sem);
      poll_wait(filp, &dev->inq, wait);
      poll_wait(filp, &dev->outq, wait);
      if (dev->rp != dev->wp)
          mask |= POLLIN | POLLRDNORM; /* readable */
      if (spacefree(dev))
          mask |= POLLOUT | POLLWRNORM; /* writable */
      up(&dev->sem);
      return mask;
  }
```
	在poll函数中、添加poll_wait和返回的POLLIN等mask
	poll_wait(filep, &head_wait, wait), 这里wait是poll_table*类型
	poll_wait() 本身并不引起阻塞，只是将等待队列头部添加到 poll_table 中，是让唤醒等待队列可以唤醒因 select() 而睡眠的进程

3. 在write, read函数中实现等待
```
static ssize_t xx_write(struct file *filp,
    const char __user *buf, size_t size, loff_t *ppos)
{
    ...
    /* initial element in wait queue, priv = current */
    DECLARE_WAITQUEUE(wait, current);
    /* insert element into wait queue */
    add_wait_queue(&xxx_wait, &wait);

    do {
        avail = device_writable(...);
        if (avail < 0) {
            if (filp->f_flags & O_NONBLOCK) {
                ret = -EAGAIN;
                goto out;
            }
            set_current_state(TASK_INTERRUPTIBLE);
            schedule();

            /* be waken up by signal */
            if (signal_pending(current)) {
                ret = -ERESTARTSYS;
                goto out;
            }
        }
    } while(avail < 0);

    device_write(...);

out:
    remove_wait_queue(&xxx_wait, &wait);
    set_current_state(TASK_RUNNING);

    return ret;
}
```

## 参考
https://doc.embedfire.com/linux/imx6/linux_base/zh/latest/system_programing/socket_io/socket_io.html
