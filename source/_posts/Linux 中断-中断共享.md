---
title: Linux 中断共享
categories: 
- Linux
tags:
- Linux
---


## 共享中断时，返回值
Dev1, Dev2共享中断，在request_irq时，Dev1, Dev2 调用request_irq时指定一个中断IRQ10，flag填写IRQF_SHARED。在中断回调函数中，判断是否是本设备返回的中断，不是就返回IRQ_NONE让继续执行下个回调。
如果中断是本设备触发，则返回IRQ_HANDLED，这样就不会再往下执行其他的中断回调了。


https://stackoverflow.com/questions/14371513/for-a-shared-interrupt-line-how-do-i-find-which-interrupt-handler-to-use

