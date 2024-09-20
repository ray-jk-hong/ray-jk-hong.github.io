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

### 2.2 Cache映射方式
#### 2.2.1 直接映射方式（Direct-mapped）
Direct-mapped cache方式的Cache，在CPU访问数据时，是怎么先通过Cache并访问到主存的。
![image](https://github.com/user-attachments/assets/58987580-7737-457c-8878-d8e69f996f2d)
如上图所示，假设我们有很多以0x824结束的地址需要访问主存。根据前一节所述，这些访问都会找到同一
个Cache line。如果Status比特位是valid但Tag对比失败，意味着此Cache line中的数据是其他以0x824结束
的地址访问的数据。这时我们就需要将Cache line中的数据同步回主存并替换成新的数据。（此过程称之
为Eviction）。
direct-mapped cache方式虽然简单，但有可能会造成Thrashing。例如两个变量使用同一个Cache line的
场景，会有很高的Cache miss率。如下图所示，以0x480结束的两个变量在同一个流程中读写的时候，会出
现Cache miss。
![image](https://github.com/user-attachments/assets/cb3a2925-d399-47af-af27-d9ae9b81f069)
一个解决方法就是所谓的组相联（Set-Associtive）方式，就是将Set index位数减少2位，增加Tag的增加2
位，将Cache分成4个Way。即使Set index一样，也能在4个Way中对比Tag并找到合适的Cache line，只要
4个Way中能找到一个就算Cache hit。
![Uploading image.png…]()
这种方式能够减少Cache miss率，但带来的后果就是芯片复杂度。为了进一步减少Cache miss率，还有
例如全相联（Full-associative）方式，其优点是Cache miss少，缺点是查找慢，不利于Cache做大。
#### 2.2.2 全相联映射（Full-associative mapped）
Cache-line可以对应所有地址
#### 2.2.3	组相联映射（Set-associative mapped）

## 3. Cache策略
### 3.1 参考

### 3.2	Cache数据更新策略
#### 3.2.1	按CPU写数据时传播到主存的时机分类
1. 写回（Write Back）
   当CPU采取写回策略时，对缓存的修改不会立刻传播到主存而只更新Cache中的数据并标记Cache-Line为Dirty。只有当缓存块被替换时，这些被修改的缓存块，才会写回并覆盖内存中过时的数据。
2. 写直达（Write Through）
  当CPU采取写直达策略时，缓存中任何一个字节的修改，都会立刻传播到内存。Cache-Line本身不会被标记位Dirty。 
#### 3.2.2	按CPU写数据时其他CPU的更新时机分类（监听一致性协议分类）
1. 写更新
  CPU每次写数据都发起一次总线请求，通知其他CPU的Cache更新数据。
  优点：多CPU读写同一地址数据频繁时，性能好。
  缺点：占用总线带宽较多
2. 写无效
  CPU每次写数据都将其他CPU的Cache置位无效。其他CPU需要读该数据时出现Cache miss，需要重新将数
  据读入Cache。在具体实现中，多数都会采用写无效策略，因为CPU写多次数据，第一次发起写无效后后续
  就不需要多次发送，可以介绍CPU核间总线带宽。
### 3.3 Cache-Line分配策略
#### 3.3.1 写Miss场景
1. 写分配
  在写入数据前将数据读入缓存。优点是，缓存中的数据块在未来读写概率较高，也就是程序局部性比较好的
  时候，写分配效率搞。
2. 写不分配
  写入数据时，直接将要写入的数据传播内存，而并不将数据块读入缓存，这是写不分配策略。当数据块中的数据在未来使用的概率较低时，写不分配性能较好。

优缺点：如果缓存块的大小比较大，该缓存块未来被多次访问的概率也会增加，这种情况下，写分配的策略性能要优
于写不分配。

#### 3.3.2 读miss场景
1. 读分配
2. 读不分配
这两种策略和上面的写miss场景差不多，不多说。

### 3.4 Cache-Line替换策略
进行Allocate动作时，如果没空闲cache line，就需要替换掉一个已经存在的cache line。替换的算法有很多，下面举几个例子。
1. 随机替换
2. 先进先出
3. 最近最少使用(LRU: Least Recently Used)
  将近一段时间未被访问的Cache-Line替换。保护刚拷贝进来的数据不会被替换
4. 最久没有使用LFU（Least Frequently Used）
  最近一段时间访问次数最少的Cache-line被替换。但LFU算法有缺点，所以据说是最不常用的替换算法。
5. Round-Robin：

## 4. Cache刷新
### 4.1 Cache基本操作
1. Invalidate(丢弃Cache中的数据)
2. clean（有修改的数据刷入主存）
clean和invalidate的操作规范：读之前做invalidate操作，写之后做clean操作
### 4.2 Arm中invalidate和clean指令
CISW:
CIVAC:
等等，具体看手册
### 4.3 刷新Cache性能
刷新Cache指令非常耗时，一个Cache-Line（一般64个字节）的Clean & Invalid操作一般会花费数us，如果内存再打一点，会造成严重的性能抖动。

## 5. Cache性能优化
1. 提高代码的局部性：就是说内存的访问最好不要在多个Cache-Line之间频繁跳跃
2. 减少跨Cache-Line的数据访问：
数据跨越两个cache line，就意味着两次load或者两次store。如果数据结构是cache line对齐的， 就有可
能减少一次读写。数据结构的首地址cache line对齐，意味着可能有内存浪费（特别是 数组这样连续分配
的数据结构），所以需要在空间和时间两方面权衡。
__attribute__((aligned(CAA_CACHE_LINE_SIZE)))：结构体cache line对齐，防止cache miss（在urcu代码里边
用到的）
3. 数据的软件预取：
gcc提供了__builtin_prefetch预期一段数据到L1 cache，可以在某些场景下提高性能。但如果没有全面的性
能测试数据，不要乱用此接口，尽量通过其他方式提高性能（例如：提高优化等级[至少-O2]）。
Linux内核态封装了如下几个接口预期数据：
a)	prefetch
b)	prefetchw
c)	prefetch_range

## 6. Cache一致性维护
### 6.1 Cache一致性协议
1. Directory Based（目录一致性协议）
  目录一致性，主要思想是利用目录的形式记录所有cache line和共享数据的位置和状态，因此当处理器对某一cache line进行操作时，根据相应的目录项进行一致性操作。
  系统中cache directory有两种的实现方式：
  集中式目录：使用一个集中式目录来记录当前信息和所有cache状态，只适用于小规模的多处理机系统。
  分布式目录：每个存储器模块维护各自单独的目录，目录中记录着各个存储块的cache line状态和当前信息。状态信息是本地的，当前信息指明哪些cache有该存储器块的cache line副本。适用于分布式存储器层次结构的多处理机系统。
  目录协议避免了广播消息，减少了完成一致性请求所需的通信量， 同时避免了对顺序化互连结构的依赖，具有较好的可扩展性。目前，绝大多数片上多核处理器都采用基于目录的一致性协议。
2. Snoop Based（监听一致性协议）
  监听一致性协议需要总线提供广播机制，cache要不断监听总线上对处理器和存储器模块间的cache操作事件，并广播根据对失效数据的处理请求，写-更新（write-update）或写-无效（write-invalidate）。
  写更新协议，当某个处理器在更新本地cache的同时，将更新后的数据块发送到其他相关的cache，覆盖掉原来的数据块。
  写无效协议，当某个处理器在更新本地cache时，仅通知其他cache，将对应的数据块置无效。
  显然，写更新策略需要耗费大量的总线周期来更新所有的高速缓存和主存中的cache line，代价很大。写更新和写无效协议都必须基于对总线广播的监听，总线带宽广播能力有限，所以对于大规模系统，总线很可能成为系统瓶颈。
  同时监听一致性协议在不支持总线监听的处理器互联网络拓扑，如网格型和超立方体型等用于多计算机消息传递的网络中效率较低。因此监听一致性协议主要用于小规模多处理机系统。
- MESI
- MSI
- MOSI
- MOESI

## 7. PCIe与CPU之间的数据一致性
1. Consistent DMA Mapping
  dma_alloc_coherent类接口，直接申请内存并映射好DMA地址。接口内部根据PCIe是否支持Cache一致性，
  已经将页表属性修改好，能够保证Cache一致性（PCIe不支持Cache一致性，则页表属性为Non-
  Cacheable。PCIe支持Cache一致性，则页表属性为Cacheable），无需在使用过程中再担心Cache一致性。
  但dma_alloc_coherent默认走CMA等内存，能够多少还得看内核配置

2. Streaming DMA Mapping
  dma_map_single类接口，可以将kmalloc出来的地址转为设备可以访问的DMA地址。这类接口需要在使用时
  填写DMA地址访问方向。方向可以填DMA_NONE，DMA_TO_DEVICE，DMA_FROM_DEVICE，DMA_BIDIRECTIONAL。
  ======================= =============================================
  DMA_NONE		no direction (used for debugging)
  DMA_TO_DEVICE		data is going from the memory to the device
  DMA_FROM_DEVICE		data is coming from the device to the memory
  DMA_BIDIRECTIONAL	direction isn't known
  ======================= =============================================
  
  此类接口在使用时，需要做同步操作，且根据方向有不同的操作方式。
  1)	DMA_TO_DEVICE：在写内存之后做同步操作。且在DMA使用这段内存时不能再对DMA内存进行修改。如果非要修改，则需要改变其方向为Bidirectional。
  2)	DMA_FROM_DEVICE：在读内存之前做同步操作。这段内存对于驱动来说应该是只读的。
  3)	DMA_BIDIRECTIONAL：用于驱动和DMA控制器都会读写的内存。驱动在使用这段内存时需要做两次操作，即读之前同步数据，写之后同步数据。
  同步的接口有：
  1)	dma_sync_single_for_cpu
  一般是CPU在读数据之前使用DMA_FROM_DEVICE调用此接口
  2)	dma_sync_single_for_device
  一般是CPU在写数据之后使用DMA_TO_DEVICE调用此接口


https://zhuanlan.zhihu.com/p/694673551
