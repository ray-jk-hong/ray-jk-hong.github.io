---
title: Numa
categories: 
- Linux
tags:
- Linux Numa
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
