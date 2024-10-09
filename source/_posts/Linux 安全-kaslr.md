---
title: Linux Kaslr
categories: 
- Linux
tags:
- Linux
---

发现一个64位高通平台的一个现象，假设一个内核函数在system.map中的地址是A，在kernel中把这个函数地址打印出来是B，A和B理论上应该相等的吧，
可是实际是每次重启B都会变化，但是A和B的低20位是相同的，为什么A和B不同

4.4内核吧，kaslr打开之后会有这种效果
