---
title: USB驱动
categories: 
- Linux Driver
tags:
- Linux Driver
---

## 物理地址的范围
刚开始需要确定node 0物理地址的范围。
1. bootloader上传物理地址范围

## memblock查询
```bash
root@ubuntu-24-04:/sys/kernel/debug/memblock# cat memory 
   0: 0x0000000080000000..0x00000001d54fffff    0 NONE
root@ubuntu-24-04:/sys/kernel/debug/memblock# cat reserved 
   0: 0x0000000080010000..0x000000008266ffff    0 NONE
   1: 0x00000000fbfff000..0x00000000ffffefff    0 NONE
   2: 0x00000001ce3a3000..0x00000001d47fffff    0 NONE
   3: 0x00000001d483fbc0..0x00000001d483ffc7    0 NONE
   4: 0x00000001d4842000..0x00000001d48426c7    0 NONE
   5: 0x00000001d4842700..0x00000001d484275f    0 NONE
   6: 0x00000001d4842780..0x00000001d4842907    0 NONE
   7: 0x00000001d4842940..0x00000001d4842b87    0 NONE
   8: 0x00000001d4842bc0..0x00000001d4842cef    0 NONE
   9: 0x00000001d4842d00..0x00000001d4842d4f    0 NONE
  10: 0x00000001d4842d80..0x00000001d4842ea6    0 NONE
  11: 0x00000001d4842ec0..0x00000001d4842fe6    0 NONE
  12: 0x00000001d4843000..0x00000001d4846027    0 NONE
  13: 0x00000001d4846040..0x00000001d4846047    0 NONE
  14: 0x00000001d4846080..0x00000001d4846087    0 NONE
  15: 0x00000001d48460c0..0x00000001d48467b7    0 NONE
  16: 0x00000001d48467c0..0x00000001d54fffff    0 NONE
  17: 0x00000003ffe00000..0x00000003ffe00f1b    0 NONE
```
17超过了memblock/memory的范围？

## 参考
https://www.kernel.org/doc/html/v4.19/core-api/boot-time-mm.html
https://www.kernel.org/doc/gorman/
