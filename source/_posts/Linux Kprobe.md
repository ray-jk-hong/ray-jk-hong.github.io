---
title: Kprobe
categories: 
- Linux Trace
tags:
- Linux Trace
---

## 内核态注册
https://docs.kernel.org/trace/kprobes.html

## 用户态使用Kprobe探测内核函数
### 开启关闭Kprobe
1. 开启：echo 1 > /sys/kernel/debug/tracing/events/kprobes/enable
2. 关闭：echo 0 > /sys/kernel/debug/tracing/events/kprobes/enable

### 设置Kprobe探测点
1. 返回值打印：
```bash
echo 'r 函数名 ret=$retval' > /sys/kernel/debug/tracing/kprobe_events
```
2. 入参打印:

x86平台使用%ax, %bx, %cx, %dx表示第0-3个参数。
arm平台使用x0, x1, x2, x3来表示第0-3个参数。
Linux4.x版本之后，应该都可以使用arg0, arg1等方式表示。

2.1 函数的参数直接打印值：

X86打印第一个第二个参数:
```bash
echo 'p function_name a=%ax:s32 b=%bx:u64' > /sys/kernel/debug/tracing/kprobe_events
```
ARM打印第一个和第二个参数:
```bash
echo 'p function_name a=%x0:x64 b=%x1' > /sys/kernel/debug/tracing/kprobe_events
```
这里参数:后面s32, u64, x64等都表示的是需要打印的类型。x64表示按照16进制打印，数据位宽就是64bit（8字节）

2.2 函数参数是指针，打印指针里边的内容：
```bash
echo 'p function_name +0(%x1):x64 +8(%x1):x64 +16(%x1):x64' > /sys/kernel/debug/tracing/kprobe_events
```
+0(指针)表示解引用指针。按照上面的打印，第二个参数就是指针，+0(%x1):x64就是表示第二个参数指针偏移0并按照x64方式打印。

### 查看结果
```bash
cat /sys/kernel/debug/tracing/trace
```

### 清空结果
```bash
echo > /sys/kernel/debug/tracing/trace
```

### 查看调用栈
```bash
echo 1 > /sys/kernel/debug/tracing/options/stacktrace
```

### 过滤入参
```bash
echo 'arg2==期望的值' > /sys/kernel/debug/tracing/events/kprobes/p_函数名_0/filter
```

### 挂死打印
当内核panic的时候ftrace_dump函数会将trace缓冲区里边的内容打印到内核日志中。
```bash
echo 1 > /proc/sys/kernel/ftrace_dump_on_oops
```

例如：调用栈显示某个函数有挂死现象，则可以按照上述方式打开并跟踪参数情况，可以追踪哪些参数传到函数中导致的异常


