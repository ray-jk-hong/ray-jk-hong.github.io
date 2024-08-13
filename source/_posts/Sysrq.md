---
title: Sysrq
categories: 
- Linux
tags:
- Linux Sysrq
---

## 代码路径
Sysrq的代码路径在/drivers/tty/sysrq.c
cmd对应的处理函数是static const struct sysrq_key_op *sysrq_key_table[62] = {};
可以往/proc/sysrq-trigger中写入对应的cmd来触发某些事件
例如往/proc/sysrq-trigger中写入c可以导致系统挂死：echo c > /proc/sysrq-trigger

## 用法
### 系统挂死
echo c > /proc/sysrq-trigger
这个命令输入之后，系统就会挂死。

### 打印所有cpu的调用栈
echo l > /proc/sysrq-trigger
```c 
[94891.950482] CPU: 5 PID: 6174 Comm: bash Not tainted 6.9.8-orbstack-00170-g7b4100b7ced4 #1
[94891.950491] Hardware name: orbstack,virt (DT)
[94891.950494] Call trace:
[94891.950496]  dump_backtrace+0xe8/0x110
[94891.950509]  show_stack+0x1c/0x30
[94891.950512]  dump_stack_lvl+0x38/0x78
[94891.950522]  dump_stack+0x14/0x20
[94891.950525]  nmi_cpu_backtrace+0xe0/0x138
[94891.950573]  nmi_trigger_cpumask_backtrace+0x90/0x180
[94891.950576]  arch_trigger_cpumask_backtrace+0x1c/0x30
[94891.950578]  sysrq_handle_showallcpus+0x20/0x30
[94891.950585]  __handle_sysrq+0x14c/0x158
[94891.950586]  write_sysrq_trigger+0xec/0x100
[94891.950588]  proc_reg_write+0x98/0x110
[94891.950596]  vfs_write+0x124/0x378
[94891.950601]  ksys_write+0x78/0xe8
[94891.950603]  __arm64_sys_write+0x20/0x30
[94891.950605]  do_el0_svc+0x90/0xe8
[94891.950607]  el0_svc+0x24/0x50
[94891.950714]  el0t_64_sync_handler+0x7c/0xf0
[94891.950717]  el0t_64_sync+0x14c/0x150
[94891.950722] Sending NMI from CPU 5 to CPUs 0-4,6-9:
[94891.950743] NMI backtrace for cpu 0
[94891.951110] CPU: 0 PID: 0 Comm: swapper/0 Not tainted 6.9.8-orbstack-00170-g7b4100b7ced4 #1
[94891.951114] Hardware name: orbstack,virt (DT)
[94891.951117] pstate: 61400005 (nZCv daif +PAN -UAO -TCO +DIT -SSBS BTYPE=--)
[94891.951120] pc : do_idle+0x100/0x2d8
[94891.951127] lr : do_idle+0x100/0x2d8
[94891.951133] sp : ffffc86062cf3dd0
[94891.951135] x29: ffffc86062cf3de0 x28: 0000000040000000 x27: ffffc86062cfa000
[94891.951140] x26: ffffc86062cfa748 x25: 0000000000000001 x24: ffffc86062cce000
[94891.951143] x23: 0000000000000000 x22: ffffc86062d13780 x21: 00000003ffe00000
[94891.951146] x20: 0000000000000000 x19: 0000000000000000 x18: 0000000000000005
[94891.951149] x17: 00000000000000a2 x16: 0000000000000082 x15: 0000000000000010
[94891.951151] x14: 0000000000000010 x13: ffffc86062cd38c0 x12: ffffffffffffffe1
[94891.951154] x11: 000000000016080c x10: 0000000000000001 x9 : ffffc86062ccc470
[94891.951157] x8 : 4000000000000000 x7 : 0000000000000000 x6 : 0000000000000000
[94891.951159] x5 : 00002c74284a5c70 x4 : ffffc86062cf3c98 x3 : ffff151c4f0adec0
[94891.951162] x2 : ffffc86062cf3d74 x1 : 0000000000000000 x0 : 00000000ffffffff
[94891.951165] Call trace:
[94891.951166]  do_idle+0x100/0x2d8
[94891.951168]  cpu_startup_entry+0x38/0x48
[94891.951170]  rest_init+0xc8/0xd0
[94891.951173]  start_kernel+0x264/0x2b0
[94891.951178]  __primary_switched+0x80/0x90
```

### 将ftrace的buffer打印出来
```bash
echo z > /proc/sysrq-trigger
```
输入完上面的命令之后，可以敲dmesg可以看到ftrace的buff里边的内容全部打印到内核日志中。

### 显示内存信息
```
echo m > /proc/sysrq-trigger
```
之前敲dmesg可以看到打印的内存信息。
