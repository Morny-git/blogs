#### 线程五种状态

**new**: implements runnable  ;extends thread
**runnable**：调用start()方法。该状态的线程位于可运行线程池中，等待被线程调度选中，获取cpu 的使用权
**running**:  执行代码
**blocked:**	放弃cpu使用权。
等待阻塞：wait()方法
同步阻塞：获取同步锁时，该锁被占用，被放到锁池中
其他阻塞：运行sleep，join 或发出io请求时。
**dead**:	执行完成或异常退出

##### **sleep yield  join wait**

**sleep**:当前线程进入阻塞，但不释放对象锁，millis后线程自动苏醒进入可运行状态
**yield**:当前线程放弃获取的cpu时间片，由运行状态变会可运行状态，让OS再次选择线程.能不能抢到不管
**join**:在自己当前线程加入你调用Join的线程（），本线程等待。等调用join方法的线程运行完了，自己再去执行
**wait**:当前线程释放对象锁，进入等待队列。依靠notify()/notifyAll()唤醒或者wait(long timeout)timeout时间到自动唤醒
**notify**:唤醒在此对象监视器上等待的单个线程，选择是任意性的。notifyAll()唤醒在此对象监视器上等待的所有线程

##### implements extends

extends:Java是单继承机制，不允许同时继承多个类
implements:多个线程共享一个对象

##### star run

**star** ：变成可执行状态，真正实现了多线程运行
**run**: 直接执行

**多线程就是分时利用CPU，宏观上让所有线程一起执行 ，也叫并发。**



#### synchronized的三种应用方式

1.作用于实例方法，当前实例加锁，进入同步代码前要获得当前实例的锁，不同的对象实例没有竞争关系；

2.作用于静态方法，当前类加锁，进去同步代码前要获得当前类对象的锁，是这个类所有的对象竞争一把锁；

3.作用于代码块，其中普通代码块 如Synchronized（obj） 这里的obj 可以为类中的一个属性、也可以是当前的对象，它的同步效果和修饰普通方法一样；Synchronized方法 （obj.class）静态代码块它的同步效果和修饰静态方法类似

####  Java对象头

对象头含有三部分：Mark Word（存储对象自身运行时数据）、Class Metadata Address（存储类元数据的指针）、Array length（数组长度，只有数组类型才有）

![image-20210327223715873](..\image\java\markdown.png)

> 引用：https://www.cnblogs.com/zaizhoumo/p/7700161.html

#### synchronized 的实现原理

**synchronized 代码块**是通过 monitorenter 和 monitorexit 指令实现的。在一个线程退出同步块时，线程释放monitor对象，它的作用是把CPU缓存数据（本地缓存数据）刷新到主内存中，从而实现该线程的行为可以被其它线程看到。在其它线程进入到该代码块时，需要获得monitor对象，它在作用是使CPU缓存失效。

**synchronized 方法**在 vm 字节码层面并没有任何特别的指令来实现被 synchronized 修饰的方法，而是在 **Class 文件的方法表**中将该方法的 **access_flags 字段中的 synchronized 标志位置1**，表示该方法是同步方法。

#### 锁分类

锁的实现有偏向锁、轻量级锁和重量级锁，其中偏向锁和轻量级锁是 JDK 针对锁的优化措施。在多线程的竞争下锁会升级，依次从无锁01->偏向锁01 -> 轻量级锁00 -> 重量级锁10，这里的锁只能升级但不能降级。在 Java 对象头中的 Mark Word 中存储了关于锁的标志位.

引入偏向锁主要目的是：为了在无多线程竞争的情况下尽量减少不必要的轻量级锁执行路径(在无竞争的情况下把整个同步都消除掉，连 CAS 操作都不做了)。

引入轻量级锁的主要目的是：在没有多线程竞争的前提下，减少传统的重量级锁使用操作系统互斥量产生的性能消耗(在无竞争的情况下使用 CAS 操作去消除同步使用的互斥量)。

重量级锁通过对象内部的监视器（monitor）实现，其中 monitor 的本质是依赖于底层操作系统的 Mutex Lock 实现，操作系统实现线程之间的切换需要从用户态到内核态的切换，切换成本非常高。锁的优缺点对比如下

|          | 优点                                                   | 缺点                                         | 适用情况                           |
| -------- | ------------------------------------------------------ | -------------------------------------------- | ---------------------------------- |
| 偏向锁   | 加锁和解锁不需要额外的消耗，和执行非同步代码相差无几。 | 如果线程存在锁竞争，需要额外的锁撤销的消耗。 | 适用于只有一个线程访问同步块的情况 |
| 轻量级锁 | 竞争的线程不会阻塞，提高了响应速度                     | 长时间得不到锁的线程使用自旋消耗CPU          | 追求响应速度。同步代码执行非常快   |
| 重量级锁 | 线程竞争不会使用自旋，不会消耗CPU                      | 线程出现竞争时会阻塞，响应速度慢             | 追求吞吐量。同步代码执行时间长     |

此外，jdk 1.6 对锁的实现引入了大量的优化，如自旋锁、适应性自旋锁、锁消除、锁粗化等技术来减少锁操作的开销。

自旋锁

所谓自旋锁，就是让该线程等待一段时间，不会被立即挂起，看持有锁的线程是否会很快释放锁。

适应自旋锁

自适应就意味着自旋的次数不再是固定的，它是由前一次在同一个锁上的自旋时间及锁的拥有者的状态来决定。==对于同一个锁，上一个线程如果自旋成功了，那么下次自旋的次数会更加多==，因为虚拟机认为既然上次成功了，那么此次自旋也很有可能会再次成功，那么它就会允许自旋等待持续的次数更多。==反之，如果对于某个锁，很少有自旋能够成功的，那么在以后要获得这个锁的时候自旋的次数会减少甚至省略掉自旋过程，以免浪费处理器资源==。

锁消除

在有些情况下，JVM 检测到不可能存在共享数据竞争，这时 JVM 会对这些同步锁进行锁消除。锁消除的依据是逃逸分析的数据支持。

锁粗化

将多个连续的加锁、解锁操作连接在一起，扩展成一个范围更大的锁，来减少频繁的加锁和释放锁带来的性能损耗。

#### 线程池的实现原理

CachedThreadPool，FixedThreadPool，SingleThreadExecutor，ScheduleThreadPool，ScheduledThreadPoolExecutor。它们分别是通过 Executors 的静态方法创建出来的。而他们底层是通过 ThreadPoolExecutor 类创建出来的。创建线程池时会传入以下参数，他们分别是：

1. corePoolSize：核心线程池的大小，在线程池被创建之后，其实里面是没有线程的。（当然，调用 prestartAllCoreThreads() 或者 prestartCoreThread() 方法会预创建线程，而不用等着任务的到来）。当有任务进来的时候，才会创建线程。当线程池中的线程数量达到corePoolSize之后，就把任务放到缓存队列当中。（就是 workQueue ）。
2. maximumPoolSize：最大线程数量是多少。它标志着这个线程池的最大线程数量。如果没有最大数量，当创建的线程数量达到了 某个极限值，到最后内存肯定就爆掉了。
3. keepAliveTime：当线程没有任务时，最多保持的时间，超过这个时间就被终止了,默认值 60 秒。默认情况下，只有线程池中线程数量大于 corePoolSize 时，keepAliveTime 值才会起作用。也就说，只有在线程池线程数量超出 corePoolSize  了。我们才会把超时的空闲线程给停止掉。否则就保持线程池中有 corePoolSize 个线程就可以了。
4. Unit：参数keepAliveTime的时间单位，就是 TimeUnit类当中的几个属性。
5. workQueue：用来存储待执行任务的队列，不同的线程池它的队列实现方式不同（因为这关系到排队策略的问题）比如有以下几种:
   - ArrayBlockingQueue：基于数组的队列，创建时需要指定大小。
   - LinkedBlockingQueue：基于链表的队列，如果没有指定大小，则默认值是 Integer.MAX_VALUE。（newFixedThreadPool和newSingleThreadExecutor使用的就是这种队列），吞吐量通常要高于ArrayBlockingQuene。
   - SynchronousQueue：一个不存储元素的阻塞队列，每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态，吞吐量通常要高于LinkedBlockingQuene（newCachedThreadPool使用的就是这种队列）。
6. threadFactory:线程工厂，用来创建线程。通过自定义的线程工厂可以给每个新建的线程设置一个具有识别度的线程名。
7. Handler：拒绝执行任务时的策略，一般来讲有以下四种策略：
   - ThreadPoolExecutor.AbortPolicy (默认的执行策略)丢弃任务，并抛出 RejectedExecutionException 异常。
   - ThreadPoolExecutor.CallerRunsPolicy：该任务被线程池拒绝，由调用 execute 方法的线程执行该任务。
   - ThreadPoolExecutor.DiscardOldestPolicy ： 抛弃队列最前面的任务，然后重新尝试执行任务。
   - ThreadPoolExecutor.DiscardPolicy，丢弃任务，不过也不抛出异常。

#### 线程池最大线程数如何确定

-  对CPU密集型任务来说，要进行大量的计算，消耗CPU资源。要最高效地利用CPU，计算密集型任务同时进行的数量应当等于CPU的核心数。**线程数=CPU总核心数+1**  (+1是为了利用等待空闲)
- 对IO密集型任务来说，大部分时间都消耗在等待IO操作完成。任务越多，CPU效率越高。**线程数=2*CPU总核心数+1**

#### 并发环境下如何设置线程数

- 高并发、任务执行时间短：线程池线程数可以设置为CPU核数+1，减少线程上下文的切换
- 并发不高、任务执行时间长：
  - IO密集型，可以增大线程数，让CPU更多的执行任务
  - CPU密集型，也就是集中在计算上，可以设置少一些，减少上下文切换
- 高并发，执行时间长：重点不在线程池上。

#### 几种线程池的比较

##### CachedThreadPool

**优点**：

工作线程的创建数量几乎没有限制(其实也有限制的,数目为Interger. MAX_VALUE), 这样可灵活的往线程池中添加线程。

如果长时间没有往线程池中提交任务，即如果工作线程空闲了指定的时间(默认为1分钟)，则该工作线程将自动终止。终止后，如果你又提交了新的任务，则线程池重新创建一个工作线程。

**缺点**：

在使用CachedThreadPool时，一定要注意控制任务的数量，否则，由于大量线程同时运行，很有会造成系统瘫痪。

##### FixedThreadPool

创建一个指定工作线程数量的线程池。每当提交一个任务就创建一个工作线程，如果工作线程数量达到线程池初始的最大数，则将提交的任务存入到池队列中。定长线程池的大小最好根据系统资源进行设置如Runtime.getRuntime().availableProcessors()

**优点**：

FixedThreadPool是一个典型且优秀的线程池，它具有线程池提高程序效率和节省创建线程时所耗的开销的优点。

**缺点：**

但是，在线程池空闲时，即线程池中没有可运行任务时，它不会释放工作线程，还会占用一定的系统资源。

##### SingleThreadExecutor

创建一个单线程化的Executor，即只创建唯一的工作者线程来执行任务，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行。如果这个线程异常结束，会有另一个取代它，保证顺序执行。单工作线程最大的特点是可保证顺序地执行各个任务，并且在任意给定的时间不会有多个线程是活动的。

#####  ScheduleThreadPool

创建一个定长的线程池，而且支持定时的以及周期性的任务执行，支持定时及周期性任务执行。

#### 线程池任务的提交流程

1. 如果当前线程池线程数目小于 corePoolSize（核心池还没满呢），那么就创建一个新线程去处理任务。
2. 如果核心池已经满了，来了一个新的任务后，会尝试将其添加到任务队列中，如果成功，则等待空闲线程将其从队列中取出并且执行，如果队列已经满了，则继续下一步。
3. 此时，如果线程池线程数量 小于 maximumPoolSize，则创建一个新线程执行任务，否则，那就说明线程池到了最大饱和能力了，没办法再处理了，此时就按照拒绝策略来处理。（就是构造函数当中的 Handler 对象）。
4. 如果线程池的线程数量大于 corePoolSize，则当某个线程的空闲时间超过了 keepAliveTime，那么这个线程就要被销毁了，直到线程池中线程数量不大于 corePoolSize 为止。

#### 线程池的 shutdown 和 shutdownNow 方法的区别是什么

1. shutdown 设置状态为 SHUTDOWN，而 shutdownNow 设置状态为 STOP
2. shutdown 只中断空闲的线程，已提交的任务可以继续被执行，而 shutdownNow 中断所有线程
3. shutdown 无返回值，shutdownNow 返回任务队列中还未执行的任务

#### volatie

1.可见性：当线程要对这个变量执行的写操作，都不会写入本地缓存，而是直接刷入主内存中。当线程读取被 volatile 关键字修饰的变量时，也是直接从主内存中读取。（简单的说，一个线程修改的状态对另一个线程是可见的）。注意：volatile 不能保证原子性

2.禁止指令重排：有volatile修饰的变量，赋值后多执行了一个 “load addl $0x0, (%esp)” 操作，这个操作相当于一个内存屏障，保证指令重排序时不会把后面的指令重排序到内存屏障之前的位置。

#### as-if-serial 语义和 happens-before 规则

as-if-serial ：不管怎么重排序，单线程程序的执行结果不能被改变

happens-before 规则:jvm会对代码进行编译优化，指令会出现重排序的情况，为了避免编译优化对并发编程安全性的影响，需要happens-before规则定义一些禁止编译优化的场景

-  **程序次序规则(Program Order Rule)：**在一个线程内，按照代码顺序，书写在前面的操作先行发生于书写在后面的操作。准确地说应该是控制流顺序而不是代码顺序，因为要考虑分支、循环等结构。

-  **监视器锁定规则(Monitor Lock Rule)**：一个 unlock 操作先行发生于后面对同一个对象锁的lock操作。这里强调的是同一个锁，而“后面”指的是时间上的先后顺序，如发生在其他线程中的 lock 操作。

- **volatile变量规则(Volatile Variable Rule):**对一个 volatile 变量的写操作早于后面对这个变量的读操作，这里的“后面”也指的是时间上的先后顺序。
- 线程启动规则(Thread Start Rule)：Thread 独享的 start() 方法先行于此线程的每一个动作。

- 线程终止规则(Thread Termination Rule)：线程中的每个操作都先行发生于对此线程的终止检测，我们可以通过 Thread.join() 方法结束、Thread.isAlive() 的返回值检测到线程已经终止执行。

- 线程中断规则(Thread Interruption Rule)：对线程 interrupte() 方法的调用优先于被中断线程的代码检测到中断事件的发生，可以通过 Thread.interrupted() 方法检测线程是否已中断。

- 对象终结原则(Finalizer Rule)：一个对象的初始化完成(构造函数执行结束)先行发生于它的 finalize() 方法的开始。

- **传递性(Transitivity)：**如果操作A先行发生于操作B，操作B先行发生于操作C，那就可以得出操作A先行发生于操作C的结论。

此外，JMM对 volatile 和 final 的语义做了扩展。

对 volatile 语义的扩展保证了 volatile 变量在一些情况下不会重排序，volatile 的 64 位变量 double 和 long 的读取和赋值操作都是原子的。

对于基本类型的final域，编译处理器遵循两个重排序规则：

- 在构造函数内对一个final域的写入，与随后把这个被构造对象的引用赋值给一个引用变量，这两个操作之间不能重排序。（StoreStore屏障）
- 初次读一个包含final域的对象的引用，与随后初次读这个final域，这两个操作之间不能重排序。（LoadLoad屏障）

对于引用类型的final域，编译处理器增加如下约束：

- 在构造函数内对一个final引用的对象的成员域的写入，与随后在构造函数外把这个被构造对象的引用赋值给一个引用变量，这两个操作之间不能重排序。

#### 内存屏障

内存屏障，是一种CPU指令，用于控制特定条件下的重排序和内存可见性问题。Java编译器也会根据内存屏障的规则禁止重排序。

LoadLoad屏障：对于这样的语句Load1; LoadLoad; Load2，在Load2及后续读取操作要读取的数据被访问前，保证Load1要读取的数据被读取完毕。

StoreStore屏障：对于这样的语句Store1; StoreStore; Store2，在Store2及后续写入操作执行前，保证Store1的写入操作对其它处理器可见。

LoadStore屏障：对于这样的语句Load1; LoadStore; Store2，在Store2及后续写入操作被刷出前，保证Load1要读取的数据被读取完毕。

StoreLoad屏障：对于这样的语句Store1; StoreLoad; Load2，在Load2及后续所有读取操作执行前，保证Store1的写入对所有处理器可见。它的开销是四种屏障中最大的。 在大多数处理器的实现中，这个屏障是个万能屏障，兼具其它三种内存屏障的功能。

#### synchronization

Synchronized 内部通过monitor保证线程的可见性。monitorexit时，缓存刷新到主内存。monitorenter 使得缓存失效。

#### final

该关键字修饰的变量在构造函数中初始化，正确构造一个对象后，该字段对其他对象是可见的。

#### volatile

主要用于线程之间的通信。保证写的内容刷新到主内存，读之前，cpu缓存失效，从主内存中读取。

#### synchronized与lock区别

|        | synchronized                                                 | lock                                                    |
| ------ | ------------------------------------------------------------ | ------------------------------------------------------- |
| 释放锁 | 线程执行完代码块；发生异常，jvm让线程释放                    | 线程执行完代码；发生异常不会主动，必须手动在finally释放 |
| 锁状态 | 不可判断                                                     | 可判断                                                  |
| 锁类型 | 可重入 不可中断 非公平                                       | 可重入 可/不可中断 可公平也可不公平                     |
| 性能   | jdk5时，性能非常低效。jdk6做了优化，有适应自旋，锁消除，锁粗化，轻量级锁，偏向锁等等。性能不比lock差 | 竞争激烈时，性能更优越                                  |
| 锁类型 | 悲观锁 独占锁                                                | 悲观锁                                                  |

#### 虚假唤醒

是一个表象，即在多处理器的系统下发出wait的程序有可能在没有notify唤醒的情形下苏醒继续执行。

可通过代码避免

```
synchronized (obj) {
     while (<condition does not hold>)
         obj.wait();
     ... // Perform action appropriate to condition
 }
```

#### JUC

https://www.jianshu.com/p/1f19835e05c0

#### 死锁产生的四个必要条件

- 互斥条件：独占性的使用资源。A获取到资源，B要获取时一直等待
- 不可剥夺条件：获取的资源未使用前不能被其他进程剥夺
- 请求和保持条件：进程每次申请它所需要的一部分资源，在申请新的资源的同时，继续占用已分配到的资源。
- 循环等待条件：A等待B占有的资源。B等待A获取的资源

> 也就是A有resA，B有resB,A在等B的resB,B在等A的resA.但是都不放手

#### 处理死锁的方法

- **预防死锁**：通过设置某些限制条件，去破坏产生死锁的四个必要条件中的一个或几个条件，来防止死锁的发生。
- **避免死锁**：在资源的动态分配过程中，用某种方法去防止系统进入不安全状态，从而避免死锁的发生。
- **检测死锁**：允许系统在运行过程中发生死锁，但可设置检测机构及时检测死锁的发生，并采取适当措施加以清除。
- **解除死锁**：当检测出死锁后，便采取适当措施将进程从死锁状态中解脱出来

#### 死锁

```
static Object res1 = "res1";
	static Object res2 = "res2";
	public static void main(String[] args) {
		Thread threadA = new Thread(()->{try {
			while (true){
				synchronized (res1){
					System.out.println("thread A get res1");
					Thread.sleep(10);
					synchronized (res2) {
						System.out.println("thread A get res2");
					}
				}
			}
		} catch (InterruptedException e) {
			e.printStackTrace();
		}});
		Thread threadB = new Thread(()->{try {
			while (true){
				synchronized (res2){
					System.out.println("thread B get res2");
					Thread.sleep(10);
					synchronized (res1) {
						System.out.println("thread B get res1");
					}
				}
			}
		} catch (InterruptedException e) {
			e.printStackTrace();
		}});
		threadA.start();
		threadB.start();
	}
```

