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


## 参考
https://docs.kernel.org/driver-api/usb/writing_usb_driver.html

LINUX下USB驱动程序的设计与实现 pdf

https://cloud.tencent.cn/developer/information/linux%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91%E5%AE%9E%E4%BE%8Bpdf
