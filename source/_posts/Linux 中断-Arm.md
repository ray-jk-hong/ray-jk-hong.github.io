---
title: Linux 中断-Arm
categories: 
- Linux 中断
tags:
- Linux 中断
---

## 中断介绍
### 中断类型介绍
#### 按照应用分类
| 中断类型 | 硬件中断号  | 描述 |
| --- | --- | --- |
| SGI | 0-15 | 软中断，由软件触发引起的中断，通过向寄存器 GICD_SGIR 写入数据来触发，系统会使用 SGI 中断来完成多核之间的通信|
| PPI | 16-31   | 私有外设中断，中断来自于外设，被特定的核处理。GIC 是支持多核的，每个核有自己独有的中断，例如timer相关的中断就是这个类型的中断 |
| SPI | 32-1019  | 共享外设中断，该中断来自于外设，所有 Core 共享的中断。比如按键中断、串口中断等等，这些中断所有的 Core 都可以处理，不限定特定 Core
| reserved | ...  |
| LPI | 8192-MAX  |

![gicv3-总体结构](/images/中断/gic-v3中断类型和范围.png)

- SGI (Software Generated Interrupt)：软件触发的中断。软件可以通过写 GICD_SGIR 寄存器来触发一个中断事件，一般用于核间通信，内核中的 IPI：inter-processor interrupts 就是基于 SGI。
- PPI (Private Peripheral Interrupt)：私有外设中断，只能发给一个指定的处理器处理。Linux timer是使用这类中断触发的。
- SPI (Shared Peripheral Interrupt)：公用的外部设备中断，也定义为共享中断。中断产生后，可以分发到某一个CPU上。比如按键触发一个中断，手机触摸屏触发的中断
- LPI (Locality-specific Peripheral Interrupt)：LPI 是 GICv3 中的新特性，它们在很多方面与其他类型的中断不同。LPI 始终是基于消息的中断，它们的配置保存在表中而不是寄存器。比如 PCIe 的 MSI/MSI-x 中断
#### 按照行为分类
按照行为可以分为消息中断（MSI）和线中断

### 中断模型
1. 1-N模型
中断被GIC发送给N个处理器，但只需要一个处理器接收处理。GIC确认有一个处理器处理了当前中断，GIC会将发给其余处理器的中断信号立刻关闭；这种模型一般用于SPI/PPI
(1) 1-1模型：SGI, PPI, SPI，LPI
(2) 1-N模型：SPI
2. N-N模型
中断被GIC发送给N个处理器了，也需要N个处理器都对其接收处理。即使GIC确认已有处理器在处理此中断，GIC仍会保持发给其余处理器的中断信号有效
(1) N-N模型：SGI、SPI。

### SPI中断


### LPI中断
#### LPI中断介绍
1. LPI中断是GICv3的新特性
2. 基于消息的中断，配置保存在表中，而不是寄存器中
3. LPI是消息中断，所有没有实体的中断线。
4. GIC内部提供一个寄存器，当外设往这个地址写入数据时，就会往GIC发送中断。

#### ITS（Interrupt Translation Service）
1. 介绍
   1) ITS用来解析LPI中断，将接收到的LPI中断解析，然后发送对应的Redistributor，再由Redistributor将中断信息发送给Cpu interface。
   2) 2) 外设通过写GITS_TRANSLATE寄存器，发起LPI中断，并提供ITS两个信息
      (1) Event Id:保存在GITS_TRANLATE中，表示外设发送中断的事件类型
      (2) Device Id:表示哪个外设发起的LPI中断
   3) ITS根据Event Id + Device Id查表，得到LPI中断号，再使用LPI中断号查表得到该中断的目标Cpu。
   4) ITS将LPI中断号，LPI中断对应的Cpu发送给Redistributor，Redistributor再将该中断号信息发给Cpu interface。
2. ITS的配置和存放信息
   1) ITS在初始化的时候需要分配内存，用于保存命令队列缓存和路由表
   2) CMDQ_BUFFER（命令队列缓存）：用于存放配置各个路由表命令，由软硬件共同维护
   3) DEVICE_TABLE（设备表）：用于映射指定设备号所占用的ITT的基地址
   4) ITT（中断传输表）：用于映射中断号和Collection（分组id）的关系
   5) CT（分组信息表）：用于映射Collection（分组id）到TA（目标地址，就是对应的GICR的地址）的关系

## GIC介绍
### GICv3组成
![gicv3-总体结构](/images/中断/gicv3-总体结构.png)

GICv3 控制器由以下三部分组成
#### GIC Distributor(GICD)：SPI 中断的分发
(1) 打开或关闭每个中断。Distributor对中断的控制分成两个级别。一个是全局中断的控制（GIC_DIST_CTRL）。一旦关闭了全局的中断，那么任何的中断源产生的中断事件都不会被传递到 CPU interface。另外一个级别是对针对各个中断源进行控制（GIC_DIST_ENABLE_CLEAR），关闭某一个中断源会导致该中断事件不会分发到 CPU interface，但不影响其他中断源产生中断事件的分发。
(2) 控制将当前优先级最高的中断事件分发到一个或者一组 CPU interface。当一个中断事件分发到多个 CPU interface 的时候，GIC 的内部逻辑应该保证只 assert 一个CPU
(3) 优先级控制
(4) interrupt属性设定。设置每个外设中断的触发方式：电平触发、边缘触发
(5) interrupt group的设定。设置每个中断的 Group，其中 Group0 用于安全中断，支持 FIQ 和 IRQ，Group1 用于非安全中断，只支持 IRQ

#### GIC Redistributor：SGI，PPI，LPI 中断的管理，将中断发送给 CPU interface
1. GICR能力
(1) 启用和禁用 SGI 和 PPI。
(2) 设置 SGI 和 PPI 的优先级。
(3) 将每个 PPI 设置为电平触发或边缘触发。
(4) 将每个 SGI 和 PPI 分配给中断组。
(5) 控制 SGI 和 PPI 的状态。
(6) 内存中数据结构的基址控制，支持 LPI 的相关中断属性和挂起状态。
(7) 电源管理支持。
2. GICR配置和信息存放
(1) 有属性配置表(Config Table)，PT表（Pending Table）和CPT表（Coarse Pending Table）。这些都存放在内存中
(2) 属性配置表(Config Table)：存放中断优先级，分组信息，使能状态
(3) PT表（Pending Table）：存放中断的挂起状态
(4) CPT表（Coarse Pending Table）：为了加快搜索内存Pending状态中断速度，因此需要一个CPT映射表，用于建立512个PT中断映射关系。每个CPT对应512个中断，里面信息存放一个优先级和pending状态，分别对应512个中断最高优先级和512个中断里是否有pending状态中断存在。类似一级页表和二级页表。

#### CPU interface：每个CPU Interface都负责把一个CPU连到GIC上
(1) 打开或关闭 CPU interface 向连接的 CPU assert 中断事件。对于 ARM，CPU interface 和 CPU 之间的中断信号线是 nIRQCPU 和 nFIQCPU。如果关闭了中断，即便是 Distributor 分发了一个中断事件到 CPU interface，也不会 assert 指定的 nIRQ 或者 nFIQ 通知 Core
(2) 中断的确认。Core 会向 CPU interface 应答中断（应答当前优先级最高的那个中断），中断一旦被应答，Distributor 就会把该中断的状态从 pending 修改成 active 或者 pending and active（这是和该中断源的信号有关，例如如果是电平中断并且保持了该 asserted 电平，那么就是 pending and active）。ack 了中断之后，CPU interface 就会 deassert nIRQCPU 和 nFIQCPU 信号线
(3) 中断处理完毕的通知。当 interrupt handler 处理完了一个中断的时候，会向写 CPU interface 的寄存器通知 GIC CPU 已经处理完该中断。做这个动作一方面是通知 Distributor 将中断状态修改为 deactive，另外一方面，CPU interface 会 priority drop，从而允许其他的 pending 的中断向 CPU 提交
(4) 为 CPU 设置中断优先级掩码。通过 priority mask，可以 mask 掉一些优先级比较低的中断，这些中断不会通知到 CPU
(5) 设置 CPU 的中断抢占（preemption）策略
(6) 在多个中断事件同时到来的时候，选择一个优先级最高的通知 CPU

#### GIC各个组成部分负责的中断类型
每个中断类型涉及的GICv3内部组成部分还不一样，如下图：
![gicv3-总体结构](/images/中断/gic-v3中断类型与模块关系.png)
- Distributor只有SPI中断类型涉及。GICv2中SGI/PPI这些都Distributor管，但在GICv3中，Distributor只管SPI中断
- PPI类型直接与Redistributor交互
- LPI有自己的中断翻译器（ITS），翻译之后再与Redistribotor交互
- SGI类型直接与CPU interface交互

### 中断状态机
![gicv3-总体结构](/images/中断/gic-v3中断状态机.png)
- Inactive：无中断状态，即没有 Pending 也没有 Active
- Pending：硬件或软件触发了中断，该中断事件已经通过硬件信号通知到 GIC，等待 GIC 分配的那个 CPU 进行处理，在电平触发模式下，产生中断的同时保持 Pending 状态
- Active：CPU 已经应答（acknowledge）了该中断请求，并且正在处理中
- Active and pending：当一个中断源处于 Active 状态的时候，同一中断源又触发了中断，进入 pending 状态

### 寄存器
GICC_AHPPIR

## 中断亲和性设置
irq_set_affinity

## 参考


## 参考
http://www.wowotech.net/irq_subsystem/gic_driver.html

https://www.google.com/search?q=Marc+Zyngier+please+interrupt+me&sca_esv=08f49ddedad890b7&biw=1766&bih=548&ei=Y6TWZrv_Lo_c2roPvOzM-AU&ved=0ahUKEwj7vI3viqaIAxUPrlYBHTw2E184FBDh1QMIEA&uact=5&oq=Marc+Zyngier+please+interrupt+me&gs_lp=Egxnd3Mtd2l6LXNlcnAiIE1hcmMgWnluZ2llciBwbGVhc2UgaW50ZXJydXB0IG1lMgUQIRigATIFECEYoAEyBRAhGKABMgUQIRigAUiPLFChB1iUK3AEeACQAQCYAX6gAbYMqgEEMTguMrgBA8gBAPgBAZgCF6AC1gzCAgUQABiABMICBBAAGB7CAggQABiABBiiBMICBxAhGKABGAqYAwCIBgGSBwQyMS4yoAe3Mw&sclient=gws-wiz-serp

https://cloud.tencent.com/developer/article/1867927

http://www.wowotech.net/irq_subsystem/gic-irq-chip-driver.html

arm generic interrupt controller architecture specification 4.0

