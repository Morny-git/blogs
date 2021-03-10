#### 1.HashTable&HashMap

##### HashTable

- - 底层数组+链表实现，无论key还是value都**不能为null**，线程**安全**，实现线程安全的方式是在修改数据时锁住整个HashTable，效率低，ConcurrentHashMap做了相关优化
  - 初始size为**11**，扩容：newsize = olesize*2+1
  - 计算index的方法：index = (hash & 0x7FFFFFFF) % tab.length

##### HashMap

- - 底层数组+链表实现，可**以存储null键和null值**，线程**不安全**
  - 初始size为**16**，扩容：newsize = oldsize*2，size一定为2的n次幂
  - 扩容针对整个Map，每次扩容时，原来数组中的元素依次重新计算存放位置，并重新插入
  - 插入元素后才判断该不该扩容，有可能无效扩容（插入后如果扩容，如果没有再次插入，就会产生无效扩容）
  - 当Map中元素总数超过Entry数组的75%，触发扩容操作，为了减少链表长度，元素分配更均匀
  - 计算index方法：index = hash & (tab.length – 1)

#### **2.HashMap 1.7与1.8的区别**

分别有以下几点：

- **数组+链表**改成了**数组+链表或红黑树**，防止发生hash冲突，链表长度过长，将时间复杂度由`O(n)`降为`O(logn)`;
- 链表的插入方式从**头插**法改成了**尾插**法，简单说就是插入时，如果数组位置上已经有元素，1.7将新元素放到数组中，原始节点作为新节点的后继节点，1.8遍历链表，将元素放置到链表的最后。因为1.7头插法扩容时，头插法会使链表发生反转，多线程环境下会产生环；
- 扩容的时候1.7需要对原数组中的元素进行**重新hash定位**在新数组的位置，1.8采用更简单的判断逻辑，位置不变或**索引+旧容量大小**；
- 在插入时，1.7先判断是否需要**扩容，再插入**，1.8先进行插入，**插入完成再判断是否需要扩容**；

#### 3.ConcurrentHashMap  1.7与1.8的区别

- **数据结构** : 1.7 分段锁+数组+链表来实现的，在ConcurrentHashMap中保存一个SegMent数组，Segment是继承ReentrantLock的可重入锁，也就是说对于每个Segment的操作可以通过加锁解锁的方式来保证线程的安全性

  1.8 类似于Hashmap了也是用数组+链表+红黑树来实现的，不过它使用了CAS的方式和Synchronized加锁来保证线程的安全.查询效率提从原来的O(n)，提高为O(logn)

- **计算大小**:1.7  先采用不加锁的方式，连续计算元素的个数，最多计算3次： 如果前后两次计算结果相同，则说明计算出来的元素个数是准确的； 如果前后两次计算结果都不同，则给每个`Segment`进行加锁，再计算一次元素的个数；

  1.8中使用一个`volatile`类型的变量`baseCount`记录元素的个数，当插入新数据或则删除数据时，会通过`addCount()`方法更新`baseCount`

- **put:**1.7 数据的方式是通过hash的方式先找到插入entry在Segment数组中的位置，然后再通过Hash的方式找到entry在Segment中HashEntry中的位置，然后再执行插入到链表中，其中在对segment操作的时候会进行trylock如果获取到锁则执行插入，如果没有获取到锁则会重试3次还没获取到则用阻塞锁的方式获取锁，同时对链表中的节点进行删除操作时，需要将删除节点的前面所有节点复制一遍然后用头插法插入链表(原因是每个节点的next是final的不可修改)

  **jdk1.8  put**流程：

  - 如果数据没初始化则初始化
  - 通过hash方式（与hashMap的hash方式类似只不过将hash值转化为正数）找到要put节点在数组中的位置，如果该位置为空，则通过CAS的方式插入/
  - 如果当前节点正在扩容则该线程参与扩容完成
  - 如果该位置有节点则通过synchronized加锁判断是如果该节点是链表则查找PUT，如果是红黑树则执行红黑树的PUT，之后通过bincount判断是否要将链表转化成红黑树。
  - 最后更新size值并且判断是否需要扩容。

- **get** 1.7 数据不需要加锁，如果读到的数据为空则会加锁后再读一遍，因为可能由于某个线程在删除某个节点导致读到的数据为空。（删除某个节点需要把前面所有节点复制一遍重新插入）

- **扩容**: 1.7 不会对整个ConcurrentHashMap扩容只会针对某个segment扩容

  

> &-位与  0 & 0= 0 ，0 & 1= 0，1 & 0= 0， 1 & 1= 1
>
> |-位或 0 | 0= 0 , 1 | 0= 1 ， 0 | 1= 1 , 1 | 1= 1 
>
> ^-异或	0 ^ 0=0 ， 0 ^ 1= 1 ， 1 ^ 0= 1 ， 1 ^ 1= 0
>
> \>>>  无符号右移
>

#### 4.HashMap 计算hash 

> static final int hash(Object key) {      
>
>   int h;      
>
>   return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);   
>
>  }

**为什么要右移16位？**
其实是为了**减少碰撞，进一步降低hash冲突的几率**。int类型的数值是4个字节的,也就是32位，右移16位异或可以同时保留高16位于低16位的特征

**为什么要异或运算？**
首先将高16位无符号右移16位与低十六位做异或运算。如果不这样做，而是直接做&运算那么高十六位所代表的部分特征就可能被丢失 将高十六位无符号右移之后与低十六位做异或运算使得高十六位的特征与低十六位的特征进行了混合得到的新的数值中就高位与低位的信息都被保留了 ，而在这里采用异或运算而不采用& ，| 运算的原因是 异或运算能更好的保留各部分的特征，如果采用&运算计算出来的值会向0靠拢，采用|运算计算出来的值会向1靠拢

**存放位置为什么用&**

(n-1)&hash判断元素存放的位置，n-1等于数组的长度，与运算相当于取余运算(**计算速度大于取余运算**)

**为何HashMap的数组长度一定是2的次幂**

index算法是(n-1)&hash。n-1用二进制表示全是1，直接取低位，减少冲突机会

**为什么Map桶中个数超过8才转为红黑树**

在 JDK 1.7 中，是用链表存储的，这样如果碰撞很多的话，就变成了在链表上的查找；在 JDK 1.8 进行了优化，当链表长度较大时（超过 8），会采用红黑树来存储，这样大大提高了查找效率。针对JDK 1.8版本的冲突解决，经常会被问到为什么是超过8才用红黑树的问题。

当桶中个数达到8就转成红黑树，当长度降到6就转成普通链表存储，而7就是为了防止链表和树频繁转换。至于选择8的原因，根据源代码的解析，是因为一个桶中存储8个或以上的概率非常小，这样小的事件都发生了，说明产生了严重的冲突，需要更高效的查找方式

**为什么hashMap的链表会形成死循环**

JDK1.7 的 HashMap 链表会有死循环的可能，因为JDK1.7是采用的头插法，在多线程环境下有可能会使链表形成环状，从而导致死循环。JDK1.8做了改进，用的是尾插法，不会产生死循环

**ConcurrentHashMap扩容**是允许多个线程并发进行扩容的，首先构造一个两倍于当前数据长度的数组，然后计算每个线程处理的槽的空间，然后通过死循环依次递减的方式对每个槽位进行判断知道有实际值的槽位，通过将槽位的每个节点分成2个链表将高位逻辑与计算为1的链表插入到i+n的位置，同时将旧的节点设置为占位符然后继续向前推进扩容操作

hashmap 使用1.8时，key必须保证实现了compare接口

> Comparable和Compator两个的区别：Comparable 重写compareTo方法，而在Compator实现了compare方法。所以Comparable可以称之为内部比较器，而Compator可以称之为外部比较器。









引用 

> https://www.bbsmax.com/A/q4zVxylX5K/