### 分布式锁

setnx expire.可以使用lua 确保原子性

### 位图bitmap

setbit sign:123:1909 0 1 用户ID=123签到，签到的时间是19年9月份，0代表该月第一天，1代表签到了

### HyperLogLog

UV 访问量巨大，抛弃set. HyperLogLog 提供了两个指令 pfadd(增加计数) 和 pfcount（获取计数）。解决很多精确度不高的统计需求。

### 布隆过滤器

bf.add  	bf.exists	bf.madd	bf.mexists

### Pipeline

可以批量执行一组指令，一次性返回全部结果，可以减少频繁的请求应答

### keys

大量数据中找出固定前缀的ey.可以使用keys.但是这是阻塞性的。可以使用scan,再程序去重

### redis延时队列

使用sortedset，拿时间戳作为score，消息内容作为key调用zadd来生产消息，消费者用**zrangebyscore**指令获取N秒之前的数据轮询进行处理。

### redis核心结构 redisObject

Redis的key是顶层模型，它的value是扁平化的。Redis中，所有的value都是一个object，它的结构如下

> ```
> typedef struct redisObject {
>     unsigned [type] 4;
>     unsigned [encoding] 4;
>     unsigned [lru] REDIS_LRU_BITS;
>     int refcount;
>     void *ptr;
> } robj;
> ```

- type：数据类型，就是我们熟悉的string、hash、list、set 、sorted set等。
- encoding：内部编码。指的是当前这个value底层是用的什么数据结构。因为同一个数据类型底层也有多种数据结构的实现，所以这里需要指定数据结构。有raw、int、ht、zipmap、linkedlist、ziplist、intset
- REDIS_LRU_BITS：当前对象可以保留的时长。
- refcount：对象引用计数，用于GC。
- ptr：指针，指向以encoding的方式实现这个对象的实际地址

![img](..\image\redis\redisObject.png)

### string

string类型有两种底层储存结构

- int：存放整数类型；
- SDS：存放浮点、字符串、字节类型；SDS（simple dynamic string）类似于 **Java** 中的 **ArrayList**，可以通过预分配冗余空间的方式来减少内存的频繁分配

> struct sdshdr{
>  int len;*// buf中已经占用的字符长度*
>  int free;*// buf中剩余可用的字符长度*
>  char buf[];*// 数据空间*,最大容量为512M
> }

相较于c字符串的区别

- 计算len时，不需要遍历
- 提前计算好内存，杜绝缓冲区溢出

应用：计数器。共享用户session

### list

两种数据结构。都包含len字段

- ziplist：元素个数少且元素内容长度不大

  是由一系列特殊编码的连续内存块组成的顺序型数据结构，一个压缩列表可以包含任意多个节点（entry），每个节点可以保存一个字节数组或者一个整数值

- linkedlist：类似数组，通过一片连续的内存空间，来存储数据。不过，它跟数组不同的一点是，它允许存储的数据大小不同。每个节点上增加一个length属性来记录这个节点的长度，这样比较方便地得到下一个节点的位置。

应用：消息队列、文章列表或数据分页展示

### hash

成员少的时候采用类似一维数组来x紧凑存储，对应的redisObject的encoding 为 zipmap，当成员增大时，换转变成真正的hashmap.

应用：缓存复杂对象

### set

- intset：当数据全是整数值，而且数量少于512个时，才使用intset，intset是一个由整数组成的有序集合，可以进行二分查找
- hashtable：不满足intset使用条件的情况下都使用字典（拉链法），使用字典时把value设置为null

应用：共同好友

### sortset

- ziplist：数据少时，使用ziplist：ziplist占用连续内存，每项元素都是（数据+score）的方式连续存储，按照score从小到大排序
- skiplis:：数据多时，使用字典+跳表

应用：排行榜

















