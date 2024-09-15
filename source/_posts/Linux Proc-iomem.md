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

2. 其他
其他的就不说了，像驱动，printk需要的内存等等都是通过insert_resource添加进去的，看代码就好了。
insert_resource的时候传入的第一个参数是iomem_resource就是添加到/proc/iomem统计里边去掉的。

https://blog.csdn.net/weixin_45030965/article/details/126122511
