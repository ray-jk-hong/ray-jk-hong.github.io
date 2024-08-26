---
title: Linux Bootmem
categories: 
- Linux MM
tags:
- Linux MM
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
在内核启动日志中"Early memory node ranges"可以看到所有的memroy范围。包括上面所有的memblock_flags，但日志中没有打印类型。

### reserve范围确定
cat /sys/kernel/debug/memblock/reserve
这些地址是memblock中被屏蔽掉的地址，这些地址有些是不是固定的，例如代码段，dtb的地址都会在内核启动的时候调用memblock_reserve接口从memblock中排除掉。
可以看到reserve的地址范围，有些是包含在memory中的，有些是没有包含在memory中的？？为啥？

dts中设置的预留内存，也是memblock.reserve处理的，例如：
```bash
reserved-memory {
    reserved_mem@0 {
        no-map;
        ret = <0x0 0xAA00000 0x0 0xBB00000>;
    }
    reserved_mem@1 {
        ret = <0x0 0xCC00000 0x0 0xBB00000>;
    }
}
```
但这里的设置在cat /sys/kernel/debug/memblock/reserve里边有些能看到有些不能看到？？

DTS中reserved-memory处理流程如下：
```c
+-- start_kernel
    +-- setup_arch
        +-- arm64_memblock_init
            +-- early_init_fdt_scan_reserved_mem
                +-- fdt_scan_reserved_mem
                    +-- __reserved_mem_reserve_reg
                        +-- early_init_dt_reserve_memory
```
疑问：在启动完之后，reserved-memory {}中定义的很多段，其实在cat /sys/kernel/debug/memblock/reserve中没有显示
	（no-map的可以理解，这些段是保存在memory中，并被标记为MEMBLOCK_NOMAP的，所以在reserve中找不到也正常）
答案：没有标记no-map的段都是有的，标记位no-map的有些是找不到的。

疑问：标记位no-map的如果在device_type="memory"段找不到会怎么样？会报错吗？

疑问：标记位no-map的如果在device_type="memory"中找到了会怎么样？
答案：找到了就会在memory段中，把原来的段一份为2，，reserve-memory段重新生成一个memory区域并把他标记位MEMBLOCK_NOMAP，剩余的就是抠掉reserve-memory的。

## DTS中memreserve处理
一般都在dts最开始就定义，例如：
```c
/memreserve/ 0x40000000 0x01000000
```
## 参考
https://www.kernel.org/doc/html/v4.19/core-api/boot-time-mm.html
https://www.kernel.org/doc/gorman/

https://0xax.gitbooks.io/linux-insides/content/MM/linux-mm-1.html
