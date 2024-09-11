

## 函数
echo -n "func xxx +p" > /sys/kernel/debug/dynamic_debug/control 

## 文件
echo "file aa.c +p" > /sys/kernel/debug/dynamic_debug/control

## 参考
https://wiki.stmicroelectronics.cn/stm32mpu/wiki/How_to_use_the_kernel_dynamic_debug
