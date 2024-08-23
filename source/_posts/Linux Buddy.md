---
title: Linux Buddy
categories: 
- Linux MM
tags:
- Linux MM
---

## Buddy相关结构体
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
    第一个page的page_type会被设置为PG_buddy。
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
## 相关宏
```c
CONFIG_FLATMEM
CONFIG_SPARSEMEM
```

## 参考
buddy初始化：https://blog.csdn.net/weixin_42262944/article/details/118276396
