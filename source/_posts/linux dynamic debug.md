
## 需要使能的宏

## 函数
echo -n "func xxx +p" > /sys/kernel/debug/dynamic_debug/control 

## 文件
echo "file aa.c +p" > /sys/kernel/debug/dynamic_debug/control

## module
要把aa.ko模块的所有debug日志打开，可以使用如下命令：
echo "module aa +p" > /sys/kernel/debug/dynamic_debug/control

## 参考
https://wiki.stmicroelectronics.cn/stm32mpu/wiki/How_to_use_the_kernel_dynamic_debug
