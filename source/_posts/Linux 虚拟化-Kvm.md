---
title: Linux Kvm
categories: 
- Linux
tags:
- Linux
---

## Ubuntu安装kvm
1. 查看当前cpu架构是否支持安装kvm
```c
egrep -c '(vmx|svm)' /proc/cpuinfo
```
![页表描述](/images/Kvm/kvm安装条件.png)

0表示不支持，非0表示支持

## 参考
QEMU KVM 源码解析与应用.pdf

https://phoenixnap.com/kb/ubuntu-install-kvm
