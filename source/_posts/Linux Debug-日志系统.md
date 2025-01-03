---
title: Linux 日志系统
categories: 
- Linux
tags:
- Linux
---

## %pK打印
kptr_restrict 限制内核地址显示
执行如下命令，可以解除地址打印限制
```
echo 0 > /proc/sys/kernel/kptr_restrict 
```

## 函数指针转函数名打印
%pf或者%ps可以打印符号/函数名称（但需要配置KALLSYMS）
%PF或者%pS除了打印名称还有偏移

## 参考
https://blog.csdn.net/jiangliuhuan123/article/details/131143236
