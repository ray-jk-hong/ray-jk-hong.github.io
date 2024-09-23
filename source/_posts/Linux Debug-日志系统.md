---
title: Linux 日志系统
categories: 
- Linux Debug
tags:
- Linux Debug
---

## %pK打印
kptr_restrict 限制内核地址显示
执行如下命令，可以解除地址打印限制
```
echo 0 > /proc/sys/kernel/kptr_restrict 
```

## 参考
https://blog.csdn.net/jiangliuhuan123/article/details/131143236
