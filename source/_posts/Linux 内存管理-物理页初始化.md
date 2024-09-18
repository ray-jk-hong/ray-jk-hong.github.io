---
title: Linux 物理页初始化
categories: 
- Linux MM
tags:
- Linux MM
---

## Buddy相关结构体
buddy的算法不多讲了，很多资料。直接看buddy相关的结构体
{% plantuml %}
allowmixing

rectangle NumaX #lightgreen

enum zone_type {
#ifdef CONFIG_ZONE_DMA
	ZONE_DMA,
#endif
    ZONE_NORMAL,
#ifdef CONFIG_HIGHMEM    
    ZONE_HIGHMEM,
#endif
    ZONE_MOVABLE,
#ifdef CONFIG_ZONE_DEVICE
    ZONE_DEVICE,
#endif
    __MAX_NR_ZONES
}

class page {
    struct list_head buddy_list;
    unsigned int page_type；
    unsigned long private；
}

note right of page::page_type
    一段pfn range的第一个page会被链接到free_list，
    第一个page的page_type会被设置为PG_buddy（通过__SetPageBuddy设置），定义在[inlcude/linux/page-flags.h]中
end note

note right of page::page_type
    一段pfn range的第一个page会被链接到free_list，
    第一个page的private会被设置为order。
end note

struct free_area {
    struct list_head	free_list[MIGRATE_TYPES];
    unsigned long		nr_free;
}

free_area::free_list -> page::buddy_list

class zone {
    unsigned long _watermark[NR_WMARK];
    unsigned long watermark_boost;
    long lowmem_reserve[MAX_NR_ZONES];
    struct pglist_data	*zone_pgdat;
    atomic_long_t		vm_stat[NR_VM_ZONE_STAT_ITEMS];
    atomic_long_t		vm_numa_event[NR_VM_NUMA_EVENT_ITEMS];

    struct free_area	free_area[NR_PAGE_ORDERS];
}

class pglist_data {
    struct zone node_zones[MAX_NR_ZONES];
    struct zonelist node_zonelists[MAX_ZONELISTS];
    struct task_struct *kswapd;

    unsigned long node_start_pfn;
    unsigned long node_present_pages;
    unsigned long node_spanned_pages;

    int node_id;
}

zone::free_area -down-> free_area
pglist_data -> zone
NumaX -down-> pglist_data : 每个numa都有一个pglist

note left of pglist_data::node_zones
    MAX_NR_ZONES由zone_type的
    最大值__MAX_NR_ZONE决定
end note

note left of pglist_data::node_start_pfn
    开始的page对应的pfn
end note

note left of pglist_data::node_id
    Numa node id
end note
{% endplantuml %}

1. 一个Numa对应一个pglist_data结构体
2. 根据zone类型（dma或者normal）找到对应的zone
3. zone->free_area[order]就是链接page的buddy结构的链表头
   通过也通过page可以找到zone: struct zone *page_zone(const struct page *page)

## pglist_data初始化
1. 根据memblock确定pglist_data的范围，并初始化page结构体
```c
+-- start_kernel:
    +-- setup_arch:
        +-- bootmem_init:
            +-- zone_sizes_init:
                +-- free_area_init
                    +-- memmap_init
```
1) free_area_init的前面的部分将pglist_data结构体初始化，确定范围等
2) memmap_init会扫描memblock的pfn范围，将page结构体拿到并初步初始化page结构体，但没有真正加到zone里边（buddy初始化还没开始）

2. zone初始化（buddy算法初始化）
```c
+-- start_kernel
    +-- mm_core_init
        +-- build_all_zonelists:
        	+-- build_all_zonelists_init
                +-- __build_all_zonelists
                    +-- build_zonelists
    +-- mem_init
        +-- memblock_free_all
            +-- free_low_memory_core_early
```

初始化日志中，可以看到初始化之后的buddy info：
```c
Fallback order for Node 0: 0 30 33 34 35 36 37 38 39 40 31
Fallback order for Node 1: 0 30 33 34 35 36 37 38 39 40 31
...
```

## 内存分布相关宏
宏的后面写的定义或者未定义写的是我自己的虚拟机的配置。
1. CONFIG_FLATMEM：（未定义）
   内存段只有一个的时候使用，即中间没有空洞预留等
2. CONFIG_SPARSEMEM（定义）
   表示内存段可以有多个的时候使用
3. CONFIG_SPARSEMEM_MANUAL（定义）
4. CONFIG_SPARSEMEM_EXTREME（定义）
5. CONFIG_SPARSEMEM_VMEMMAP（定义）
   和CONFIG_SPARSEMEM相似，但将page结构体全部保存到一个虚拟地址中，提高page_to_pfn等操作的性能。
   pfn to page就定义成这样，，可以看到page结构体都放在vmemmap基地址的虚拟地址上。
```c
    #define __pfn_to_page(pfn)	(vmemmap + (pfn))
```
   这相比于CONFIG_SPARSEMEM的pfn->section->pag的方式性能提高了一些。
   
4. CONFIG_SPARSEMEM_VMEMMAP_ENABLE（定义）

https://qiita.com/akachochin/items/121d2bf3aa1cfc9bb95a

## 地址相关宏
```c
#define VA_BITS                (CONFIG_ARM64_VA_BITS)
#define _PAGE_OFFSET(va)        (-(UL(1) << (va)))
#define PAGE_OFFSET            (_PAGE_OFFSET(VA_BITS))
```
1. CONFIG_ARM64_VA_BITS：虚拟地址的bit位数，可以设置54/48等（具体可以看mmu寄存器那篇）。
2. PAGE_OFFSET：就是TTBR1选择的地址了，，如果CONFIG_ARM64_VA_BITS=48， 则PAGE_OFFSET的值就是0xFFFF000000000000
3. VA_BITS_MIN：虚拟地址最小值，如果4K页表的话就跟CONFIG_ARM64_VA_BITS是一样的
```c
#if VA_BITS > 48
#ifdef CONFIG_ARM64_16K_PAGES
#define VA_BITS_MIN    (47)
#else
#define VA_BITS_MIN    (48)
#endif
#else
#define VA_BITS_MIN    (VA_BITS)
#endif
```

## 参考
buddy初始化：https://blog.csdn.net/weixin_42262944/article/details/118276396

buddy算法：https://www.kernel.org/doc/gorman/html/understand/understand009.html
