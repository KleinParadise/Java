### Memcached
  - 内存性缓存组件,其一旦重启,就会丢失所有数据,Memcached组件之间不通信,由客户端通过key hash后进行分布和协同。Memcached是多线程的,主线程与工作线程协同,利用多核,	提升开发效率

### Memcached五大模块
  - 哈希表对key快速定位
  - LRU缓存回收算法
  - slab内存分配
  - 多线程并发处理用户请求
  - 网络处理模块基于Libevent实现(事件的分发处理)


### 网络命令处理流程
  1. 主线程监听是否有新连接接入,如果有新连接,主线程进入come_listening状态,接收新连接,将新连接调度给工作线程
  2. worker线程监听管道,当收到主线程通过管道发送新连接消息后,worker线程创建conn结构体,做一些初始化重置的操作,然后进入等待状态,注册读事件,等待网络io
  3. 有数据到来时,进入read状态,读取网络数据
  4. 数据读完后,解析指令
  5. 如果是读取指令,获取缓存数据,进入到结果响应状态
  6. 如果是变更指令,继续读取变更数据,将读取的数据写入缓存,进入到结果响应状态
  7. 将结果响应发给客户端，连接重置等待下一次命令处理
  8. 在读取、解析、处理、响应过程，遇到任何异常就进入 conn_closing，关闭连接


### 通过hash表快速定位key细节
把key映射到hash表一个位置,来达到快速访问的目的
  - hash冲突
    - 不同key通过hash算法映射到了相同位置,即hash冲突。每个桶维护了一个单向链表,将key hash值相同的item通过单向链表连接起来
  - key定位算法
    - key通过hash算法得到32位整数,然后与hash表的size取模得到最后结果
  - 插入&查找数据
    - 新数据,将该item插入单向链表的头部
    - 查找数据,遍历单向链表,依次对比key字符串,返回与key字符串相同的数据
  - hash表扩容
    - item数目是hash桶的1.5倍时会启用hash线程进行扩容,hash线程会暂停其他辅助线程,然后将当前hash表设置为旧hash表,将新的hash表设置为旧表的两倍。
	  工作线程和辅助线程恢复工作,hash线程将item从旧的hash表复制到新的hash表
     - 每次按桶链表纬度迁移，即一次迁移一个桶里单向链表的所有Item元素。按照迁移位置选择新旧hash表
  - 脏数据处理
     - Mc哈希表在读取、变更以及扩容迁移过程中,会对定位到的桶和item加锁来避免脏数据的产生

### 淘汰冷key和失效key
  - MC删除回收数据的三种方式
     1. 惰性删除,key失效后,不立即删除淘汰数据,当客户端通过该key再次获取数据,发现key已过期,在真正的进行删除和回收。(提高cpu性能,但是会浪费内存空间)
     2. 对item进行内存分配时,如果内存用完,无法申请到新内存。则到LRU队尾扫描,回收过期失效的key,如无过期失效key,则强制删除一个key
     3. LRU维护线程,定时扫描LRU四个队列,对过期失效的key异步淘汰
  - flush_all指令
    - 通过该指令让key在指定时间内全部失效,但是数据不会从缓存中移除,还是会通过getkey和LRU维护线程执行回收操作

### slab内存分配,减少内存碎片,保证高效读写
  - 内存分配规则
    1. MC在启动的时候,会初始化一个slabclass的数组,该数组用于存储slabclass_t结构体(最多支持63个)
    2. 每个slabclass_t都会分配一个1M大小的slab,并且每个slabclass_t，都只存储一定大小范围的数据(前一个slabclass_t可以存储的数据要比后一个slabclass_t可存储的数据小)
    3. slab会被切分为N个小的内存块chunk，这个小的内存块的大小取决于slabclass_t结构上的size的大小(mc能存储不同大小数据并且碎片少的原因)
    4. 被切割的小的内存块，主要用来存储item
    5. 通过freelist链表来管理slab中item的分配
  - item
    - item在空闲期间，即初始分配时以及被回收后，都被freelist管理。在存储期间，被哈希表、LRU 管理。












