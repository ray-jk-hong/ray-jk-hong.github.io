---
title: Kasan
categories: 
- Linux MM
tags:
- Linux MM
---

主要功能是检查内存越界访问和使用已释放的内存等问题。

## 打开方法
CONFIG_KASAN=Y
CONFIG_KASAN_INLINE=Y

```
inline vs outline
Inline instrumentation gives a bigger image but is faster, because of no overhead of calling the KASan callbacks, 
while outline instrumentation gives a smaller image but is slower
```

## log分析

## 参考
https://www.kernel.org/doc/html/v4.14/dev-tools/kasan.html
https://blog.csdn.net/feelabclihu/article/details/109685476
http://www.wowotech.net/memory_management/424.html

CONFIG_PAGE_OWNER
./Documentation/dev-tools/kasan.rst

