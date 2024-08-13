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
