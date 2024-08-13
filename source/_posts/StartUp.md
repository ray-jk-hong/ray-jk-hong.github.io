---
title: StartUp
categories: 
- Linux
tags:
- Linux StartUp
---

## 内核入口
内核入口需要查看lds链接脚本。KBUILD_LDS定义了链接脚本的路径：arch/$(SRCARCH)/kernel/vmlinux.lds。
```c
# Linker scripts preprocessor (.lds.S -> .lds)
# ---------------------------------------------------------------------------
quiet_cmd_cpp_lds_S = LDS     $@
      cmd_cpp_lds_S = $(CPP) $(cpp_flags) -P -U$(ARCH) \
                             -D__ASSEMBLY__ -DLINKER_SCRIPT -o $@ $<

$(obj)/%.lds: $(src)/%.lds.S FORCE
        $(call if_changed_dep,cpp_lds_S)
```

从链接脚本 arch/arm64/kernel/vmlinux.lds可以查到，程序的入口为 _text，镜像起始位置存放的是 .head.text段生成的指令
```c [arch/arm64/mm/proc.S]
OUTPUT_ARCH(aarch64)
ENTRY(_text)

SECTIONS
{
 . = ((((((-(((1)) << ((((48))) - 1)))) + (0x08000000))) + (0x08000000)));
 .head.text : {
  _text = .;
  KEEP(*(.head.text))
 }
 ...
}
```

搜索 .head.text，可以找到 include/linux/init.h对 __HEAD定义 .section ".head.text","ax"

```c
/* For assembly routines */
#define __HEAD      .section    ".head.text","ax"
#define __INIT      .section    ".init.text","ax"
#define __FINIT     .previous
```

通过搜索 __HEAD，可以看到程序起始代码位于 arch/arm64/kernel/head.S

## 从入口到start_kernel
```
+-- _text()                                 /// 内核启动入口
    \-- primary_entry()
        +-- preserve_boot_args()            /// 保存x0~x3到boot_args[0~3]
        +-- init_kernel_el()                /// 根据内核运行异常等级进行配置，返回启动模式
        |   +-- init_el1()                  /// 通常情况下从EL1启动内核
        |   \-- init_el2()                  /// 从EL2启动内核，用于开启VHE(Virtualization Host Extensions)
        +-- create_idmap()                  /// 建立恒等映射init_idmap_pg_dir和内核镜像映射init_pg_dir的页表
        +-- __cpu_setup()                   /// 为开启MMU做的CPU初始化
        \-- __primary_switch()
            +-- __enable_mmu()              /// 开启MMU，将init_idmap_pg_dir加载到ttbr0，reserved_pg_dir加载到ttbr1
            +-- clear_page_tables()         /// 清空init_pg_dir
            +-- create_kernel_mapping()     /// 填充init_pg_dir
            +-- load_ttbr1()                /// 将init_pg_dir加载到ttbr1
            \-- __primary_switched()        /// 初始化init_task栈，设置VBAR_EL1，保存FDT地址，计算kimage_voffset，清空bss段
                +-- set_cpu_boot_mode_flag()/// 设置__boot_cpu_mode变量
                +-- early_fdt_map()
                |   +-- early_fixmap_init() /// 尝试建立fixmap的页表，可能失败，后边init_feature_override会用到
                |   \-- fixmap_remap_fdt()  /// 如果成功建立fixmap页表，将fdt映射到fixmap的FIX_FDT区域
                +-- init_feature_override() /// 根据BootLoader传入的bootargs，对一些参数的改写
                +-- finalise_el2()          /// Prefer VHE if possible
                \-- start_kernel()          /// 跳转到start_kernel执行
```

## 参考

https://blog.csdn.net/u014001096/article/details/131342636
