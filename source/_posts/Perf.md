---
title: Perf
categories: 
- Linux
tags:
- Linux Perf
---
## perf命令
### stat
记录给定条件下事件的发生次数
参数：
- -p    指定待分析进程的 pid（可以是多个，用,分隔列表）
- -t     指定待分析线程的 tid（可以是多个，用,分隔列表）
- -a    从所有 CPU 收集系统数据
- -d    -d：打印更详细的信息，可重复 3 次；
追加显示L1 和 LLC data cache
- -d -d 追加显示dTLB 和 iTLB events
- -d -d -d    追加 prefetch events
- -r     重复运行命令 n 次，打印平均值。n 设为 0 时无限循环打印
- -c    只统计指定 CPU 列表的数据，如：0,1,3或1-2
- -A   与-a选项联用，不要将 CPU 计数聚合

输出结果分析：
- Task-clock-msecs：CPU 利用率，该值高，说明程序的多数时间花费在 CPU 计算上而非 IO。
- Context-switches：进程切换次数，记录了程序运行过程中发生了多少次进程切换，一般来说，频繁的进程切换有可能只是CPU正常调度下一个任务，也有可能是一些性能问题的外在表现，导致的原因可能是锁竞争，IO阻塞，硬件中断等。如果可能，频繁的进程切换是应该避免的。
- Cache-misses：程序运行过程中总体的 cache 利用情况，对于性能敏感的程序来说，需要避免出现cache miss的情况。解决方法是对于程序频繁访问的热数据可以集中紧凑存储、程序不去大跨度离散的访问内存等等。
- CPU-migrations：表示进程 t1 运行过程中发生了多少次 CPU 迁移，即被调度器从一个 CPU 转移到另外一个 CPU 上运行。
- Cycles：处理器时钟，一条机器指令可能需要多个 cycles。
- Instructions: 机器指令数目。
- branches：为分支数量。
- branch-misses：为分支预测时失败的数量。

### record
#### lock
内核锁性能
需要编译选项的支持：CONFIG_LOCKDEP、CONFIG_LOCK_STAT。
CONFIG_LOCKDEP ：defines acquired and release events.
CONFIG_LOCK_STAT ：defines contended and acquired lock events.
-i     输入文件
-k    sorting key，默认为acquired，还可以按contended、wait_total、wait_max和wait_min来

执行命令：
1. perf lock record ./xxx
2. perf lock report

结果分析：
- Name      内核锁的名字
- aquired    该锁被直接获得的次数，因为没有其它内核路径占用该锁，此时不用等待。
- contended       该锁等待后获得的次数，此时被其它内核路径占用，需要等待
- total wait 为了获得该锁，总共的等待时间。
- max wait 为了获得该锁，最大的等待时间
- min wait  为了获得该锁，最小的等待时间。

#### kmem
slab分配器性能分析
执行命令：
1. perf kmem record ./xxx
2. perf kmem stat --caller --alloc -l 20
结果分析：
- Callsite    内核代码中调用kmalloc和kfree的地方
- Total_alloc/Per       总共分配的内存大小，平均每次分配的内存大小
- Total_req/Per  总共请求的内存大小，平均每次请求的内存大小
- Hit   调用的次数
- Ping-pong      kmalloc和kfree不被同一个CPU执行时的次数，这会导致cache效率降低。
- Frag 碎片所占的百分比，碎片 = 分配的内存 - 请求的内存，这部分是浪费的。
- Alloc Ptr 有使用--alloc选项，还会看到此列，即所分配内存的地址(案例未使用)

#### sched
执行命令记录1秒钟的调度事件：
```bash
perf sched record -- sleep 1
```
执行以下命令可以对生成的recored进行分析：
1. perf sched map
2. perf sched timehist
3. 
### top
主要用于实时剖析各个函数在某个性能事件上的热度。利用perf top，能够直观地观察到当前的热点函数，并利用工具中内置的annotate功能，进一步查找热点指令

### diff

### timechart

由于perf timechart只记录线程粒度的信息，无法替代火焰图在函数级别上分析，需要结合火焰图一
起使用

### probe
可以在程序中添加或者删除动态追踪点，即自定义事件。
例如给malloc/free等添加事件来追踪内存泄露情况：
```bash
perf probe --exec=/lib64/libc-2.17.so --add malloc
或者
# perf stat -e probe_libc:free -e probe_libc:malloc -ag -p $(pgrep $process_name$) sleep 4
```

probe也可以在（内核）函数的某一行添加事件进行追踪：
例如在schedule函数的12行处增加一个探测点
```bash
perf probe -a schedule:12
```

若需要对自定义函数进行跟踪，和上文相似，假设我们定义了一个名为loop的函数
```bash
#perf probe -x /root/code/test-perf-probe.o "--add=loop"
```
执行之后会显示：
```bash
Added new event: probe_test_perf_probe:loop (on loop in /root/code/test_perf_probe.o)
```
然后就可以执行perf工具记录：
```bash
perf record -e probe_test_perf_probe:loop -aR sleep 1
```

## 火焰图

## 参考
https://www.brendangregg.com/perf.html
