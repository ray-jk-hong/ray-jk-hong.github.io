---
title: Linux Proc-iomem
categories: 
- Linux Proc
tags:
- Linux Proc
---

/proc/iomem的显示，显示的都是insert_resource接口加进去的。包括System Ram，Reserved和内核代码段，还有一些驱动。
1. 启动时添加Memblock
/proc/iomem里边System Ram和Reserved的一部分是在启动的时候通过Memblock添加进去的。

```c
+-- setup_arch
  +-- request_standard_resources
    +-- insert_resource
```

https://blog.csdn.net/weixin_45030965/article/details/126122511
