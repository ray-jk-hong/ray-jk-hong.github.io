---
title: DeviceTree
categories: 
- Linux
tags:
- Linux DeviceTree
---

## 编译与反编译命令
1. 编译命令
dtc -O dtb -o dest.dtb src.dts

2. 反编译命令
dtc -I dtb -O dts src.dtb > des.dts

## 预留内存
### reserved-memory方式
1. 在reserve-memoyr区域添加要预留的内存。
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

### 通过DMA API预留
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


### CMA内存预留
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