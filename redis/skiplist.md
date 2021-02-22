#### 跳表的高度

```
public static int random_level(int high)
{   Random random = new Random();
    int level = 1;
    while (1 == random.nextInt(2))
        level++;
    return level < high ? level : high+1;
}
```

用实验中丢硬币的次数 K 作为元素占有的层数。显然随机变量 K 满足参数为 p = 1/2 的几何分布

跳表的查询

```
find(x)
{
    p = top;
    while (1) {
        while (p->next->key < x)
            p = p->next;
        if (p->down == NULL)
            return p->next;
        p = p->down;
    }
}
```