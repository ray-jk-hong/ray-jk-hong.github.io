---
title: Linux Usb
categories: 
- Linux Driver
tags:
- Linux Driver
---

## USB基础
### 设备与驱动基础架构
USB设备大致可以分为USB驱动，USB设备。
下图大致描绘了USB设备的组成部分（configuration, interface, endpoints），并描绘了USB驱动是怎么绑定到interface上的（USB驱动不是绑定到整个USB设备上，是绑定到interface上）。
![Usb设备架构](/images/Usb/Usb设备大致结构.png)

lsusb -vvv查看usb设备，是按照如下架构打印的
```
+-- Device Descriptor:
    +-- Configuration Descriptor:
        +-- Interface Descriptor:
            +-- Endpoint Descriptor:
```

下图描绘了USB驱动的大致结构
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

note left of usb_endpoint_descriptor::bInterval
end note
{% endplantuml %}

## 参考
https://docs.kernel.org/driver-api/usb/writing_usb_driver.html

LINUX下USB驱动程序的设计与实现 pdf

https://cloud.tencent.cn/developer/information/linux%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91%E5%AE%9E%E4%BE%8Bpdf
