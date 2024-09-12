---
title: Linux DynamicDebug
categories: 
- Linux Debug
tags:
- Linux Debug
---

## 需要使能的宏
CONFIG_DYNAMIC_DEBUG

## 按照范围使能
### 函数
echo -n "func xxx +p" > /sys/kernel/debug/dynamic_debug/control 

### 整个文件
echo "file svcsock.c +p" > /sys/kernel/debug/dynamic_debug/control

### 文件中的某一行
echo -n '  file   svcsock.c     line  1603 +p  ' > /sys/kernel/debug/dynamic_debug/control

### 模块
要把aa.ko模块的所有debug日志打开，可以使用如下命令：
echo "module aa +p" > /sys/kernel/debug/dynamic_debug/control

### 多个命令组合
多个命令可以放到一次输入中，通过';'或者'\n'区分多个命令
echo "func pnpacpi_get_resources +p; func pnp_assign_mem +p" > /sys/kernel/debug/dynamic_debug/control

### 按文件输入
如果要设置的比较多，可以写入文件中，按文件输入。
cat query-batch-file > /sys/kernel/debug/dynamic_debug/control

### 按照文件目录
按照如下方式，可以将所有usb驱动的debug打印出来
echo "file drivers/usb/* +p" > /sys/kernel/debug/dynamic_debug/control

## 参考
https://www.kernel.org/doc/html/v4.19/admin-guide/dynamic-debug-howto.html
