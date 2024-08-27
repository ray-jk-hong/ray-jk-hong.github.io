---
title: Linux Bootmem
categories: 
- Linux MM
tags:
- Linux MM
---

## 物理地址的范围
memblock模块在设备启动的时候，确定可用的物理地址范围，在page初始化之前提供内存分配和释放功能。
相关结构体和全局变量如下：
{% plantuml %}
allowmixing

enum memblock_flags {
	MEMBLOCK_NONE		= 0x0,	/* No special request */
	MEMBLOCK_HOTPLUG	= 0x1,	/* hotpluggable region */
	MEMBLOCK_MIRROR		= 0x2,	/* mirrored region */
	MEMBLOCK_NOMAP		= 0x4,	/* don't add to kernel direct mapping */
	MEMBLOCK_DRIVER_MANAGED = 0x8,	/* always detected via a driver */
	MEMBLOCK_RSRV_NOINIT	= 0x10,	/* don't initialize struct pages */
}

class memblock_region {
    phys_addr_t base;
    phys_addr_t size;
    enum memblock_flags flags;
#ifdef CONFIG_NUMA
	int nid;
#endif
}
memblock_flags -up-> memblock_region
memblock::memory -> memblock_type
memblock_type::regions -> memblock_region

class memblock_type {
	unsigned long cnt;
	unsigned long max;
	phys_addr_t total_size;
	struct memblock_region *regions;
	char *name;
}

class memblock {
    bool bottom_up;
    phys_addr_t current_limit;
    struct memblock_type memory;
    struct memblock_type reserved;
}

rectangle memory #lightgreen
rectangle reserved #lightgreen
rectangle phymem

memblock -down-> memory 
memblock -down-> reserved
memblock -down-> phymem

{% endplantuml %}

上图中memory, reserved是常驻的，physmem是只有定义了CONFIG_HAVE_MEMBLOCK_PHYS_MAP才有

### memory
在启动过程中，主要有以下几个方式添加内存到memory中。
1. device_type="memory"添加
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
在内核启动日志中"Early memory node ranges"可以看到所有的memroy范围，但没有包含每段的memblock_flags类型。

2. reserved-memory的no-map添加
在DTS中的定义如下：
```bash
reserved-memory {
    reserved_mem@0 {
        no-map;
        ret = <0x0 0xAA00000 0x0 0xBB00000>;
    }
}
```
启动过程中，会读取reserved-memory中的no-map部分将区域添加到memblock.memory中。如果memblock.memory中已经存在其他类型的，则会将该区域拆分减去no-map部分之后将两块都添加进去。
读取no-map的这段代码调用栈如下：
```c
+-- start_kernel
    +-- setup_arch
        +-- arm64_memblock_init
            +-- early_init_fdt_scan_reserved_mem
                +-- fdt_scan_reserved_mem
                    +-- __reserved_mem_reserve_reg
                        +-- early_init_dt_reserve_memory
```

在debugfs节点可以查到对应的memory区域：cat /sys/kernel/debug/memblock/memory

疑问：标记位no-map的如果在device_type="memory"段找不到会怎么样？会报错吗？

### reserve范围确定
reserve区域的添加有以下几种方式：
1. DTS预留
```bash
reserved-memory {
    reserved_mem@1 {
        ret = <0x0 0xCC00000 0x0 0xBB00000>;
    }
}
```
在启动过程中，处理reserved-memory并将区域添加到memblock.reserve中。
处理函数调用栈：
```c
+-- start_kernel
    +-- setup_arch
        +-- arm64_memblock_init
            +-- early_init_fdt_scan_reserved_mem
                +-- fdt_scan_reserved_mem
                    +-- __reserved_mem_reserve_reg
                        +-- early_init_dt_reserve_memory
```

2. 调用memblock_reserve接口预留
内核启动过程中，会将内核代码段之类的内存调用memblock_reserve添加到memblock.reserve中。


疑问：在启动完之后，reserved-memory {}中定义的很多段，其实在cat /sys/kernel/debug/memblock/reserve中没有显示
	（no-map的可以理解，这些段是保存在memory中，并被标记为MEMBLOCK_NOMAP的，所以在reserve中找不到也正常）
答案：没有标记no-map的段都是有的，标记位no-map的有些是找不到的。

疑问：标记位no-map的如果在device_type="memory"中找到了会怎么样？
答案：找到了就会在memory段中，把原来的段一份为2，，reserve-memory段重新生成一个memory区域并把他标记位MEMBLOCK_NOMAP，剩余的就是抠掉reserve-memory的。

## Linux启动日志中的内存区域
疑问：ZONE_DMA大小怎么计算出来的？
答案：可以看Initmem setup node就可以看出来
```bash
[    0.000000]Initmem setup node 0 [mem 0x0000000000AAA-0x000000BBBBffffff]
[    0.000000]On node 0 totalpages: ZZ
[    0.000000]DMA zone: AA pages used for memmap
[    0.000000]DMA zone: 0 pages reserved
[    0.000000]DMA zone: BB pages, LIFO batch:63
[    0.000000]Normal zone: CC pages used for memmap
[    0.000000]Normal zone: DD pages, LIFO batch:31
```
DMA zone在zone_sizes_init里边可以看出来，DMA zone的地址被赋值为0xFFFFFFFF（就是u32的最大值）。
free_area_init_node函数中，会将相同node0的memory中，在memory.lowest - 0xFFFFFFFF范围内的全部拿出来放到DMA zone中，并计算page个数。
DMA zone: BB pages, LIFO batch:63这句打印中，BB就是这么算出来的，，比如node0有5个memory范围，其中4个在memory.lower-0xFFFFFFFF范围内，就计算这4个的page个数然后加起来。
当然这些都是在64位系统中是这样的，32位不一样

## DTS中memreserve处理
一般都在dts最开始就定义，例如：
```c
/memreserve/ 0x40000000 0x01000000
```
## 参考
https://www.kernel.org/doc/html/v4.19/core-api/boot-time-mm.html
https://www.kernel.org/doc/gorman/

https://0xax.gitbooks.io/linux-insides/content/MM/linux-mm-1.html
