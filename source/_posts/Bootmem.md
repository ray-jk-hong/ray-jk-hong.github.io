---
title: USB驱动
categories: 
- Linux Driver
tags:
- Linux Driver
---

## 物理地址的范围
### memblodk物理地址范围
通过DTS定义确定, 例如dts的定义如下：
```bash
memory0@numa0 {
  device_type = "memory";
  reg = <0x0 0x0AA00000 0x0 0xBB00000>,
        <0x0 0x0BB00000 0x0 0xBB00000>,
        ...;
  numa-node-id = <0>;
}
memory10@numa10 {
  device_type = "memory";
  reg = <0xAA 0xCC0000 0x0 0xBB00000>; 
  numa-node-id = <10>;
  hotpluggable; // 在/proc/buddyinfo中显示为movable
};
memory11@numa11 {
  device_type = "memory";
  reg = <0xBB 0xDD0000 0x0 0xBB00000>; 
  numa-node-id = <101>;
  hotpluggable;  // 在/proc/buddyinfo中显示为movable
};
```
cat /sys/kernel/debug/memblock/memory：范围和dts范围定义一致
```bash
   0: 0x000000000AA00000..0x000000000BB00000
   1: 0x000000000BB00000..0x000000000BB00000
   2: 0x0000000AACC00000..0x000000000BB00000
   3: 0x0000000AADD00000..0x000000000BB00000
```
读取dts并把memoy加到memblock流程：
```c
+-- start_kernel
   +-- setup_arch
      +-- setup_machine_fdt
         +-- early_init_dt_scan
            +-- early_init_dt_scan_nodes
               +-- early_init_dt_scan_memory
```
early_init_dt_scan_memory函数中会scan dts并找到device_type为memory的节点并把地址加到memblock中。
memblock.memory的类型如下：
```c
enum memblock_flags {
	MEMBLOCK_NONE		= 0x0,	/* No special request */
	MEMBLOCK_HOTPLUG	= 0x1,	/* hotpluggable region */
	MEMBLOCK_MIRROR		= 0x2,	/* mirrored region */
	MEMBLOCK_NOMAP		= 0x4,	/* don't add to kernel direct mapping */
	MEMBLOCK_DRIVER_MANAGED = 0x8,	/* always detected via a driver */
	MEMBLOCK_RSRV_NOINIT	= 0x10,	/* don't initialize struct pages */
};
```
可惜在/sys/kernel/debug/memblock/memory并没有打印memblock.meory的类型。

### reserve范围确定
cat /sys/kernel/debug/memblock/reserve
这些地址是memblock中被屏蔽掉的地址，这些地址有些是不是固定的，例如代码段，dtb的地址都会在内核启动的时候调用memblock_reserve接口从memblock中排除掉。
可以看到reserve的地址是在memory包含在里边的，就是说reserve地址表示从memory地址范围中扣出去的部分。

## memblock查询
```bash
root@ubuntu-24-04:/sys/kernel/debug/memblock# cat memory 
   0: 0x0000000080000000..0x00000001d54fffff    0 NONE
root@ubuntu-24-04:/sys/kernel/debug/memblock# cat reserved 
   0: 0x0000000080010000..0x000000008266ffff    0 NONE
   1: 0x00000000fbfff000..0x00000000ffffefff    0 NONE
   2: 0x00000001ce3a3000..0x00000001d47fffff    0 NONE
   3: 0x00000001d483fbc0..0x00000001d483ffc7    0 NONE
   4: 0x00000001d4842000..0x00000001d48426c7    0 NONE
   5: 0x00000001d4842700..0x00000001d484275f    0 NONE
   6: 0x00000001d4842780..0x00000001d4842907    0 NONE
   7: 0x00000001d4842940..0x00000001d4842b87    0 NONE
   8: 0x00000001d4842bc0..0x00000001d4842cef    0 NONE
   9: 0x00000001d4842d00..0x00000001d4842d4f    0 NONE
  10: 0x00000001d4842d80..0x00000001d4842ea6    0 NONE
  11: 0x00000001d4842ec0..0x00000001d4842fe6    0 NONE
  12: 0x00000001d4843000..0x00000001d4846027    0 NONE
  13: 0x00000001d4846040..0x00000001d4846047    0 NONE
  14: 0x00000001d4846080..0x00000001d4846087    0 NONE
  15: 0x00000001d48460c0..0x00000001d48467b7    0 NONE
  16: 0x00000001d48467c0..0x00000001d54fffff    0 NONE
  17: 0x00000003ffe00000..0x00000003ffe00f1b    0 NONE
```
17超过了memblock/memory的范围？

## 参考
https://www.kernel.org/doc/html/v4.19/core-api/boot-time-mm.html
https://www.kernel.org/doc/gorman/
