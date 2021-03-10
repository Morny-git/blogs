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















表的索引存储在.MYI文件，数据存储在.MYD文件中


