---
title: Linux Cgroup
categories: 
- Linux Cgroup
tags:
- Linux Cgroup
---

/proc/pids目录下统计的信息: pids是进程的tgid号
- Name：进程的名字
- State：进程状态。I (Idle)等状态
- Threads：统计进程生成的线程数
- Uid：
- Gid：
- Cpus_allowed：可以执行的Cpu Mask
- Cpus_allowed_list：可以自行的Cpu号
- Mems_allowed：这个再看一下，不太懂什么意思
- Mems_allowed_list：可以申请的内存Numa号
- voluntary_ctxt_switches
- nonvoluntary_ctxt_switches