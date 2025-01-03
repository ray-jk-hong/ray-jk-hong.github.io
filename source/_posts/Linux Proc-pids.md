---
title: Linux Cgroup
categories: 
- Linux
tags:
- Linux
---

/proc/pids目录下统计的信息: pids是进程的tgid号
- Name：进程的名字
- State：进程状态。I (Idle)等状态
- Threads：统计进程生成的线程数
- Uid：
- Gid：
- Cpus_allowed：可以执行的Cpu Mask
  进程的Cpu绑定，可以看taskset接口使用方法
- Cpus_allowed_list：可以执行的Cpu号
- Mems_allowed：这个再看一下，不太懂什么意思
- Mems_allowed_list：可以申请的内存Numa号
- voluntary_ctxt_switches
- nonvoluntary_ctxt_switches
