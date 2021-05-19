#### AQS(AbstractQueuedSynchronizer)

AQS 的锁在内部实现上分为独占和共享两种模式。

独占锁： 锁在一个时间点只能被一个线程占有。ReentrantLock 和 ReentrantReadWriteLock.Writelock 是独占锁。

共享锁：同一个时候能够被多个线程获取的锁，能被共享的锁。JUC包中ReentrantReadWriteLock.ReadLock，CyclicBarrier，CountDownLatch和Semaphore都是共享锁

独占锁加锁过程：

> tryAcquire() 获取资源（cas 设置state 为1），成功直接返回
>
> 没成功，使用addWaiter()将线程封装成node,添加到queue的tail.并标记为独占模式
>
> acquireQueued()使线程在等待队列中休息，unpark()后被唤醒尝试获取资源，获取后返回。如果等待过程中被中断，则返回true,否则false.
>
> 如果被中断过，不响应，直到获取资源后才进行自我中断，将中断补上

共享锁加锁过程：

>  tryAcquireShared() 获取资源，成功获取资源后，如果还有剩余资源，那么还会唤醒后面的线程来尝试获取资源。
>
> 失败则通过doAcquireShared()进入等待队列，直到被unpark()/interrupt()并成功获取到资源才返回。整个等待过程也是忽略中断的

独占锁解锁过程：

> tryRelease() 释放资源，成功返回true，否则返回false。
>
> 如果已经彻底释放资源，则调用 unparkSuccessor 方法，唤醒等待队列里的下一个线程

共享锁解锁过程：

>  tryReleaseShared() 释放资源，如果成功，那么就调用 doReleaseShared() 方法唤醒后面的线程。

#### 公平锁和非公平锁

ReentrantLock 是基于 AQS 实现的独占锁。内部可分为非公平和公平锁。默认非公平。

非公平锁：线程获取锁的顺序和调用lock的顺序无关，全凭运气

公平锁：线程获取锁的顺序和调用lock的顺序一样

公平锁和非公平锁只在加锁时有差异。

非公平加锁

```
if(CAS(state,0,1)){
	setExclusiveOwnerThread= currentthread
}else{
	state = getState();
	if(state = 0 ){
		if(CAS(state,0,1)){
			setExclusiveOwnerThread= currentthread
		}else{
            //和下面的else一样
		}
	}else{
		if(getOwnerThread ==currentthread){
			state++;
		}else{
			addWaiter()
		}
	}
}
```

公平加锁

```
state = getState()
if (1.state == 0 && 2,currentThread == node.pre && 3.CAS(state,0,1)){
    setExclusiveOwnerThread= currentthread
}else{
	if(getOwnerThread ==currentThread){
			state++;
		}else{
			addWaiter()
		}
}
```

https://blog.csdn.net/sinat_32873711/article/details/106619980

#### CountDownLatch

初始化时用给定的数字。调用 countDown() 方法每次减1，在到达 0 之前，await 方法会一直受阻塞。

之后，会释放所有等待的线程，await 的所有后续调用都将立即返回

#### CyclicBarrier

使用了ReentrantLock以及Condition来实现.每当一个线程调用  await() 方法时，将剩余拦截的线程数减 1，然后判断剩余拦截数是否为 0，如果不是进入 Lock 对象的条件队列等待。如果是，执行 barrierAction 对象的 Runnable 方法，然后signalAll方法唤醒所有的线程，并将parties重新赋值给count

#### Semaphore

可以用来控制最大并发量。初始定义好有几个信号，然后在需要获取信号的地方调用acquire方法，执行完成后，需要调用release方法回收信号.

acquire方法和CountDownLatch是一样的，只是tryAcquireShared区分了公平和非公平方式。获取到信号相当于加共享锁成功，否则则进入队列阻塞等待；而release方法和读锁解锁方式也是一样的，只是每次release都会将state+1。

#### 阻塞队列

ArrayBlockingQueue：数组组成有界阻塞队列

LinkedBlockingQueue：链表组成有界阻塞队列。也可以不设上限默认Integer.MAX_VALUE

DelayQueue：优先级的无界阻塞队列。对元素进行持有直到一个特定的延迟到期。注入其中的元素必须实现 Delayed 接口

PriorityBlockingQueue ：优先级排序的无界阻塞队列。所有插入的数据必须实现Comparable接口

SynchronousQueue：只能存储一个元素的阻塞队列。

LinkedTransferQueue：一个由链表结构组成的无界阻塞队列。

LinkedBlockingDeque：一个由链表结构组成的双向阻塞队列。

原理：阻塞队列实现阻塞同步的方式很简单，使用的就是是lock锁的多条件（condition）阻塞控制。使用BlockingQueue封装了根据条件阻塞线程的过程，而我们就不用关心繁琐的await/signal操作了。

#### 如何使用aqs写出一个互斥锁



























> 引用:https://zhuanlan.zhihu.com/p/265096774