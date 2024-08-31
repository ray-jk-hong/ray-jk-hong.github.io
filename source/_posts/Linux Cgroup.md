---
title: Cgroup
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
- Cpuset子系统：对于多核CPU，可以设置进程组只能在指定的CPU核上运行，且可以指定进程组在指定的NUMA节点上申请内存
- Cpuacct子系统：只用于生成进程组内的进程对Cpu的使用报告
- Memory子系统：提供以页面为单位对内存的访问，可以设置进程组使用内存的上限，同时可以生成内存资源报告。
- Blkio子系统：限制进程组对块设备的输入输出。与Cpu子系统类似，为每个进程组设置权重控制进程组对块设备的I/O操作时间，也可以是限制I/O带宽和IOPS
- Device子系统：限制进程组对设备的访问，即允许或者禁止进程组对某设备的访问
- Freezer子系统：可以控制整个进程组中的所有进程的挂起
- Net-cls子系统：提供网络带宽的访问限制

## 相关结构体
```c
struct task_struct {
    struct css_set *cgroups;
    struct list_head cg_list;
}
```

## 在线显示

{% plantuml %}
    AAA->BBB : hello
{% endplantuml %}


## preview用
```plantuml
AAA->BBB : hello
```
