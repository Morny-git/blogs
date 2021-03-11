vm options处加入-XX:+PrintGCDetails  可打印GC日志

```
[PSYoungGen: 1042240K->5232K(1042944K)] 1598466K->565722K(2091520K), 0.0166536 secs] [Times: user=0.04 sys=0.00, real=0.01 secs] 
[Full GC (System) [Tenured: 2241K->193K(5312K), 0.0056517 secs] 4289K->193K(7680K), [Perm : 2950K->2950K(21248K)], 0.0057094 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
Heap
 def new generation   total 2432K, used 43K [0x00000000052a0000, 0x0000000005540000, 0x0000000006ea0000)
  eden space 2176K,   2% used [0x00000000052a0000, 0x00000000052aaeb8, 0x00000000054c0000)
  from space 256K,   0% used [0x00000000054c0000, 0x00000000054c0000, 0x0000000005500000)
  to   space 256K,   0% used [0x0000000005500000, 0x0000000005500000, 0x0000000005540000)
 tenured generation   total 5312K, used 193K [0x0000000006ea0000, 0x00000000073d0000, 0x000000000a6a0000)
   the space 5312K,   3% used [0x0000000006ea0000, 0x0000000006ed0730, 0x0000000006ed0800, 0x00000000073d0000)
 compacting perm gen  total 21248K, used 2982K [0x000000000a6a0000, 0x000000000bb60000, 0x000000000faa0000)
   the space 21248K,  14% used [0x000000000a6a0000, 0x000000000a989980, 0x000000000a989a00, 0x000000000bb60000)
```

- 回收分类。先说明是YGC 还是FullGC.  FullGC有stw
- 收集器。PSYoungGen使用Parallel Scavenge收集器  DefNew使用Parallel New收集器
- []内的数量变化，代表该区域回收前后已使用的容量，[]外的数字变化代表GC前后堆的已使用容量
- 消耗的时间。[]紧跟的时间代表GC消耗时间。user:用户态消耗的CPU时间.sys:内核态消耗的CPU时间.real:操作从开始到结束经过的墙钟时间
- heap:堆内存目前各个年代的区域的内存情况

