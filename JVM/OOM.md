## OutOfMemoryError

> > > > > ` OutOfMemoryError: Java heap space`
> > > > >
> > > > > 原因：对象不能被分配到堆内存中
> > > > >
> > > > > > `OutOfMemoryError: PermGen space`
> > > > > >
> > > > > > 原因：类或者方法不能被加载到老年代。它可能出现在一个程序加载很多类的时候，比如引用了很多第三方的库
> > > > > >
> > > > > > `OutOfMemoryError: Requested array size exceeds VM limit`
> > > > >
> > > > > 原因：创建的数组大于堆内存的空间
> > > > >
> > > > > `OutOfMemoryError: request <size> bytes for <reason>. Out of swap space?``

原因：分配本地分配失败。JNI、本地库或者Java虚拟机都会从本地堆中分配内存空间

`OutOfMemoryError: <reason> <stack trace>（Native method）`

原因：同样是本地方法内存分配失败，只不过是JNI或者本地方法或者Java虚拟机发现

unable to create new native thread

是创建了太多的线程，而能创建的线程数是有限制的，导致了这种异常的发生

> GC overhead limit exceeded
>
> 是在并行或者并发回收器在GC回收时间过长、超过98%的时间用来做GC并且回收了不到2%的堆内存，然后抛出这种异常进行提前预警，用来避免内存过小造成应用不能正常工作
>
> ​	