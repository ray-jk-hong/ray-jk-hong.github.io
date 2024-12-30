---
title: Linux Proc-stack
categories: 
- Linux
tags:
- Linux
---

## /proc/[pid]/stack
显示当前进程的内核调用栈信息，只有内核编译时打开了CONFIG_STACKTRACE编译选项，才会生成这个文件

```
> cat /proc/1751/stack 
[<ffffffffa4112496>] futex_wait_queue_me+0xc6/0x130
[<ffffffffa411323b>] futex_wait+0x17b/0x280
[<ffffffffa4114fa6>] do_futex+0x106/0x5a0
[<ffffffffa41154c0>] SyS_futex+0x80/0x190
[<ffffffffa4793f92>] system_call_fastpath+0x25/0x2a
[<ffffffffffffffff>] 0xffffffffffffffff
```
