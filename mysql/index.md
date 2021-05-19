#### B树：

- 父节点的数据不会出现在子节点中
- 所有的节点中存储了data
- 叶子节点之间没有指针相互连接

#### B+树

- 只有叶子节点才会存储数据，非叶子节点至存储键值。
- 叶子节点之间使用双向指针连接，最底层的叶子节点形成了一个双向有序链表
- MyISAM叶子节点中存储的key为索引列的值，数据为索引所在行的磁盘地址
- InnoDB叶子节点中存储的key为索引列的值，聚集索引的value为对应的data.非聚集索引的vaue为该行的主键值

#### 联合索引

idx_abc(a,b,c)索引，相当于创建了(a)、（a,b）（a,b,c）三个索引

**最左匹配原则**：使用组合索引查询时，mysql会一直向右匹配直至遇到范围查询(>、<、between、like)就停止匹配

**覆盖索引**：为了减少回表造成的性能降低

**索引下推**: 5.6之前对第一个条件查到的id hui表过滤。之后可以多条件查找后，通过id 回表查询

#### Innodb Myisam

| MyISAM                                                       | InnoDB                                                       |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 不支持事务                                                   | 支持事务                                                     |
| 锁粒度是表的                                                 | 锁粒度是行的                                                 |
| 非聚集索引，叶节点的data域存放的是数据记录的地址             | 聚集索引，叶子节点中存储的数据                               |
| 保存行数，select count(*)快                                  | select count(*)慢，需全表扫描                                |
| 不支持外键                                                   | 支持外键                                                     |
| 创建表后生成的文件有<br />frm:创建表的语句<br />MYD:表里面的数据文件（myisam data）<br />MYI:表里面的索引文件（myisam index） | 创建表后生成的文件有<br />frm:创建表的语句<br />ibd:表里面的数据+索引文件<br />ibdata:共享数据 |

#### B+插入数据

1. Leaf Page有空间：直接插入
2. Leaf Page无空间，Index Page有空间：插入index page中，然后调整leaf page
3. Leaf Page无空间，Index Page无空间：创建index page，插入index page中后 调整index page与leaf
4. Leaf Page无空间，但是兄弟Leaf Page有空间：调整index中的数据，使得该数据能插入到leaf

图片见图片路径中

#### B+数据删除

1. Leaf Page大于50%，Index Page大于50%：直接删除
2. Leaf Page小于50%，Index Page大于50%：删除数据后，合并leaf,对应的index 也调整
3. Leaf Page小于50%，Index Page小于50%：删除数据，合并index,



