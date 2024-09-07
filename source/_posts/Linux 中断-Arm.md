---
title: Linux GICv3代码分析
categories: 
- Linux 中断
tags:
- Linux 中断
---

## 参考
http://www.wowotech.net/irq_subsystem/gic_driver.html

https://www.google.com/search?q=Marc+Zyngier+please+interrupt+me&sca_esv=08f49ddedad890b7&biw=1766&bih=548&ei=Y6TWZrv_Lo_c2roPvOzM-AU&ved=0ahUKEwj7vI3viqaIAxUPrlYBHTw2E184FBDh1QMIEA&uact=5&oq=Marc+Zyngier+please+interrupt+me&gs_lp=Egxnd3Mtd2l6LXNlcnAiIE1hcmMgWnluZ2llciBwbGVhc2UgaW50ZXJydXB0IG1lMgUQIRigATIFECEYoAEyBRAhGKABMgUQIRigAUiPLFChB1iUK3AEeACQAQCYAX6gAbYMqgEEMTguMrgBA8gBAPgBAZgCF6AC1gzCAgUQABiABMICBBAAGB7CAggQABiABBiiBMICBxAhGKABGAqYAwCIBgGSBwQyMS4yoAe3Mw&sclient=gws-wiz-serp

## 寄存器
GICC_AHPPIR
