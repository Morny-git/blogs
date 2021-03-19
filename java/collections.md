#### 1.list set map 区别

List和Set接口自Collection接口，而Map不是继承的Collection接口

**List接口**

```
元素有放入顺序，元素可重复 
List接口有三个实现类：LinkedList，ArrayList，Vector       
LinkedList：底层基于链表实现，链表内存是散乱的，每一个元素存储本身内存地址的同时还存储下一个元素的地址。链表增删快，查找慢 
ArrayList和Vector的区别：ArrayList是非线程安全的，效率高；Vector是基于线程安全的，效率低 
List是一种有序的Collection，可以通过索引访问集合中的数据,List比Collection多了10个方法，主要是有关索引的方法。
      1).所有的索引返回的方法都有可能抛出一个IndexOutOfBoundsException异常
      2).subList(int fromIndex, int toIndex)返回的是包括fromIndex，不包括toIndex的视图，该列表的size()=toIndex-fromIndex。
      所有的List中只能容纳单个不同类型的对象组成的表，而不是Key－Value键值对。例如：[ tom,1,c ];
      所有的List中可以有相同的元素，例如Vector中可以有 [ tom,koo,too,koo ];
      所有的List中可以有null元素，例如[ tom,null,1 ];
      基于Array的List（Vector，ArrayList）适合查询，而LinkedList（链表）适合添加，删除操作;
```

**Set接口**

```
元素无放入顺序，元素不可重复（注意：元素虽然无放入顺序，但是元素在set中的位置是有该元素的HashCode决定的，其位置其实是固定的）
Set接口有两个实现类：HashSet(底层由HashMap实现)，LinkedHashSet 
  SortedSet接口有一个实现类：TreeSet（底层由平衡二叉树实现）
  Query接口有一个实现类：LinkList 
  Set具有与Collection完全一样的接口，因此没有任何额外的功能，不像前面有两个不同的List。实际上Set就是Collection,只是行为不同。(这是继承与多态思想的典型应用：表现不同的行为。)Set不保存重复的元素(至于如何判断元素相同则较为负责)
  Set : 存入Set的每个元素都必须是唯一的，因为Set不保存重复元素。加入Set的元素必须定义equals()方法以确保对象的唯一性。Set与Collection有完全一样的接口。Set接口不保证维护元素的次序。
  HashSet : 为快速查找设计的Set。存入HashSet的对象必须定义hashCode()。
  TreeSet : 保存次序的Set, 底层为树结构。使用它可以从Set中提取有序的序列。
  LinkedHashSet : 具有HashSet的查询速度，且内部使用链表维护元素的顺序(插入的次序)。于是在使用迭代器遍历Set时，结果会按元素插入的次序显示。
```

**map接口**

```
以键值对的方式出现的 
Map接口有三个实现类：HashMap，HashTable，LinkeHashMap 

      HashMap非线程安全，高效，支持null；初始化16，扩容2n

      HashTable线程安全，低效，不支持null 。初始化11.扩容2n+1

      SortedMap有一个实现类：TreeMap 
```

关于数组与链表：

- 数组：占用空间连续。寻址容易，查询速度快，但是，增加和删除效率低。
- 链表：占用空间不连续。寻址困难，查询速度慢，但是，增加和删除效率高。
- 数组+链表：综合，查询速度快，增删速度也快

![image-20210311155330841](..\image\java\iterator.png)lsit

### list

ArrayList:空间连续，适合查询。允许重复和null.初始化10.扩容1.5倍。采用了Fail-Fast机制，面对并发的修改时，迭代器很快就会完全失败。

> 原理参考：http://zhangshixi.iteye.com/blog/674856l
> https://www.cnblogs.com/leesf456/p/5308358.html

LinkedList:空间不连续，适合新增删除。允许重复和null

> 原理参考：	.http://www.cnblogs.com/ITtangtang/p/3948610.htmll
> https://www.cnblogs.com/leesf456/p/5308843.html

vector :线程安全，已不使用。可用hashtable 代替

### set

hashset:value为空的hashmap

> 参考：https://www.iteye.com/blog/zhangshixi-673143

### map

hashmap：数组+链表，链表长度超过一定长度转成红黑树。允许key-value为空。采用了Fail-Fast机制，通过一个modCount值记录修改次数，对HashMap内容的修改都将增加这个值。初始化16.扩容2n

> 参考：http://zhangshixi.iteye.com/blog/672697
>  参考：http://blog.csdn.net/lizhongkaide/article/details/50595719

hashtable:数组+链表。不允许null.线程安全。synchronized是针对整张Hash表的，即每次锁住整张表让线程独占

> 参考：http://blog.csdn.net/zheng0518/article/details/42199477

ConcurrentHashMap：线程安全的hashmap.

> http://blog.csdn.net/zheng0518/article/details/42199477

#### ArrayList查询速度快的原因

基于数组实现，可以根据元素下标进行查询，查询方式为(数组首地址+元素长度*下标，基于这个位置读取相应的字节数就可以了。

#### 如果数组存的是对象，怎么根据下标定位元素所在位置

对象数组每个元素存放的是对象的引用，而引用类型如果开启指针压缩占用4字节，不开启则占用8字节，所以对象数组同样适用上面的公式

#### ArrayList的扩容

new 时指定大小。默认10.新容量=(旧容量*3)/2+1。因为必须指明数组的长度才能给数组分配空间；由于数组的特性，ArrayList扩容是创建一个更大的数组，然后将原来的元素拷贝到更大的数组中，扩容的核心方法是Arrays.copyOf方法

#### treemap 按照key  value 排序

https://blog.csdn.net/sh542610/article/details/25005885

