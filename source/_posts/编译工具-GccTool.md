---
title: Gcc Tool
categories: 
- 编译
tags:
- 编译
---
## Summary
ar, nm, objdump, addr2line

## ar
库文件的操作指令
经常用法：

```bash
ar -t libname.a // 显示所有对象文件(.o文件)的列表.例： # ar t libtest.a
libtest1.o
libtest2.o
ar -rv libname.a  objfile1.o objfile2.o ... objfilen.o  // 把objfile1.o--objfilen.o打包成一个库文件
```

ar -help打印：
- d：从库中删除模块。按模块原来的文件名指定要删除的模块。如果使用了任选项v则列出被删除的每个模块。
- m：该操作是在一个库中移动成员。当库中如果有若干模块有相同的符号定义(如函数定义)，则成员的位置顺序很重要。如果没有指定任选项，任何指定的成员将移到库的最后。也可以使用'a'，'b'，或'I'任选项移动到指定的位置。
- p：显示库中指定的成员到标准输出。如果指定任选项v，则在输出成员的内容前，将显示成员的名字。如果没有指定成员的名字，所有库中的文件将显示出来。
- q：快速追加。增加新模块到库的结尾处。并不检查是否需要替换。'a'，'b'，或'I'任选项对此操作没有影响，模块总是追加的库的结尾处。如果使用了任选项v则列出每个模块。 这时，库的符号表没有更新，可以用'ar s'或ranlib来更新库的符号表索引。
- r：在库中插入模块(替换)。当插入的模块名已经在库中存在，则替换同名的模块。如果若干模块中有一个模块在库中不存在，ar显示一个错误消息，并不替换其他同名模块。默认的情况下，新的成员增加在库的结尾处，可以使用其他任选项来改变增加的位置。
- t：显示库的模块表清单。一般只显示模块名。
- x：从库中提取一个成员。如果不指定要提取的模块，则提取库中所有的模块。

下面在看看可与操作选项结合使用的任选项：
- a：在库的一个已经存在的成员后面增加一个新的文件。如果使用任选项a，则应该为命令行中membername参数指定一个已经存在的成员名。
- b：在库的一个已经存在的成员前面增加一个新的文件。如果使用任选项b，则应该为命令行中membername参数指定一个已经存在的成员名。
- c：创建一个库。不管库是否存在，都将创建。
- f：在库中截短指定的名字。缺省情况下，文件名的长度是不受限制的，可以使用此参数将文件名截短，以保证与其它系统的兼容。
- i：在库的一个已经存在的成员前面增加一个新的文件。如果使用任选项i，则应该为命令行中membername参数指定一个已经存在的成员名(类似任选项b)。
- l：暂未使用
- N：与count参数一起使用，在库中有多个相同的文件名时指定提取或输出的个数。
- o：当提取成员时，保留成员的原始数据。如果不指定该任选项，则提取出的模块的时间将标为提取出的时间。
- P：进行文件名匹配时使用全路径名。ar在创建库时不能使用全路径名（这样的库文件不符合POSIX标准），但是有些工具可以。
- s：写入一个目标文件索引到库中，或者更新一个存在的目标文件索引。甚至对于没有任何变化的库也作该动作。对一个库做ar s等同于对该库做ranlib。
- S：不创建目标文件索引，这在创建较大的库时能加快时间。
- u：一般说来，命令ar r...插入所有列出的文件到库中，如果你只想插入列出文件中那些比库中同名文件新的文件，就可以使用该任选项。该任选项只用于r操作选项。
- v：该选项用来显示执行操作选项的附加信息。
- V：显示ar的版本.

## nm
列出目标文件的符号清单
```bash
nm -s filename.a或者filename.o或者filename.out  
```

nm -help打印

- -a或--debug-syms：显示调试符号。
- -B：等同于--format=bsd，用来兼容MIPS的nm。
- -C或--demangle：将低级符号名解码(demangle)成用户级名字。这样可以使得C++函数名具有可读性。
- -D或--dynamic：显示动态符号。该任选项仅对于动态目标(例如特定类型的共享库)有意义。
- -f format：使用format格式输出。format可以选取bsd、sysv或posix，该选项在GNU的nm中有用。默认为bsd。
- -g或--extern-only：仅显示外部符号。
- -n、-v或--numeric-sort：按符号对应地址的顺序排序，而非按符号名的字符顺序。
- -p或--no-sort：按目标文件中遇到的符号顺序显示，不排序。
- -P或--portability：使用POSIX.2标准输出格式代替默认的输出格式。等同于使用任选项-f posix。
- -s或--print-armap：当列出库中成员的符号时，包含索引。索引的内容包含：哪些模块包含哪些名字的映射。
- -r或--reverse-sort：反转排序的顺序(例如，升序变为降序)。
- --size-sort：按大小排列符号顺序。该大小是按照一个符号的值与它下一个符号的值进行计算的。
- -t radix或--radix=radix：使用radix进制显示符号值。radix只能为"d"表示十进制、"o"表示八进制或"x"表示十六进制。
- --target=bfdname：指定一个目标代码的格式，而非使用系统的默认格式。
- -u或--undefined-only：仅显示没有定义的符号(那些外部符号)。
- -l或--line-numbers：对每个符号，使用调试信息来试图找到文件名和行号。对于已定义的符号，查找符号地址的行号。对于未定义符号，查找指向符号重定位入口的行号。如果可以找到行号信息，显示在符号信息之后。
- -V或--version：显示nm的版本号。
- --help：显示nm的任选项。

## objdump
常用命令：
1. objdump -h file<.o,.a,.out> // 查看对象文件所有的节sections.例如
   #objdump -h libtest1.o
```bash
   libtest1.o:     file format elf32-i386
   Sections:
   Idx Name          Size      VMA       LMA       File off  Algn
     0 .text         00000014  00000000  00000000  00000034  2**2
                     CONTENTS, ALLOC, LOAD, RELOC, READONLY, CODE
     1 .data         00000000  00000000  00000000  00000048  2**2
                     CONTENTS, ALLOC, LOAD, DATA
     2 .bss          00000000  00000000  00000000  00000048  2**2
                     ALLOC
     3 .rodata       0000000e  00000000  00000000  00000048  2**0
                     CONTENTS, ALLOC, LOAD, READONLY, DATA
     4 .comment      0000001f  00000000  00000000  00000056  2**0
                     CONTENTS, READONLY
     5 .note.GNU-stack 00000000  00000000  00000000  00000075  2**0
                     CONTENTS, READONLY
```
2. objdump -t 查看对象文件所有的符号列表，相当于 nm -s objfilename,如：
#objdump -t libtest1.o
```bash
   libtest1.o:     file format elf32-i386
   
   SYMBOL TABLE:
   00000000 l    df *ABS*  00000000 libtest1.c
   00000000 l    d  .text  00000000 .text
   00000000 l    d  .data  00000000 .data
   00000000 l    d  .bss   00000000 .bss
   00000000 l    d  .rodata        00000000 .rodata
   00000000 l    d  .note.GNU-stack        00000000 .note.GNU-stack
   00000000 l    d  .comment       00000000 .comment
   00000000 g     F .text  00000014 print_test1
   00000000         *UND*  00000000 puts
```

## addr2line
将可执行文件反汇编并打印对应文件的行号。

### 内核模块
查看oops打印，看到挂死在xxx模块中，函数是func_xxx，偏移是0x100/0x200。
我们可以按照如下方法打印具体的行号。
1. 反汇编
   aarch64-linux-gnu-objdump -D xxx.o > out.txt
   在out.txt中找到func_xxx的开始地址。假设开始地址是0x800。
2. 计算挂死的地址
   0x800 + 0x100 = 0x900。0x800是函数开始地址，0x100是偏移
3. 打印
   aarch64-linux-gnu-addr2line -i -C -f -e xxx.o **900**
### 内核
跟上面一样，只需要将xxx.o文件替换成vmlinux就可以了。

## 参考

https://blog.csdn.net/longbei9029/article/details/76397089
