---
title: Linux Kbuild
categories: 
- Linux 编译
tags:
- Linux 编译
---


## KBUILD_MODNAME
可以在代码里边直接使用，KBUILD_MODNAME是字符串，是当前模块的名字。（疑问：多个.o文件组成的，怎么确定名字？）
可以使用KBUILD_MODNAME，在打印的时候显示模块名字，在有多个ko组成的系统中避免混淆。
例如在.c文件最开头，定义：
#define pr_fmt(fmt) "KBUILD_MODNAME: <%s:%d> " fmt, __func__, __LINE__
这样在调用pr_xx打印的时候就可以显示模块名，函数，行号之类的信息了。
