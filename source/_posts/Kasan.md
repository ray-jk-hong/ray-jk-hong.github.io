---
title: Kasan
categories: 
- Linux
tags:
- Linux Kasan
---

## 打开方法
CONFIG_KASAN=Y
CONFIG_KASAN_INLINE=Y

```
inline vs outline
Inline instrumentation gives a bigger image but is faster, because of no overhead of calling the KASan callbacks, 
while outline instrumentation gives a smaller image but is slower
```

## 参考
https://www.kernel.org/doc/html/v4.14/dev-tools/kasan.html
https://blog.csdn.net/feelabclihu/article/details/109685476
