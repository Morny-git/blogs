#### 一级缓存和二级缓存

- 一级缓存是单个session级别的。在操作数据库时需要构造 sqlSession对象，在对象中有一个(内存区域)数据结构（HashMap）用于存储缓存数据。**Mybatis默认开启一级缓存**。
- 二级缓存是多个session级别的，只不过多个session需要在一个namespave下



查询结果实时性要求不高的情况下可采用mybatis二级缓存降低数据库访问量，提高访问速度，同时配合设置缓存刷新间隔flushInterval来根据需要改变刷新缓存的频次。

通常情况下，如果同时设置了一级缓存和二级缓存，会先使用二级缓存的数据，然后再使用一级缓存的数据，最后才会访问数据库

#### 如何判断两次查询一摸一样

- statementId
- 结果集中的结果范围
- 传给statement的参数值

#### 一级缓存配置

可设置seesion 和statement两个参数。默认session.即在一个MyBatis会话中执行的所有语句，都会共享这一个缓存。一种是`STATEMENT`级别，可以理解为缓存只对当前执行的这一个`Statement`有效

```
<setting name="localCacheScope" value="SESSION"/>
```

xml文件 的<select>标签添加 flushCache="true", 禁用此查询的一级缓存

#### 二级缓存配置

开启二级缓存

```
<setting name="cacheEnabled" value="true"/>
```

且在namespace中使用<cache/>  

禁用二级缓存

在statement中设置useCache=false










引用

> https://blog.csdn.net/a1036645146/article/details/106811411
>
> https://blog.csdn.net/key_768/article/details/105855925
>
> https://tech.meituan.com/2018/01/19/mybatis-cache.html