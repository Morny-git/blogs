可以显示调用缓存，把**query_cache_type**设置成为DEMAND，这样SQL默认不适用缓存，想用缓存就用SQL_CACHE。不用使用SQL_NO_CACHE.缓存在8.0之后就去取消了

一级缓存

也称本地缓存，sqlSession级别的缓存。一级缓存是一直开启的

二级缓存：全局缓存；基于namespace级别的缓存。一个namespace对应一个二级缓存。

缓存首先一进来去查二级缓存，二级缓存没有去找一级缓存，一级缓存没有去找数据库。二级缓存----->一级缓存-------->数据库。



innoDB 内存架构  内存+磁盘

MySQL 会先改内存，然后记录 redo log，（redis 的aof 是先执行命令再写入内存，内存再写入aof）等有空了再刷磁盘，如果内存里没有数据，就去磁盘 load

### bufferpool

![在这里插入图片描述](..\image\mysql\bufferPool.png)

InnoDB访问一个Page时，首先会从Buffer Pool中获取，如果未找到，则会访问数据文件，读取到Page，并将其put到LRU List中，当一个Instance的Buffer Pool中没有可用的空闲Page时，会对LRU List中的Page进行淘汰。

**Free List：** Free List中存放的都是未曾使用的空闲Page，InnoDB需要Page时从Free List中获取，如果Free List为空，即没有任何空闲Page，则会从LRU List和Flush List中通过淘汰旧Page和Flush脏Page来回收Page。在InnoDB初始化时，会将Buffer chunks中的所有Page加入到Free List中

**LRU List：** 所有从数据文件中新读取进来的Page都会缓存在LRU List，并通过LRU策略对这些Page进行管理。LRU List实际划分为Young和Old两个部分，其中Young区保存的是较热的数据，Old区保存的是刚从数据文件中读取出来的数据，如果LRU List的长度小于512，则不会将其拆分为Young和Old区。当InnoDB读取Page时，首先会从当前Buffer Pool Instance的page_hash查找，并分为三种情况来处理：

- 如果在page_hash找到，即Page在LRU List中，则会判断Page是在Old区还是Young区，如果是在Old区，在读取完Page后会把它添加到Young区的链表头部
- 如果在page_hash找到，并且Page在Young区，需要判断Page所在Young区的位置，只有Page处于Young区总长度大约1/4的位置之后，才会将其添加到Young区的链表头部
- 如果未能在page_hash找到，则需要去数据文件中读取Page，并将其添加到Old区的头部

**Flush List：** 所有被修改过且还没来得及被flush到磁盘上的Page（脏页），都会被保存在这个链表中。所有保存在Flush List上的数据都会在LRU List中，但在LRU List中的数据不一定都在Flush List中。在Flush List上的每个Page都会保存其最早修改的lsn，即oldest_modification，虽然一个Page可能被修改多次，但只记录最早的修改。Flush List上的Page会按照其各自的oldest_modification进行降序排序，链表尾部保存oldest_modification最小的Page，在需要从Flush List中回收Page时，从尾部开始回收

> 参考：https://blog.csdn.net/u010647035/article/details/104717579

#### `Change Buffer`

`Change Buffer`是用来缓存那些在`Buffer Pool`中没有的数据页中的二级缓存。`Insert`,`update`,`delete`等操作会触发缓存动作，并且在数据页缓存进`Buffer Pool`的时候，会将`Change Buffer`中的数据合并到`缓存的数据页`中

- `Change Buffer`默认占用所有缓存的`25%`,最高调整到`50%`

- 如果当前数据库中的二级索引较少，或者数据库是一个只读数据库，可以关闭`Change Buffer`

- 相反如果有大量的`update`,`Delete`,`Insert`操作，可以加大`Change Buffer`

  

> 参考：https://blog.csdn.net/weixin_29491885/article/details/102957641

#### Double Write Buffer

MySQL的buffer一页的大小是16K，文件系统一页的大小是4K，也就是说，MySQL将buffer中一页数据刷入磁盘，要写4个文件系统里的页

![img](..\image\mysql\DWB.png)

如上图所示，当有页数据要刷盘时：

第一步：页数据先memcopy到DWB的内存里；

第二步：DWB的内存里，会先刷到DWB的磁盘上；

第三步：DWB的内存里，再刷到数据磁盘存储上；

步骤2和步骤3要写2次磁盘，这就是“Double Write”的由来.假设步骤2掉电，磁盘里依然是1+2+3+4的完整数据,假如步骤3掉电，DWB里存储着完整的数据.

MySQL有很强的数据安全性机制：

（1）在异常崩溃时，如果不出现“页数据损坏”，能够通过redo恢复数据；

（2）在出现“页数据损坏”时，能够通过double write buffer恢复页数据

> 参考：https://blog.csdn.net/xx123698/article/details/107201808

### redo log 和 undo log

undolog实现事务原子性,记录的是数据的逻辑变化，redolog实现事务的持久性,redo log也是基于页的格式来记录的

避免数据写入时io瓶颈带来的性能问题，MySQL采用了这样一种缓存机制：当query修改数据库内数据时，InnoDB先将该数据从磁盘读取到内存中，修改内存中的数据拷贝，并将该修改行为持久化到磁盘上的事务日志（先写redo log buffer，再定期批量写入），而不是每次都直接将修改过的数据记录到硬盘内，等事务日志持久化完成之后，内存中的脏数据可以慢慢刷回磁盘，称之为Write-Ahead Logging。事务日志采用的是追加写入，顺序io会带来更好的性能优势。

为了避免脏数据刷回磁盘过程中，掉电或系统故障带来的数据丢失问题，InnoDB采用事务日志（redo log）来解决该问题

##### InnoDB如何保证原子性(Atomicity)？

在事务里任何对数据的修改都会写一个undo log，然后进行数据的修改，如果出现错误，存储引擎会利用undo log的备份数据恢复到事务开始之前的状态。

##### InnoDB如何保证一致性(Consistency)？

事务的原子性和隔离性保证了数据的一致性

##### InnoDB如何保证隔离性(Isolation)？

InnoDB的默认隔离级别REPEATABLE READ + Next-Key Locking，保证了数据库的隔离性，不出现并发一致性问题，且理论上效率高于SERIALIZABLE隔离级别。undo log实现多版本并发控制（MVCC）来辅助保证事务的隔离性

##### InnoDB如何保证持久性(Durability)？

InnoDB存储引擎在启动时不管上次数据库运行时是否正常关闭，都会尝试通过redo log进行恢复操作。

**1.redo log通常是物理日志，记录的是数据页的物理修改，而不是某一行或某几行修改成怎样怎样，它用来恢复提交后的物理数据页(恢复数据页，且只能恢复到最后一次提交的位置)。**

修改innodb_flush_log_at_trx_commoit参数来控制重做日志刷新到磁盘的策略，

- 0表示事务提交时，重做日志缓存并不立即写入重做日志文件，而是随着Master Thread的间隔进行fsync操作。
- 1参数默认值，表示事务提交时必须调用一次fsync操作。
- 2表示事务提交时将重做日志写入重做日志文件，但仅写入文件系统的缓存中，不进行fsync操作。

**2.undo用来回滚行记录到某个版本。undo log一般是逻辑日志，根据每行记录进行记录**，记录相反的操作。

事务提交后并不能马上删除undo log，这是因为可能还有其他事务需要通过undo log 来得到行记录之前的版本。故事务提交时将undo log 放入一个链表中，是否可以删除undo log 根据操作不同分以下2种情况：

- Insert undo log： insert操作的记录，只对事务本身可见，对其他事务不可见(这是事务隔离性的要求),故该undo log可以在事务提交后直接删除。不需要进行 purge操作。

  > purge的主要任务是将数据库中已经 mark del 的数据删除，另外也会批量回收undo pages

- update undo log：记录的是对 delete和 update操作产生的 undo log。该undo log可能需要提供MVCC机制，因此不能在事务提交时就进行删除。提交时放入undo log链表,等待 purge线程进行最后的删除

####  delete/update操作的内部机制

当事务提交的时候，innodb不会立即删除undo log，因为后续还可能会用到undo log，如隔离级别为repeatable read时，事务读取的都是开启事务时的最新提交行版本，只要该事务不结束，该行版本就不能删除，即undo log不能删除。

但是在事务提交的时候，会将该事务对应的undo log放入到删除列表中，未来通过purge来删除。并且提交事务时，还会判断undo log分配的页是否可以重用，如果可以重用，则会分配给后面来的事务，避免为每个独立的事务分配独立的undo log页而浪费存储空间和性能。

通过undo log记录delete和update操作的结果发现：(insert操作无需分析，就是插入行而已)

- delete操作实际上不会直接删除，而是将delete对象打上delete flag，标记为删除，最终的删除操作是purge线程完成的。
- update分为两种情况：update的列是否是主键列。
  - 如果不是主键列，在undo log中直接反向记录是如何update的。即update是直接进行的。
  - 如果是主键列，update分两部执行：先删除该行，再插入一行目标行

> 参考：https://www.cnblogs.com/better-farther-world2099/p/9290966.html

InnoDB有三种行锁的算法：

- Record Lock：单个行记录上的锁
- Gap Lock：间隙锁，锁定一个范围，而非记录本身
- Next-Key Lock：结合Gap Lock和Record Lock，锁定一个范围，并且锁定记录本身。主要解决的问题是REPEATABLE READ隔离级别下的幻读。可以参考文章了解事务隔离级别的相关知识点



#### checkpion机制

**在innodb中，数据刷盘的规则只有一个：checkpoint。**但是触发checkpoint的情况却有几种。**不管怎样，****checkpoint****触发后，会将buffer****中脏数据页和脏日志页都刷到磁盘。**

innodb存储引擎中checkpoint分为两种：

- sharp checkpoint：在重用redo log文件(例如切换日志文件)的时候，将所有已记录到redo log中对应的脏数据刷到磁盘。
- fuzzy checkpoint：一次只刷一小部分的日志到磁盘，而非将所有脏日志刷盘。有以下几种情况会触发该检查点：
  - master thread checkpoint：由master线程控制，**每秒或每10秒**刷入一定比例的脏页到磁盘。
  - flush_lru_list checkpoint：从MySQL5.6开始可通过 innodb_page_cleaners 变量指定专门负责脏页刷盘的page cleaner线程的个数，该线程的目的是为了保证lru列表有可用的空闲页。
  - async/sync flush checkpoint：同步刷盘还是异步刷盘。例如还有非常多的脏页没刷到磁盘(非常多是多少，有比例控制)，这时候会选择同步刷到磁盘，但这很少出现；如果脏页不是很多，可以选择异步刷到磁盘，如果脏页很少，可以暂时不刷脏页到磁盘
  - dirty page too much checkpoint：脏页太多时强制触发检查点，目的是为了保证缓存有足够的空闲空间。too much的比例由变量 innodb_max_dirty_pages_pct 控制，MySQL 5.6默认的值为75，即当脏页占缓冲池的百分之75后，就强制刷一部分脏页到磁盘。

由于刷脏页需要一定的时间来完成，所以记录检查点的位置是在每次刷盘结束之后才在redo log中标记的

> 参考：https://www.cnblogs.com/f-ck-need-u/archive/2018/05/08/9010872.html

#### 当MySQL更新数据时，InnoDB内部的操作流程大致是:

(1).将数据读入InnoDB buffer pool,并对相关记录加独占锁；
(2).将UNDO信息写入undo表空间的回滚段中；
(3).更改缓存页的数据，并将更新记录写入redo buffer中；
(4).提交时根据innodb_flush_log_at_trx_commit的设置，用不同的方式将redo buffer中的更新记录刷新到InnoDB redo log file中，然后释放独占锁;
(5).后台IO线程根据需要择机将缓存中更新过的数据刷新写入到磁盘文件中。

#### binlog

简单理解为存储着每条变更的`SQL`语句，update/delete/insert/truncate/creat。**逻辑变化**

作用：复制（主从复制)和恢复数据

#### redolog

Mysql的基本存储结构是**页**,update时先把这条记录所在的**页**找到，然后把该页加载到内存中，将对应记录进行修改。不是每个请求都会立马落盘，速度会很慢。

但是如果内存把数据修改了，还没来得及落盘，此时数据库挂掉了。此时通过redolog实现恢复

内存写完了，然后会写一份`redo log`，这份`redo log`记载着这次**在某个页上做了什么修改**，写redolog时会先写buffer，并通过配置的参数来决定什么时候落盘

`redo log`的存在为了：当我们修改的时候，写完内存了，但数据还没真正写到磁盘的时候。此时我们的数据库挂了，我们可以根据`redo log`来对数据进行恢复。因为`redo log`是顺序IO，所以**写入的速度很快**，并且`redo log`记载的是**物理变化**（xxxx页做了xxx修改），文件的体积很小，**恢复速度很快**

事务开始时写redolog ,提交时写binlog.其中一个log失败，那么：

- 如果写`redo log`失败了，那我们就认为这次事务有问题，回滚，不再写`binlog`。
- 如果写`redo log`成功了，写`binlog`，写`binlog`写一半了，但失败了怎么办？我们还是会对这次的**事务回滚**，将无效的`binlog`给删除（因为`binlog`会影响从库的数据，所以需要做删除操作）
- 如果写`redo log`和`binlog`都成功了，那这次算是事务才会真正成功

**两阶段提交**来保证`redo log`和`binlog`的数据是一致

- 阶段1：InnoDB`redo log` 写盘，InnoDB 事务进入 `prepare` 状态
- 阶段2：`binlog` 写盘，InooDB 事务进入 `commit` 状态
- 每个事务`binlog`的末尾，会记录一个 `XID event`，标志着事务是否提交成功，也就是说，恢复过程中，`binlog` 最后一个 XID event 之后的内容都应该被 purge。

> 参考：https://mp.weixin.qq.com/s/Lx4TNPLQzYaknR7D3gmOmQ