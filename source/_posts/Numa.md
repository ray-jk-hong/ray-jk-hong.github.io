---
title: Numa
categories: 
- Linux MM
tags:
- Linux MM
---

## Dts定义NUMA
```bash
memory0@numa0 {
  device_type = "memory";
  reg = <0x0 0x0AA00000 0x0 0xBB00000>,
        <0x0 0x0CC00000 0x0 0xBB00000>,
        ...;
  numa-node-id = <0>;
}
memory10@numa10 {
  device_type = "memory";
  reg = <0xAA 0xBB0000 0x0 0x10000000>; // 1GB
  numa-node-id = <10>;
  hotpluggable; // 在/proc/buddyinfo中显示为movable
};
memory11@numa11 {
  device_type = "memory";
  reg = <0xAA 0xBB0000 0x0 0x20000000>; // 2GB
  numa-node-id = <101>;
  hotpluggable;  // 在/proc/buddyinfo中显示为movable
};
```

cat /proc/buddyinfo
```bash
Node 0, zone      DMA    264    121     36     17      8      4      4     15     25     12    100
Node 0, zone   Normal     75    105     64     10      1      1      0      2      1      0      0
Node 10, zone  Movable      0      0      0      0      0      0      0      0      0      0      7
Node 11, zone  Movable     13      3      0     12      8      5      2      1      0      0    253
```

相关的宏
CONFIG_NODES_SHIFT

