---
title: Linux DeviceTree
categories: 
- Linux Driver
tags:
- Linux Driver
---

## DTS的意义
以前每个Platform设备都需要在代码中定义platofm-device-xxx.c文件被Linus那个人骂了，说像痔疮。
DTS就是为了解决这个问题，使得Linux代码中不再一个一个定义Platform设备。
DTS出现之前，Bootloader使用ATAGS方式将设备类型之类的发送给内核，但现在这些都成了历史。

## DTS加载
1. 普通的加载
DTS出现之后，Bootloader需要加载两个东西。一个是Linux内核，一个是Dtb文件。加载之后用r2寄存器（32bit arm上）传dtb文件基地址给内核。
![device-tree加载](/images/驱动/device-tree加载.png)
2. 另外的方式
有些bootloader太老了不支持dts，则可以使能CONFIG_ARM_APPENDED_DTB。这个宏高速内核，紧接着内核后面的地址就是dtb的地址。
这种方式没有现成的Makefile可以编译打包内核Image和dtb文件，需要按照如下方式手动打包。
```bash
cat arch/arm/boot/zImage arch/arm/boot/dts/myboard.dtb > my-zImage
mkimage ... -d my-zImage my-uImage
```
还有CONFIG_ARM_ATAG_DTB_COMPAT，这个宏告诉内核从ATAGS中读取bootloader传上来的信息，并使用这些更新DTS。

但上面说的方式都只在arm32存在，arm64上都没有了，都是一些过渡性的方案。

## 编译与反编译命令
1. 编译命令
```bash
dtc -O dtb -o dest.dtb src.dts
```

2. 反编译命令
```bash
dtc -I dtb -O dts src.dtb > des.dts
```
编译工具的代码在scripts/dtc目录下。

## DTS相关目录
1. /sys/firmware/fdt
   原始dtb文件，如下命令可以查看dtb文内容
   hexdump -C /sys/firmware/fdt
2. /sys/firmware/devicetree
   以目录结构程现的dtb文件, 根节点对应base目录, 每一个节点对应一个目录, 每一个属性对应一个文件。这个文件很好，可以看到所有的dts文件配置都对应生成了哪些。
3. /sys/devices/platform
   系统中所有的platform_device, 有来自设备树的, 也有来有.c文件中注册的，对于来自设备树的platform_device, 可以进入 /sys/devices/platform/<设备名>/of_node 查看它的设备树属性。
4. /proc/device-tree
   链接文件, /sys/firmware/devicetree/base

https://www.cnblogs.com/multimicro/p/11905647.html

## DTS脚本基本
打开DTS脚本，有如下几个典型的最上层目录：
1. cpu：描述系统中的cpu信息
2. memory：描述系统中的内存，例如基地址，大小等
3. chosen：
   有两个用途
   (1) 在启动阶段传递参数给内核
   (2) 传递command line给内核
4. aliases：定义的别名，例如有些node名字太长了，可以在这里定义一个简短的别名

## 使用举例
### 预留内存
#### reserved-memory方式
1. 在reserve-memory区域添加要预留的内存。
   下面就是在该区域添加了reserve:buffer@0区域。no-map表示该区域不要被映射进内核。预留后/proc/iomem也显示System RAM区域小于内存量。
```c
reserved-memory {
  #address-cells = <2>;
  #size-cells = <2>;
  ranges;
  reserved: buffer@0 {
    no-map;
    reg = <0x0 0x70000000 0x0 0x10000000>;
  };
};
```
2. 在驱动dts节点添加引用
```c
reserved-driver@0 {
  compatible = "xlnx,reserved-memory";
  memory-region = <&reserved>;
};
```
3. 在驱动代码中读驱动dts节点并使用
```c
/* Get reserved memory region from Device-tree */
np = of_parse_phandle(dev->of_node, "memory-region", 0);
if (!np) {
  dev_err(dev, "No %s specified\n", "memory-region");
  goto error1;
}
  
rc = of_address_to_resource(np, 0, &r);
if (rc) {
  dev_err(dev, "No memory address assigned to the region\n");
  goto error1;
}
  
lp->paddr = r.start;
lp->vaddr = memremap(r.start, resource_size(&r), MEMREMAP_WB);
dev_info(dev, "Allocated reserved memory, vaddr: 0x%0llX, paddr: 0x%0llX\n", 
(u64)lp->vaddr, lp->paddr);
```

#### 通过DMA API预留
```c
reserved-memory {
   #address-cells = <2>;
   #size-cells = <2>;
   ranges;
   
   reserved: buffer@0 {
      compatible = "shared-dma-pool";
      no-map;
      reg = <0x0 0x70000000 0x0 0x10000000>;
   };
};
   
reserved-driver@0 {
   compatible = "xlnx,reserved-memory";
   memory-region = <&reserved>;
};
```
然后在驱动代码中使用dma接口申请。
```c
/* Initialize reserved memory resources */
rc = of_reserved_mem_device_init(dev);
if(rc) {
   dev_err(dev, "Could not get reserved memory\n");
   goto error1;
}
  
/* Allocate memory */
dma_set_coherent_mask(dev, 0xFFFFFFFF);
lp->vaddr = dma_alloc_coherent(dev, ALLOC_SIZE, &lp->paddr, GFP_KERNEL);
dev_info(dev, "Allocated coherent memory, vaddr: 0x%0llX, paddr: 0x%0llX\n", 
(u64)lp->vaddr, lp->paddr);
```

#### CMA内存预留
CMA跟上面DMA API预留方式一致，但多了两个属性。
需要的属性：
reusable;
linux,cma-default
```
reserved-memory {
      #address-cells = <2>;
      #size-cells = <2>;
      ranges;
 
      reserved: buffer@0 {
         compatible = "shared-dma-pool";
         reusable;
         reg = <0x0 0x70000000 0x0 0x10000000>;
         linux,cma-default;
      };
   };
```
这样在设备启动的时候会显示：
```
[    0.000000] Reserved memory: created CMA memory pool at 0x0000000070000000, size 
256 MiB
[    0.000000] Reserved memory: initialized node buffer@0, compatible id shared-dma-
pool
```

### simple-bus关键字

### pinctrl
1. pinctrl子系统支持管理pin脚复用。如果某个设备需要复用管脚，那在DTS中就需要定义pinctrl。
2. pinctrl-<n>支持定义一组pinctrl配置。
3. pinctrl-names可以给每个pinctrl配置定义名字。
4. 设备probe之后，default的pinctrl就会被默认配置。
```bash
   ssp0: ssp@80010000 {
      pinctrl-names = "default";
      pinctrl-0 = <&mmc0_8bit_pins_a &mmc0_cd_cfg &mmc0_sck_cfg>;
      [...]
   };
```
5. pinctrl配置项
```bash
mmc0_8bit_pins_a: mmc0-8bit@0 {
  fsl,pinmux-ids = <
    0x2000 /* MX28_PAD_SSP0_DATA0__SSP0_D0 */
    0x2010 /* MX28_PAD_SSP0_DATA1__SSP0_D1 */
    [...]
    0x2090 /* MX28_PAD_SSP0_DETECT__SSP0_... */
    0x20a0 /* MX28_PAD_SSP0_SCK__SSP0_SCK */
  >;
  fsl,drive-strength = <1>;
  fsl,voltage = <1>;
  fsl,pull-up = <1>;
};
```

### 中断配置
```
drv@0xabadf {
   #interrupt-cells = <0x3>;
   interrupts = <0 30 1>;
   interrupt-names = "xx";
};
```
- interrupt-cells：表示interrupt由几组数字组成
- interrupts：<0 30 0>根据interrupt-cells的定义，一个interrupt由3组数字组成
  第一个数字：表示中断类型，0表示spi，1表示ppi，其他值留着以后用
  第二个数字：表示中断号
    (1) SPI中断：中断号从0-987，spin中断号要注意的一点是，写到dts里边的中断号要在实际的硬件中断号中-32
    (2) PPI中断：中断号从16-31，PPI中断是每个CPU一份的。
  第三个数字：表示触发方式
    (1) 1表示边沿触发
    (2) 4表示电平触发
  SGI中断，DTS是不支持的，没有SGI中断的相关DTS配置。
疑问：有些设备没有定义interrupt-cells，且没有定义interrupt-parent。这种设备应该是直接继承gic？
例如在dts文件中会有：
```
interrupt-controller@abc0 {
   #interrupt-cells = <0x3>;
}
```
- interrupt-names：可以给每个中断起名字
- interrupt-map：
https://stackoverflow.com/questions/69849394/in-device-tree-a-nodes-interrupts-output-cell-size-is-1-but-its-interrupt-pare
