---
title: Linux GICv3中断控制器代码
categories: 
- Linux 中断
tags:
- Linux 中断
---

## irq-domain/irq-chip/interrupt-controller关系
![irqdomain-irqcontroller关系](/images/中断/中断控制器-irqdomain关系.png)

### irq domain
中断系统中出现多个中断控制器，每个中断控制器对中断号都独立编号（HW interrupt ID），这时多个中断控制器就会出现中断号重复的情况。如果这样，注册中断不仅需要知道中断号（HW interrupt ID），而且需要知道是哪个中断控制器，irq domain就是为了解决这个问题。有了irq domain我们只需要知道虚拟的i中断号（IRQ number）就可以了。每个interrupt-controller可以，也可以没有自己的irq domain，就看你需不需要将自己的interrupt-controller的中断号转成硬件中断号了。例如gpio这个interrupt-controller，为了方便，中断号都是以gpio号替换掉的，这个时候gpio interrupt-controller就需要有一个irq domain将gpio号转成硬件中断号。

### interrupt-controlle
所谓interrupt-controller就是那些实际产生中断的设备或者芯片IP。例如gic，gpio这些都是一个interrupt-controller。interrupt-controller之间是可以级联的，例如gpio这个interrrup-controller就是级联到gic这个interrupt-controller。

### irq chip
是直接给CPU中断的interrupt-controller，其他的就不需要定义了。

### 应用

![max7325](/images/中断/max7325-1.png)
像max7325芯片ip，它有8外部输入，并与一个产生中断的端口可以连到gpio，另外SCL/SDA就是一个i2c链路了。当P0-P7中某个管脚电平变化时，就可以拉高gpio管脚产生中断，并通过i2c链路读到是哪个管脚发生的。
因为这个芯片ip是产生中断的设备，所以就是一个interrpt-controller。

考虑如下场景：
![max7325](/images/中断/max7325-2.png)
一个设备接到了P4上，拉高了P4管脚，这时max7325会将gpio4_29拉高触发对应中断给cpu。所以max7325就是级联到gpio interrupt-controller上的一个interrupt-controller。gpio interrupt-controller又是级联到gic interrupt-controller.
```
i2c@xxx {
    xxx
    expander: max7325@6d {
        compatible = "maxim,max7325";
        reg = <0x6d>;

        gpio-controller;
        #gpio-cells = <2>;

        interrupt-controller;
        #interrupt-cells = <2>;

        interrupt-parent = <&gpio4>;
        interrupts = <29 IRQ_TYPE_EDGE_FALLING>;
    };
};
```
上面DTS的属性：
- interrupt-controller：表示是一个中断控制器。只有定义了这个，别的设备才能在interrup-parent中加上并使用。
- #interrupt-cells：表示interrupts = <xx>分为几段。
- interrupt-parent：表示max7325是级联到哪里去了。因为现在是级联到gpio4上的，所以这里填的就是gpio4。
- interrupts = <29 IRQ_TYPE_EDGE_FALLING>：max7325驱动需要注册interrupt-parent=<&gpio4>的29这个中断，因为芯片连线就是这么连的。
这里说明一下, expander是放到i2c里边的，因为其是一个i2c的设备。像这样做完之后，其实是不需要额外注册i2c board info了。可以看一下drivers/gpio/gpio-max732x.c文件的probe函数。
```c
static int max732x_probe(struct i2c_client *client)
{
    pdata = dev_get_platdata(&client->dev); // 由于没有注册i2c board info，pdata是空
	node = client->dev.of_node; // node是非空的

	if (!pdata && node) // 由于pdata是空，node是非空，所以会创建一个pdata。由于从dts读i2c信息的时候，gpio_base可以要赋值成负值
		pdata = of_gpio_max732x(&client->dev);
    ret = devm_gpiochip_add_data(xx); // 动态分配一个gpio_base，，这个都不重要了，后面都不会用到
    xx

    /*
     * client->irq就是上面dts里边定义的，，在启动的时候，读取interrupt并赋值给client->irq的。
     * ???? i2c下面有多个设备怎么办?? 一个client结构体是对应i2c dts下面的一个块。
     * interrupt-parent = <&gpio4>; 
     * interrupts = <29 >;
     */
    ret = request_irq(client->irq, xx, ); IRQ_TYPE_EDGE_FALLING>;

    return 0;
}

int gpiochip_add_data_with_key(xxx)
{
    base = gc->base;
    if (base < 0) { // 上面gpio_base是负数，所以这里会动态分配。
		base = gpiochip_find_base(gc->ngpio);
    xxx
    return 0;
}

```

对于最左边的"Some device"，我们也要定义DTS几点，并有相应的驱动。DTS定义如下：
```
some_device: some_device@1c {
    reg = <0x1c>;
    interrupt-parent = <&expander>;
    interrupts = <4 IRQ_TYPE_EDGE_RISING>;
};
```

中断的传播：
有了上面的DTS定义，我们在"Some device"的驱动里边可以注册中断了。
```c
devm_request_threaded_irq(core->dev, core->gpio_irq, NULL,
        some_device_isr, IRQF_TRIGGER_RISING | IRQF_ONESHOT,
        dev_name(core->dev), core);
```
但此中断不是gic直接触发的，具体怎么做到的？
从硬件角度看，max7325触发中断通知到cpu的流程如下：
1. max7325的p4端口电平变化
2. max7325感知到电平变化，改变INT管脚的电平
3. GPIO4模块配置感知到电平变化，并触发中断给GIC
4. GIC通知CPU

从软件角度，"Some device"中断回调被调用的流程，跟上面的硬件流程正好相反。
软件流程如下：
1. CPU 现在处于 GIC 中断处理程序的中断上下文中。从 gic_handle_irq() 调用handle_domain_irq()，后者又调用 generic_handle_irq()。参考Documentation/gpio/driver.txt。现在我们在Soc的GPIO控制器的中断上下文。
2. Soc的GPIO驱动也会调用generic_handle_irq()去触发对应pin脚的中断回调。可以看omap_gpio_irq_handler()是怎么做的。现在我们进入到了max7325中断回调函数
3. max7325中断回调函数max732x_irq_handler()会调用handle_nested_irq()来触发"Some device"的中断回调
4. 最终，"Some device"的中断回调被调用

irq domain：
gpio和max7325都是interrupt controller，都使用了irq domain方式来把硬中断号转成软中断并触发下一级interrupt-controller的中断回调函数。
max732x_irq_handler()函数中有这样一行代码：
```c
handle_nested_irq(irq_find_mapping(chip->gpio_chip.irqdomain, level));
```
这行代码先使用irq_find_mapping()函数将硬中断号转成软件中断号，然后使用handle_nested_irq() 将"Some device"驱动的对应中断回调函数调用起来。
max7325使用irq domain的提交（https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=479f8a5744d8141e95ef40ab364ae2d3648848ef）

疑问：generic_handle_irq函数和haldle_nested_irq函数有什么区别
答案：https://stackoverflow.com/questions/34377846/what-is-chained-irq-in-linux-when-are-they-need-to-used

疑问：interrupts=<中断号 >，这些中断号是怎么一层一层传递到irq domain中，然后在回调的时候被一层一层解析出来的？
这个可以看drivers/of/irq.c文件的of_irq_init函数的处理，，看怎么把interrupt-controller信息汇集的。? 
这个只是gic之类的irq chip的解析。其他的都是在platform_get_irq的时候，动态解析并注册生成irq domain的。
https://blog.csdn.net/jasonactions/article/details/115541386

疑问：系统启动的时候，是怎么读取所有的interrupt controller然后把这些关系都组织好的？包括interrupt-parent也是在启动的时候都读完，建立关系并组织irq domain。
[drivers/of/irq.c] of_irq_parse_raw()

## interrupt-controller和interrupt-parent指定和级联示意图
![controller-parent示意图](/images/中断/controller-parent示意图.png)

https://stackoverflow.com/questions/26667082/max732x-c-i2c-io-expander-gpio-keys-w-linux-device-tree-not-working
https://stackoverflow.com/questions/34371352/what-are-linux-irq-domains-why-are-they-needed
https://stackoverflow.com/questions/34377846/what-is-chained-irq-in-linux-when-are-they-need-to-used

## 参考

https://blog.csdn.net/qq_16106195/article/details/135276562

https://cloud.tencent.com/developer/article/1867927

http://www.wowotech.net/irq_subsystem/gic-irq-chip-driver.html

https://android.googlesource.com/kernel/msm/+/android-msm-bullhead-3.10-marshmallow-dr/Documentation/IRQ-domain.txt

http://www.wowotech.net/irq_subsystem/irq-domain.html
