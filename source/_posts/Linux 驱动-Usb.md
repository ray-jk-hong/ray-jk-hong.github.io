---
title: Linux Usb
categories: 
- Linux Driver
tags:
- Linux Driver
---

## USB基础
### 设备与驱动基础架构
Linux USB子系统，大的可以分成USB驱动和USB设备。
USB设备由Config, Interface, Endpoints组成。
如下图：
![Usb设备架构](/images/Usb/Usb设备大致结构.png)

lsusb -vvv查看usb设备，是按照如下架构打印的
```
+-- Device Descriptor:
    +-- Configuration Descriptor:
        +-- Interface Descriptor:
            +-- Endpoint Descriptor:
```

Linux USB子系统软件架构如下：
![Usb驱动架构](/images/Usb/Usb驱动大致结构.png)

### Endpoint
USB的基础通信是通过Endpoint进行。一个Endpoint只能是发送或者接收。Endpoint可以想象成一个单方向的pipe。
Endpoint可以是如下4种之一：
- CONTROL：
    一般用来配置设备，收集设备信息，发送命令给设备。这种Endpoint一般size较小。每个USB设备都有一个叫“endpoint 0”的CONTROL Endpoint。"endpoint 0"会在USB插入的时间点用来配置设备。
- INTERRUPT：
    在USB host请求USB device数据的时候，Device会使用此Endpoint按照固定速率传回小量数据。
- BULK：
    发送大量的数据，且数据不能丢失的场景。通常在printer, storage或者network设备上使用。
- ISOCHRONOUS
    也是发送大量数据，但数据不是100%保证不丢失的。通常audio/video设备都使用这种。这种一般都是可以容忍部分数据丢失的，但数据必须要是实时到达的数据。

CONTROL和BULK模式用于异步数据发送，而且不是周期性的，是驱动什么时候想发送的时候就会用一下。INTERRUP和ISOCHRONOUS模式是周期性的。

传输方式：
https://www.cnblogs.com/linhaostudy/p/11393740.html


Linux内核使用struct usb_host_endpoint描述endpoint。

{% plantuml %}
allowmixing

class usb_endpoint_descriptor {
	__u8  bLength;
	__u8  bDescriptorType;

	__u8  bEndpointAddress;
	__u8  bmAttributes;
	__le16 wMaxPacketSize;
	__u8  bInterval;

	/* NOTE:  these two are _only_ in audio endpoints. */
	/* use USB_DT_ENDPOINT*_SIZE in bLength, not sizeof. */
	__u8  bRefresh;
	__u8  bSynchAddress;
}

class usb_host_endpoint {
    struct usb_endpoint_descriptor		desc;
}

usb_endpoint_descriptor -> usb_host_endpoint

note left of usb_endpoint_descriptor::bEndpointAddress
    1. endpoint的地址
    2. endpoint的方向，linux中由USB_DIR_IN/US_DIR_OUT表示
end note

note left of usb_endpoint_descriptor::bmAttributes
    表示endpoint类型，linux中由USB_ENDPOINT_XFER_BULK等表示
end note

note left of usb_endpoint_descriptor::wMaxPacketSize
    表示Endpoint最大能处理的最大数据大小（单位是Byte）。
    USB驱动可以通过Endpoint发送比这个数值大的多的数据，但是多切多次发送，每次大小不能超过这个值。
end note

note left of usb_endpoint_descriptor::bInterval
    如果Endpoint是中断类型，则该值是Endpoint的间隔设置——即Endpoint之间中断请求的时间。
    该值以毫秒为单位表示。
end note

{% endplantuml %}

### Interface
Interface捆绑了几个Endpoint。Interface只处理一种类型的逻辑链接，例如mouse, keyboard, audio流。有些USB设备有多个Interface。
例如Speaker可能有两个USB Interface。一个是USB按键，另一个是USB音频流。由于一个USB驱动控制一个Interface，所以这种Speaker需要两个
USB驱动。
每个USB Interface可以有多个备用配置。启动的时候的设置是0号行为，启动之后可以通过备用配置，配置USB的某个Endpoint。
例如配置某个Endpoint预留带宽等。ISOCHRONOUS模式的Endpoint会使用备用的配置。

Linux驱动通过struct usb_interface structure描述USB Interface。

- struct usb_host_interface *altsetting：
  备用设置的数组，每个struct usb_host_interface 都有一组Endpoint的配置（在结构体struct usb_host_endpoint中定义）。
- unsigned num_altsetting：
  备用配置的个数。
- struct usb_host_interface *cur_altsetting
  Interface当前的配置。
- int minor
  USB core分配的USB minor number。只有在成功调用usb_register_dev之后才生效。

### Configurations
由多个Interface组成一个Configurations。一个USB设备可以有多个Configuration，并且在多个Configuration之间切换，以切换设备状态。
例如某些设备支持下载Firmware到设备中，这种设备就需要有多个Configuration。Linux USB驱动同一时间只支持一个Configuration使能（Linux内核支持不太好，但幸运的是这种设备很少）。

Configuration由结构体structure struct usb_host_config描述。整个USB设备由struct usb_device描述。
这两个结构体，Linux内核很少会读写。

Enedpoint/Interface/Configuration/Usb device之间的关系总结：
- USB设备有一个或多个Configuration
- Configuration有一个或多个Interface
- Interface有一个或多个Settings
- Interface有0个或多个Endpoint

## USB对应的Sysfs
USB sysfs包含物理USB设备信息和USB Interface信息。
例如，一个USB鼠标，只有一个Interface，其Sysfs目录如下：
As an example, for a simple USB
```bash
/sys/devices/pci0000:00/0000:00:09.0/usb2/2-1
|-- 2-1:1.0
|   |-- bAlternateSetting
|   |-- bInterfaceClass
|   |-- bInterfaceNumber
|   |-- bInterfaceProtocol
|   |-- bInterfaceSubClass
|   |-- bNumEndpoints
|   |-- detach_state
|   |-- iInterface
|   `-- power
|       `-- state
|-- bConfigurationValue
|-- bDeviceClass
|-- bDeviceProtocol
|-- bDeviceSubClass
|-- bMaxPower
|-- bNumConfigurations
|-- bNumInterfaces
|-- bcdDevice
|-- bmAttributes
|-- detach_state
|-- devnum
|-- idProduct
|-- idVendor
|-- maxchild
|-- power
| `-- state
|-- speed
`-- version
```

结构体struct usb_device对应的目录：
    /sys/devices/pci0000:00/0000:00:09.0/usb2/2-1
与设备绑定的Interface对应的目录：
    /sys/devices/pci0000:00/0000:00:09.0/usb2/2-1/2-1:1.0
要了解这一长串的目录的意义，需要知道Linux内核是怎么标记USB设备。
第一个USB设备是root hub，这是一个USB控制器（USB Controller），一般包含在一个PCIe设备中。
之所以命名为USB Controller是因为其控制整个连接的USB bus。
USB控制器（USB Controller）是 PCI 总线和 USB 总线之间的桥接，同时也是该总线上第一个 USB 设备.
所有的root hub都被USB core赋予了一个number，上面的例子中就是usb2，就是说是第二个注册给USB core的root hub。
- usb2：就是一个USB bus。
- 2-1: USB设备的目录，第一个2是和USB bus的号是一样的，- 后面的1表示是USB设备插入的port号。
- 2-1:1.0：USB Interface目录，前面是和USB设备目录名字一样。1.0：前面的1表示是第一个Configuration，后面的0表示Interface号0。
USB interface的sysfs节点的值是可以被修改的，例如bConfigurationValue就可以被修改，以便修改USB Configuration行为。

其他的Endpoint相关信息，没有包含在sysfs中，而是包含在 /proc/bus/usb/ 目录下。
 /proc/bus/usb/ 目录下的内容也可以被修改，以便用户态USB驱动可以控制USB设备。

## Urb(USB Request Block)
### 介绍
用来从特定的Endpoint，以异步方式读写数据的数据结构。与文件系统异步IO代码中的kiocb结构体，网络代码中的skbuff一样。
USB驱动根据需求，可能给一个Endpoint创建多个Urb结构体或者多个Endpoint复用同一个Urb。
典型的Urb的生命周期如下：
- 被USB驱动创建
- 被指定给某个USB设备的某个Endpoint
- 被USB驱动提交给USB core
- 被USB host controller驱动处理（被发送给USB设备）
- 等到Urb结束，USB host controller会通知USB设备驱动
Urb提交可以被驱动和USB core随时取消（例如在USB设备被移除时）。

### Urb结构体(struct urb)
成员说明：
- struct usb_device *dev
    指向此 urb 所要发送的 struct usb_device 的指针。在 urb 可以发送到 USB 核心之前，此变量必须由 USB 驱动程序初始化。
- unsigned int pipe
    此 urb 将被发送到的特定 struct usb_device 的端点信息。此变量必须由 USB 驱动程序初始化，然后 urb 才能被发送到 USB 核心
    要设置此结构的字段，USB驱动要根据传输方向酌情使用如下函数中的一个。请注意，每个端点只能属于一种类型。  
    (1) unsigned int usb_sndctrlpipe(struct usb_device *dev, unsigned int endpoint) : control OUT endpoint 
    (2) unsigned int usb_rcvctrlpipe(struct usb_device *dev, unsigned int endpoint) : control IN endpoint
    (3) unsigned int usb_sndbulkpipe(struct usb_device *dev, unsigned int endpoint) : bulk OUT endpoint
    (4) unsigned int usb_rcvbulkpipe(struct usb_device *dev, unsigned int endpoint) : bulk IN endpoint
    (5) unsigned int usb_sndintpipe(struct usb_device *dev, unsigned int endpoint)  : interrupt OUT
    (6) unsigned int usb_rcvintpipe(struct usb_device *dev, unsigned int endpoint)  : interrupt IN
    (7) unsigned int usb_sndisocpipe(struct usb_device *dev, unsigned int endpoint)
    (8) unsigned int usb_rcvisocpipe(struct usb_device *dev, unsigned int endpoint)
- unsigned int transfer_flags：
    此变量可以设置为多个不同的位值，具体取决于USB 驱动程序希望对 urb 执行的操作。可用的值有：
    (1) URB_SHORT_NOT_OK：
        设置后，它指定任何可能发生的 IN 端点上的短读（Short Read）操作都应被 USB 核心视为错误。
        此值仅对要从 USB 设备读取的 urb 有用，对写入 urb 则无用
    (2) URB_ISO_ASAP：
- void *transfer_buffer：
    指向向设备发送数据（对于 OUT urb）或从设备接收数据（对于 IN urb）时要使用的缓冲区的指针。为了让主机控制器能够正确访问此缓冲区，必须通过调用 kmalloc 来创建它，而不是在堆栈上或静态地创建。对于控制端点，此缓冲区用于传输的数据阶段。
- dma_addr_t transfer_dma：
    用于通过 DMA 将数据传输到 USB 设备的缓冲区。
-int transfer_buffer_length：
    对于 OUT 端点，如果端点最大值小于此变量中指定的值，则到 USB 设备的传输将被分解为较小的块，以便正确传输数据。这种大型传输发生在连续的 USB 帧中。在一个 urb 中提交大量数据，并让 USB 主机控制器将其拆分成较小的块，比按连续顺序发送较小的缓冲区要快得多。

### 创建Urb
### 发送Urb
### Urb发送结束
### Urb取消

## USB Driver
### USB Driver支持哪些设备
struct usb_device_id 结构提供了此驱动程序支持的不同类型的 USB 设备的列表。USB core使用此列表来决定将设备交给哪个驱动程序，热插拔脚本则使用此列表来决定在特定设备插入系统时自动加载哪个驱动程序。
struct usb_device_id结构体成员如下：
- __u16 match_flags：
    确定设备应与结构中的以下哪个字段匹配。这是由 include/linux/mod_devicetable.h 文件中指定的不同 USB_DEVICE_ID_MATCH_*值定义的位字段。此字段通常不会直接设置，而是由后面描述的 USB_DEVICE 类型宏初始化。
__u16 idVendor：
    设备的 USB Vendor ID。此编号由 USB Forum分配给其成员，其他人无法编造。
__u16 idProduct：
    设备的 USB 产品 ID。所有已分配Vendor ID 的供应商都可以按照自己的选择管理其产品 ID。

## 参考
https://docs.kernel.org/driver-api/usb/writing_usb_driver.html

LINUX下USB驱动程序的设计与实现 pdf

https://cloud.tencent.cn/developer/information/linux%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91%E5%AE%9E%E4%BE%8Bpdf
