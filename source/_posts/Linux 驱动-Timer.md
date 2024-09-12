---
title: Linux Timer
categories: 
- Linux Driver
tags:
- Linux Driver
---

## Sleep接口注意
在使用这类接口的时候，有一些原则需要遵守：（根据等待时间长短）

1.等待时间很短（< ~10us）
  这时最好使用udelay替换usleep_range。

2.睡眠时间在（10us ~ 20ms）
  这个时间范围，最好使用usleep_range进行睡眠

3.睡眠时间较长（10ms+）
  使用msleep或者msleep_interruptible。
