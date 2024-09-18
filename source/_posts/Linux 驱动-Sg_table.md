---
title: Linux Sg_table
categories: 
- Linux Driver
tags:
- Linux Driver
---

struct sg_table 
struct scatterlist

## struct scatterlist
```c
struct scatterlist {
#ifdef CONFIG_DEBUG_SG
        unsigned long   sg_magic;
#endif
        unsigned long   page_link;
        unsigned int    offset;
        unsigned int    length;
        dma_addr_t      dma_address;
#ifdef CONFIG_NEED_SG_DMA_LENGTH
        unsigned int    dma_length;
#endif
};
```

## 参考
https://www.wowotech.net/memory_management/scatterlist.html

https://stackoverflow.com/questions/29270447/how-does-scatterlist-works-in-linux

