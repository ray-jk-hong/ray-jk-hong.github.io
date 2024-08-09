---
title: Kprobe
---

## 内核态注册
https://docs.kernel.org/trace/kprobes.html

## 用户态使用Kprobe探测内核函数
### 开启关闭Kprobe
1) 开启：echo 1 > /sys/kernel/debug/tracing/events/kprobes/enable
2) 关闭：echo 0 > /sys/kernel/debug/tracing/events/kprobes/enable
### 设置Kprobe探测点
1) 返回值打印：
  echo 'r 函数名 ret=$retval' > /sys/kernel/debug/tracing/kprobe_events
2) 入参打印：
  x86平台使用%ax, %bx, %cx, %dx表示第0~3个参数。arm平台使用x0, x1, x2, x3来表示第0~3个参数。Linux4.x版本之后，应该都可以使用arg0, arg1等方式表示。
  (1) 函数的参数直接打印值：
     X86打印第一个第二个参数:echo 'p function_name a=%ax:s32 b=%bx:u64' > /sys/kernel/debug/tracing/kprobe_events
     ARM打印第一个第二个参数:echo 'p function_name a=%x0:x64 b=%x1' > /sys/kernel/debug/tracing/kprobe_events
     这里参数:后面s32, u64, x64等都表示的是需要打印的类型。x64表示按照16禁止打印，数据位宽就是64bit（8字节）
  (2) 函数参数是指针，打印指针里边的内容：
    echo 'p function_name +0(%x1):x64 +8(%x1):x64 +16(%x1):x64' > /sys/kernel/debug/tracing/kprobe_events
    +0(指针)表示解引用指针。按照上面的打印，第二个参数就是指针，+0(%x1):x64就是表示第二个参数指针偏移0并按照x64方式打印。
### 查看结果
cat /sys/kernel/debug/tracing/trace

### 清空结果
echo > /sys/kernel/debug/tracing/trace

### 查看调用栈
echo 1 > /sys/kernel/debug/tracing/options/stacktrace

### 过滤入参
echo 'arg2==期望的值' > /sys/kernel/debug/tracing/events/kprobes/p_函数名_0/filter

### 挂死打印
echo 1 > /proc/sys/kernel/ftrace_dump_on_oops
