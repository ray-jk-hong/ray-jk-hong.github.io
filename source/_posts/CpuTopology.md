---
title: CpuTopology
categories: 
- Aarch64
tags:
- Aarch64
---

## lscpu查看cpu信息
```bash
[xxx@cs ~]# lscpu
Architecture:          x86_64
CPU op-mode(s):        32-bit, 64-bit
Byte Order:            Little Endian
CPU(s):                24
On-line CPU(s) list:   0-23
Thread(s) per core:    2
Core(s) per socket:    6
Socket(s):             2
NUMA node(s):          2
Vendor ID:             GenuineIntel
CPU family:            6
Model:                 62
Stepping:              4
CPU MHz:               2100.118
BogoMIPS:              4199.92
Virtualization:        VT-x
L1d cache:             32K
L1i cache:             32K
L2 cache:              256K
L3 cache:              15360K
NUMA node0 CPU(s):     0,2,4,6,8,10,12,14,16,18,20,22
NUMA node1 CPU(s):     1,3,5,7,9,11,13,15,17,19,21,23
```
能看到的信息：
- Architecture：x86或者aarch64等
- Byte Order: 大小端
- CPU(s)：总的CPU核数
- On-line CPU(s)：上线的CPU核数，PG版本可能这个核数会少于总的CPU核数
- Socket(s), Core(s), Thread(s)的意义：
- BogoMIPS：
- CPU MHz: CPU频率
- Virtulization：支持的虚拟化特性
- L1d/L1i/L2/L3 Cache：每层的Cache内存大小
- Numa nodex：有几个Numa，且每个Numa对应的CPU核是哪些

## 启动
```
cpu-map {
  socket0 {
    cluster0 {
      core0 {
        cpu = <&cpu0>;
      };
      core1 {
        cpu = <&cpu1>;
      };      
    };
    cluster1 {
      cpu = <&cpu2>;
    };
  };
};
```
解析这段DTS定义的函数：drivers/base/arch_topology.c文件的parse_dt_topology函数

## 参考

Documentation\cputopology.txt
