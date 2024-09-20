## 1. 内存层次
### 1.1	典型的ARM CPU存储系统
CPU与主存之间有几层Cache用于缓存
![image](https://github.com/user-attachments/assets/915dbaaa-607a-4606-8274-d80d77a82061)
不同层级的Cache数据共享不同：
1)	一个CPU core独享L1 Cache，不与其他CPU core共享
2)	Cluster内部，CPU core之间共享L2 Cache
3)	不同Cluster或者外设之间，只共享L3 Cache

不同层级的Cache访问效率不同：
![image](https://github.com/user-attachments/assets/d55cb2de-cc3b-45f1-8032-c0a19c93a8f6)

### 1.2	Cache与MMU/TLB
![image](https://github.com/user-attachments/assets/631b0dcd-f4f5-4f0b-b7ca-c95b5fd14423)

在支持虚拟地址的芯片，Cache可以处于不同的位置，分成以下两种
1)	Logical Cache（Virtual Cache）：Cache处于CPU和MMU之间
2)	Physical Cache：Cache处于MMU和主存之间

## 2. Cache架构

https://zhuanlan.zhihu.com/p/694673551
