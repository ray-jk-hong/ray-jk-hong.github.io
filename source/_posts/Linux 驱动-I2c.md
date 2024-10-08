---
title: I2c Protocol
categories: 
- Linux
tags:
- Linux
---

## I2C协议综述
I2C总线包含两条线，SDA(串行数据线)/SDL(串行时钟线). 原理是通过SDA/SDL的电平高低的时序控制来进行数据的传递。
在空闲状态时，这两个线一般被上面所接的上拉电阻拉高，保持高电平。
I2C是半双工通信方式，同一时间只能单向通信。通信速度根据通信模式如下：
(1) 标准模式：100Kbit/s
(2) 快速模式：400Kbit/s
(3) 高速模式：3.4Mbit/s。

## I2C主设备/从设备
I2C的是分主设备与从设备的。
1. I2C通信时，通信设备之间的地位是平等的，分为主设备和从设备，其中主设备一个、从设备多个。主设备要主导整个通信过程，从设备根据I2C协议被动的响应主设备；
2. 在I2C通信中，没有规定谁做主设备、谁做从设备，是通信双方自己协商的。一个设备在同一时间只能做主设备或者从设备，但是有的设备可以通过软件配置来决定在此次通信时做主设备还是从设备。

## I2C总线
### I2C总线状态：I2C数据传输单位是一个字节(8bit)，数据前后要有一个开始信号和结束信号。根据SDA/SDL电平高低，I2C总线状态可以分为如下几种：
1. SDA/SDL高电平：空闲
2. SDA由高变低，SDL高电平：开始信号
3. SDL由低变高：SDL高电平：结束信号

### I2C总线状态转移：
在开始条件产生后，总线出于忙状态，总线由数据传输的主从设备独占，其他I2C期间无法访问总线。
在停止条件产生后，本次数据传输的主从设备将释放总线，总线再次出于空闲状态。

### I2C数据传输：
I2C发送完开始信号之后，==主设备== 在SCL线上产生每个时钟脉冲的过程中将在SDA线上传输一个数据位，当一个字节按数据位从高位到低位的顺序传输完后，紧接着==从设备==将拉低SDA线，回传给主设备一个应答位， 此时才认为一个字节真正的被传输完成。当然，并不是所有的字节传输都必须有一个应答位，比如：当从设备不能再接收主设备发送的数据时，从设备将回传一个否定应答位。ACK信号就是从设备在拉低SDA之后，再给一个SDL脉冲？

## 参考
https://doc.embedfire.com/linux/imx6/driver/zh/latest/linux_driver/i2c_mpu6050.html

https://www.cnblogs.com/multimicro/p/11905647.html

## 代码
寄存器级别的操作，初始化，软复位等等
drivers/i2c/busses/i2c-designware-core.h
drivers/i2c/busses/i2c-designgware-master.c
drivers/i2c/busses/i2c-designware-common.c

designeware i2c datasheet
https://www.datasheetarchive.com/?q=designware%20i2c


