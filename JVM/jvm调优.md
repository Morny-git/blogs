平时的开发中会遇到的问题：

- OOM
- 内存泄露
- 线程死锁
- 锁争用
- 进程消耗CPU过高

#### **jps**(JVM Process Status Tool)

显示指定系统内正在运行的JVM进程

#### jinfo pid

用于查询当前运行这的JVM属性和参数的值

#### jstack

查看某个进程内的线程的堆栈信息。可以根据堆栈信息定位到具体代码

#### jstat(JVM statistics Monitoring)

用于监视虚拟机运行时状态信息的命令，它可以显示出虚拟机进程中的类装载、内存、垃圾收集、JIT编译等运行数据
`jstat -gc 1262` [interval] [count] ：间隔  总次数

 S0C    S1C     S0U     S1U   EC       EU        OC         OU        PC       PU         YGC    YGCT    FGC    FGCT     GCT   

26112.0 24064.0 6562.5  0.0   564224.0 76274.5   434176.0   388518.3  524288.0 42724.7    320    6.417   1      0.398

```
C即Capacity 总容量，U即Used 已使用的容量
```

#### jmap(JVM Memory Map)

用于生成heap dump文件，如果不使用这个命令，还可以使用-XX:+HeapDumpOnOutOfMemoryError参数来让虚拟机出现OOM的时候·自动生成dump文件。 jmap不仅能生成dump文件，还阔以查询finalize执行队列、Java堆和永久代的详细信息，如当前使用率、当前使用的是哪种收集器等

#### 排查一次线程占用cpu太高

- top命令展示出所有的程序占用cpu情况，按照有高到低排序
- jps 命令展示现存活的程序。查看cpu占比高的几个程序是不是自己的
- top -Hp pid 查看进程中最耗费CPU的线程.此时展示的线程pid是十进制的
- printf '%x' 十进制   可转换成16进制的。在jstack中线程pid通过十六进制展现
- jstack pid  输出该进程的堆栈信息，在信息中寻找十六进制线程的情况

> https://blog.csdn.net/jek123456/article/details/80423199

#### 排查oom

dts项目开始时，常常监控日志。发现一直没有商品入库日志。因启动命令中配置-XX:+PrintGCDetails ,查看gc.log，发现频繁fullgc。且释放空间不大。可能存在隐患。定时使用jmap -histo:live pid |head -20 查看堆内存中的对象数目，发现prometheus 占用内存居高不下。问题可能出现在peometheus中,排查代码。也可以使用jmap dump 出hprof文件使用mat分析。

后期项目正常运行后，给项目加监控，如果项目down掉，发送报警，且jmap dump 出内存情况，再自动重启

> 更多案例查看https://blog.csdn.net/u013256816/article/details/94518811