---
title: Kvm
categories: 
- Linux
tags:
- Linux Virt
---

## Ubuntu安装kvm
1. 查看当前cpu架构是否支持安装kvm
egrep -c '(vmx|svm)' /proc/cpuinfo


## 参考
QEMU KVM 源码解析与应用.pdf
