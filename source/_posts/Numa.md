---
title: Numa
categories: 
- Linux MM
tags:
- Linux MM
---

## Dts定义NUMA
```bash
memory10@numa10 {
  device_type = "memory";
  reg = <0xAA 0xBB0000 0x0 0x10000000>; // 1GB
  numa-node-id = <10>;
  hotpluggable;
};
memory11@numa11 {
  device_type = "memory";
  reg = <0xAA 0xBB0000 0x0 0x20000000>; // 2GB
  numa-node-id = <101>;
  hotpluggable;
};
```

cat /proc/buddyinfo
```bash
Node 0, zone      DMA    264    121     36     17      8      4      4     15     25     12    100
Node 0, zone   Normal     75    105     64     10      1      1      0      2      1      0      0
Node 31, zone  Movable      0      0      0      0      0      0      0      0      0      0      7
Node 32, zone  Movable     13      3      0     12      8      5      2      1      0      0    253
Node 33, zone  Movable      0      0      0      0      0      0      0      0      0      0    269
Node 34, zone  Movable      0      0      0      0      0      0      0      0      0      0    269
Node 35, zone  Movable      0      0      0      0      0      0      0      0      0      0    269
Node 36, zone  Movable      0      0      0      0      0      0      0      0      0      0    269
Node 37, zone  Movable      0      0      0      0      0      0      0      0      0      0    269
Node 38, zone  Movable      0      0      0      0      0      0      0      0      0      0    269
Node 39, zone  Movable      0      0      0      0      0      0      0      0      0      0    269
Node 40, zone  Movable      0      0      0      0      0      0      0      0      0      0     35
```
