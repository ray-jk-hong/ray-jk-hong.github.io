---
title: Linux 进程调度相关用户态接口
categories: 
- Linux
tags:
- Linux
---


## taskset命令：让程序执行在某些Cpu上
控制进程可执行的Cpu id。
https://blog.csdn.net/test1280/article/details/87991302

## 执行的Cpu设置
例如让程序只执行在Cpu0和Cpu2执行
```c
#define _GNU_SOURCE
#include <sched.h>

cpu_set_t  mask;
CPU_ZERO(&mask);
CPU_SET(0, &mask);
CPU_SET(2, &mask);
int result = sched_setaffinity(0, sizeof(mask), &mask);
```

## 设置进程优先级


