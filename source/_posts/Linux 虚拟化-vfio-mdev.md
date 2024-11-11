---
title: Linux vfio-mdev
categories: 
- Linux
tags:
- Linux
---

## vfio-mdev的设备模型
![mdev](/images/Linux/虚拟化/mdev.png)

1. mdev.ko[在vfio/mdev目录下的文件]在内部完成总线注册工作，向外提供device注册和driver注册接口。而且mdev.ko在mtty.c等device注册设备的时候，创建对应的sysfs接口，写该接口就会回调到mtty.c文件中的接口创建，销毁设备。

2. vfio_mdev.ko[在vfio目录下的文件]，向mdev.ko注册driver。vfio_mdev.ko向用户态提供/dev/vfio/vfio设备和/dev/vfio/group设备驱动接口。

3. mtty.c这种活着上图中的nvidia.ko等都是具体的device了。会调用mdev_register_device添加设备。



## 软件架构
![mdev](/images/Linux/虚拟化/vfio-mdev.svg)

## 参考
https://www.cnblogs.com/LoyenWang/category/1828942.html

https://www.kernel.org/doc/Documentation/vfio-mediated-device.txt


https://kernelgo.org/vfio-insight.html

