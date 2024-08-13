---
title: StartUp
categories: 
- Linux
tags:
- Linux StartUp
---

## 内核入口
内核入口需要查看lds链接脚本。KBUILD_LDS定义了链接脚本的路径：arch/$(SRCARCH)/kernel/vmlinux.lds。
arch/arm64/kernel/vmlinux.lds
