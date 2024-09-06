---
title: Linux Cgroup
categories: 
- Linux Cgroup
tags:
- Linux Cgroup
---

## Cgroup整体
Cgroup功能：以分组的形式对进程组使用系统资源的行为进行管理和控制。用户通过Cgroup对所有进程进行分组，再对该分组整体进行资源的分配和控制。
{% plantuml %}
rectangle   RootGroup   #lightgreen
rectangle   Group1      #lightgreen
rectangle   Group2      #lightgreen
rectangle   Group3      #lightgreen
rectangle   Group4      #lightgreen

RootGroup -down-> Group1
RootGroup -down-> Group2
Group2 -down-> Group3
Group2 -down-> Group4
{% endplantuml %}

Cgroup提供：
- Resource limitation：资源使用限制
- Prioritization：优先级控制
- Accrounting: 一些统计，例如内存统计
- Control：挂起进程，恢复执行进程
可以使用Cgroup进行如下事情：
- 隔离一个进程集合，限定他们所占用的资源，比如绑定的核限制
- 限制某个进程组的内存分配大小
- 限制某个进程组分配足够的贷款以及进行存储限制
- 限制某个进程组访问某些设备

## Cgroup子系统 
Cgroup每个子系统都代表一种类型的资源
- Cpu子系统：为每个进程组设置一个使用Cpu的权重值，以此来管理进程对CPU的访问
  cpu.cfs_period_us配置时间周期长度
  cpu.cfs_quota_us配置当前cgroup在设置的周期长度内所使用的Cpu时间数，和cpu.cfs_period_us一起配合起来设置Cpu的使用上限
- Cpuset子系统：对于多核CPU，可以设置进程组只能在指定的CPU核上运行，且可以指定进程组在指定的NUMA节点上申请内存
- Cpuacct子系统：只用于生成进程组内的进程对Cpu的使用报告
- Memory子系统：提供以页面为单位对内存的访问，可以设置进程组使用内存的上限，同时可以生成内存资源报告。
  (1) 限制Cgroup中所有进程使用的内存总量
  (2) 限制Cgroup中所有进程使用的物理内存+Swap交换总量
  (3) CONFIG_MEMCG_KMEM
  cgroup.event_control：用于eventfd的接口
  memory.usage_in_bytes：显示当前已用的内存
  memory.limit_in_bytes：设置/显示当前限制的内存额度
  memory.failcnt：显示内存使用量达到限制值的次数
  memory.max_usage_in_bytes：历史内存最大使用量
  memory.soft_limit_in_bytes：设置/显示当前限制的内存软额度
  memory.stat：显示当前Cgroup的内存使用情况
  memory.use_hierarchy：设置/显示是否将子Cgroup的内存使用情况统计到当前Cgroup里面
  memory.force_empty：触发系统立即尽可能的回收当前Cgroup中可以回收的内存
  memory.pressure_level：设置内存压力的通知事件，配合cgroup.event_control一起使用
  memory.swappiness：设置核显示当前的swappiness
  memory.move_charge_at_immigrate：设置当前进程移动到其他Cgroup中时，它所占用的内存是否也随着移动过去
  memory.oom_control：设置/显示oom controls相关配置
  memory.numa_stat：显示numa相关的内存  
- Blkio子系统：限制进程组对块设备的输入输出。与Cpu子系统类似，为每个进程组设置权重控制进程组对块设备的I/O操作时间，也可以是限制I/O带宽和IOPS
- Device子系统：限制进程组对设备的访问，即允许或者禁止进程组对某设备的访问
- Freezer子系统：可以控制整个进程组中的所有进程的挂起
- Net-cls子系统：提供网络带宽的访问限制
- Pids子系统：限制Cgroup及其所有子孙Cgroup里边能创建的总的进程数量。进程是指通过fork或者clone函数创建的进程和线程。
  /sys/fs/cgroup/pids
  某个进程fork了一个子进程，子进程默认与父进程处于同一个Cgroup中

## Cgroup Oom行为配置
如果panic_on_oom参数为2，则OOM会导致kernel panic，建议使用1，只影响当前进程
echo 1 > /proc/sys/vm/panic_on_oom

## 软件架构
```c
struct task_struct {
    struct css_set *cgroups;
    struct list_head cg_list;
}
```

## 进程统计与Cgroup统计差异问题
1. page cache，例如文件系统写入也会统计到Cgroup。例如往tmpfs写入的文件都是内存申请的，会统计到Cgroup中，但不会统计到进程。
   疑问：怎么知道哪些是盘是ramfs?
2. 所以有时看到Cgroup显示内存已经超过limit报了oom，但oom打印的当前cgroup所有的进程的内存大小加起来却很小。
   oom时打印的内存大小，可以看"Linux 进程内存统计"这篇

## 进程内存统计
进程上下文申请的内存要统计到Cgroup中需要在kmalloc的flag中添加__GFP_ACCOUNT

