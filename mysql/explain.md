使用explain 显示执行计划

![image-20201222151911467](..\image\mysql\explain.png)

### id

id值越大优先级越高，越先执行。id相同，从上到下

### select_type

表示查询中每个select子句的类型

- SIMPLE `简单的select查询`，查询中`不包含子查询或者UNION`
- PRIMARY 查询中若`包含任何复杂的`子部分，`最外层查询则被标记为PRIMARY`
- SUBQUERY `在SELECT或WHERE列表中包含了子查询`
- DERIVED 在FROM列表中包含的`子查询被标记为DERIVED`（衍生），MySQL会递归执行这些子查询，把`结果放在临时表`中
- UNION 若第二个SELECT出现在UNION之后，则被标记为UNION：若UNION包含在FROM子句的子查询中，外层SELECT将被标记为：DERIVED
- UNION RESULT 从UNION表获取结果的SELECT

### type

type所显示的是查询使用了哪种类型.从最好到最差：system > const > eq_ref > ref > range > index > all

- `system` 表只有一行记录（等于系统表），这是const类型的特列，平时不会出现，这个也可以忽略不计
- `const` 表示通过索引一次就找到了，const用于比较primary key 或者unique索引。因为只匹配一行数据，所以很快。如将主键置于where列表中，MySQL就能将该查询转换为一个常量

- `eq_ref` 唯一性索引扫描，对于每个索引键，表中只有一条记录与之匹配。常见于主键或唯一索引扫描
- `ref` 非唯一性索引扫描，返回匹配某个单独值的所有行，本质上也是一种索引访问，它返回所有匹配某个单独值的行，然而，它可能会找到多个符合条件的行，所以他应该属于查找和扫描的混合体
- `range` 只检索给定范围的行，使用一个索引来选择行，key列显示使用了哪个索引，一般就是在你的where语句中出现between、< 、>、in等的查询，这种范围扫描索引比全表扫描要好，因为它只需要开始于索引的某一点，而结束于另一点，不用扫描全部索引
- `index` Full Index Scan，Index与All区别为index类型只遍历索引树。这通常比ALL快，因为索引文件通常比数据文件小。（也就是说虽然all和Index都是读全表，但index是从索引中读取的，而all是从硬盘读取的）
- `all` Full Table Scan 将遍历全表以找到匹配的行

### possible_keys 和 key

`possible_keys` 显示可能应用在这张表中的索引，一个或多个。`key` 实际用的索引

### key_len

表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度，在`不损失精确性的情况下，长度越短越好`

### ref	

显示索引的那一列被使用了，如果可能的话，最好是一个常数。哪些列或常量被用于查找索引列上的值

### rows

根据表统计信息及索引选用情况，大致估算出找到所需的记录所需要读取的行数，也就是说，用的越少越好





