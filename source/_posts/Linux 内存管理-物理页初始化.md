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
                    +-- free_area_init_node
                        +-- free_area_init_core 
                    +-- memmap_init
```
1) free_area_init的前面的部分将pglist_data结构体初始化，确定范围等
2) memmap_init会扫描memblock的pfn范围，将page结构体拿到并初步初始化page结构体，但没有真正加到zone里边（buddy初始化还没开始）

- node_zones The zones for this node, ZONE_HIGHMEM, ZONE_NORMAL, ZONE_DMA;
- node_zonelists This is the order of zones that allocations are preferred from. build_zonelists() in mm/page_alloc.c sets up the order when called by free_area_init_core(). A failed allocation in ZONE_HIGHMEM may fall back to ZONE_NORMAL or back to ZONE_DMA;
- nr_zones Number of zones in this node, between 1 and 3. Not all nodes will have three. A CPU bank may not have ZONE_DMA for example;
- node_mem_map This is the first page of the struct page array representing each physical frame in the node. It will be placed somewhere within the global mem_map array;
- valid_addr_bitmap A bitmap which describes “holes” in the memory node that no memory exists for. In reality, this is only used by the Sparc and Sparc64 architectures and ignored by all others;
- bdata This is only of interest to the boot memory allocator discussed in Chapter 5;
- node_start_paddr The starting physical address of the node. An unsigned long does not work optimally as it breaks for ia32 with Physical Address Extension (PAE) for example. PAE is discussed further in Section 2.5. A more suitable solution would be to record this as a Page Frame Number (PFN). A PFN is simply in index within physical memory that is counted in page-sized units. PFN for a physical address could be trivially defined as (page_phys_addr >> PAGE_SHIFT);
- node_start_mapnr This gives the page offset within the global mem_map. It is calculated in free_area_init_core() by calculating the number of pages between mem_map and the local mem_map for this node called lmem_map;
- node_size The total number of pages in this zone;
- node_id The Node ID (NID) of the node, starts at 0;
- node_next Pointer to next node in a NULL terminated list.

## zone初始化（buddy算法初始化）
初始化zone并将page连接到zone的流程如下：
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
                +-- __free_memory_core
                    +-- __free_pages_memory
                        +-- memblock_free_pages
                            +-- __free_pages_core
                                +-- __free_pages_ok
                                    +-- free_one_page
                                        +-- __free_one_page
```

初始化日志中，可以看到初始化之后的buddy info：
```c
Fallback order for Node 0: 0 30 33 34 35 36 37 38 39 40 31
Fallback order for Node 1: 0 30 33 34 35 36 37 38 39 40 31
...
```

## Page结构体
### page_type
  include/linux/page-flags.h文件中PAGE_TYPE_OPS定义的一组page类型的定义。有如下几个：
```
    PAGE_TYPE_OPS(Buddy, buddy) ： 表示page是free的且在buddy系统中（连接的zone的free_area中，当然这个page肯定是componed head的page，其他page即使在buddy系统中也不会设置这个）
    PAGE_TYPE_OPS(Offline, offline)
    PAGE_TYPE_OPS(Table, table)
    PAGE_TYPE_OPS(Guard, guard)
```
  上面的几个函数定义，就是将如下定义设置到page->page_type里边去，表示page的用途的。
```
    #define PAGE_TYPE_BASE	0xf0000000
    /* Reserve		0x0000007f to catch underflows of page_mapcount */
    #define PAGE_MAPCOUNT_RESERVE	-128
    #define PG_buddy	0x00000080
    #define PG_offline	0x00000100
    #define PG_table	0x00000200
    #define PG_guard	0x00000400
```
### flags
PG_active: This bit is set if a page is on the active_list LRU and cleared when it is removed. It marks a page as being hot
PG_arch_1：Quoting directly from the code: PG_arch_1 is an architecture specific page state bit. The generic code guarantees that this bit is cleared for a page when it first is entered into the page cache. This allows an architecture to defer the flushing of the D-Cache (See Section 3.9) until the page is mapped by a process
PG_checked：Only used by the Ext2 filesystem
PG_dirty：This indicates if a page needs to be flushed to disk. When a page is written to that is backed by disk, it is not flushed immediately, this bit is needed to ensure a dirty page is not freed before it is written out
PG_error：If an error occurs during disk I/O, this bit is set
PG_fs_1：Bit reserved for a filesystem to use for it's own purposes. Currently, only NFS uses it to indicate if a page is in sync with the remote server or not
PG_highmem：Pages in high memory cannot be mapped permanently by the kernel. Pages that are in high memory are flagged with this bit during mem_init()
PG_launder：This bit is important only to the page replacement policy. When the VM wants to swap out a page, it will set this bit and call the writepage() function. When scanning, if it encounters a page with this bit and PG_locked set, it will wait for the I/O to complete
PG_locked：This bit is set when the page must be locked in memory for disk I/O. When I/O starts, this bit is set and released when it completes
PG_lru：If a page is on either the active_list or the inactive_list, this bit will be set
PG_referenced：If a page is mapped and it is referenced through the mapping, index hash table, this bit is set. It is used during page replacement for moving the page around the LRU lists
PG_reserved：This is set for pages that can never be swapped out. It is set by the boot memory allocator (See Chapter 5) for pages allocated during system startup. Later it is used to flag empty pages or ones that do not even exist
PG_slab：This will flag a page as being used by the slab allocator
PG_skip：Used by some architectures to skip over parts of the address space with no backing physical memory
PG_unused：This bit is literally unused
PG_uptodate：When a page is read from disk without error, this bit will be set.

flag设置/校验/清楚的接口：
| **Bit name** | **Set** | Test | Clear |
| ---- | ---- | ---- | ---- |
| PG_active | SetPageActive() | PageActive() | ClearPageActive() |
| PG_arch_1 | N/A | N/A | N/A |
| PG_checked | SetPageChecked() | PageChecked() | N/A |
| PG_dirty | SetPageDirty() | PageDirty() | ClearPageDirty() |
| PG_error | SetPageError() | PageError() | ClearPageError() |
| PG_highmem | N/A | PageHighMem() | N/A |
| PG_launder | SetPageLaunder() | PageLaunder() | ClearPageLaunder() |
| PG_locked | LockPage() | PageLocked() | UnlockPage() |
| PG_lru | TestSetPageLRU() | PageLRU() | TestClearPageLRU() |
| PG_referenced | SetPageReferenced() | PageReferenced() | ClearPageReferenced() |
| PG_reserved | SetPageReserved() | PageReserved() | ClearPageReserved() |
| PG_skip | N/A | N/A | N/A |
| PG_slab | PageSetSlab() | PageSlab() | PageClearSlab() |
| PG_unused() | N/A | N/A | N/A |
| PG_uptodate | SetPageUptodate() | PageUptodate() | ClearPageUptodate() |

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
https://www.kernel.org/doc/gorman/html/understand/understand005.html#tab:%20Page%20Status%20Macros
