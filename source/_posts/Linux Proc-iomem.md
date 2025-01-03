---
title: Linux Proc-iomem
categories: 
- Linux
tags:
- Linux
---

## 说明
代码路径：kernel/resource.c文件

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

## 使用说明
1. page_is_ram：这个函数就是通过resource.c文件里边的iomem_resource中查找"System Ram"来确定pfn地址是否属于内存的。
2. kernel地址，数据范围： cat /proc/iomem | grep Kernel可以看到kernel的代码段和数据段范围

## 参考
https://blog.csdn.net/weixin_45030965/article/details/126122511

