---
title: Ftrace
categories: 
- Linux
tags:
- Linux Trace
---

## Ftrace原理
gcc编译时添加-pg编译选项。这个编译选项在函数开始处添加-bl指令

## Ftrace跟踪函数
```bash
cd /sys/kernel/debug/tracing/
echo 0 > tracing_on
echo 5 > max_graph_depth
echo function_graph > current_tracer
echo 1 > options/func_stack_trace
echo xxx > set_graph_function     ## set_ftrace_filter这个只能打印本函数没法打印子函数等，而set_graph_function是可以打印子函数，而且可以看到函数是不是在执行过程中被中断切出去了
echo 0 > trace  
echo 1 > tracing_on

./a.out 执行完程序
echo 0 > tracing_on
cat trace
```

### set_graph_function
If you want to trace only one function and all of its children,
you just have to echo its name into set_graph_function:


## Trace-cmd
https://man7.org/linux/man-pages/man1/trace-cmd-record.1.html
使用trace-cmd record -e workqueue:workqueue_queue_work查看workqueue在执行时cpu 100%的问题
https://community.frame.work/t/tracking-kworker-stuck-at-near-100-cpu-usage-with-ubuntu-22-04/23053?page=2

## Trace Event

### Trace Event添加

### 应用

## 参考
https://docs.kernel.org/trace/tracepoints.html
https://www.kernel.org/doc/Documentation/trace/ftrace.txt
https://www.cnblogs.com/arnoldlu/p/7211249.html
