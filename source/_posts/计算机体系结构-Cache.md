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
### 2.1	Cache基本结构
#### 2.1.1 Cache Memory
简单的Cache内存如下所示，由三个部分组成：
1)	Directory store（Cache-tag）
Cache数量极为有限，需要根据地址进行换算找到对应的Cache-tag并进行访问。
2)	Data Section
保存主存的数据内容，一般大小称之为Cache line大小。（当前看到的一般为64个字节）
3)	Status information
由Valid/Dirty两个比特位组成。
Valid比特位显示当前Cache-Line是否已经被分配使用。
Dirty比特位显示当前Cache-Line是否与下级Cache或者主存保持数据一致性。
![image](https://github.com/user-attachments/assets/2d37a4af-9ff9-4d13-bfbb-4747dacd30cd)

#### 2.1.2	Cache Controller
Cache Controller软件不可见，但其执行大部分Cache line查找，判定Cache line合法性，替换Cache数据等
操作。
Cache Controller会拦截CPU的读写请求，并将地址划分为Tag, Set index, Data index。首先使用Set index
寻找Cache line，根据status比特位查看Cache line数据是否为最新并对比地址中的Tag和Cache-tag。如果
Tag对比成功且Status比特位为Valid，就是Cache hit。两个条件有一个不满足成为Cache miss。
Cache miss时会把整个Cache line的数据从主存中拷贝到Cache line中（Cache Fill）。
Cache hit时会进一步根据Data index进一步找到Cache line中对应的字节。

https://zhuanlan.zhihu.com/p/694673551
