---
title: Kvm
categories: 
- Linux 虚拟化
tags:
- Linux 虚拟化
---

## Ubuntu安装kvm
1. 查看当前cpu架构是否支持安装kvm
```c
egrep -c '(vmx|svm)' /proc/cpuinfo
```
![页表描述](/images/Kvm/kvm安装条件.png)

## 参考
QEMU KVM 源码解析与应用.pdf

https://phoenixnap.com/kb/ubuntu-install-kvm
