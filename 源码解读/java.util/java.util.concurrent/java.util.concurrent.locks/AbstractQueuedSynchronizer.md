# AbstractQueuedSynchronizer

`AbstractQueuedSynchronizer`翻译过来就是“抽象队列同步器”，它定义了一套多线程访问共同资源的同步器框架，之后的很多同步实现类都依赖于它，比如`ReentrantLock`、`ReentrantReadWriteLock`、`Semaphore`、`CountDownLatch`等等。可以说，`AbstractQueuedSynchronizer`，简称`AQS`，是整个`java.util.concurrent`的核心。

其结构如下：

![AbstractQueuedSynchronizer](http://ovn0i3kdg.bkt.clouddn.com/AbstractQueuedSynchronizer.png?imageView/2/w/500)
![AbstractQueuedSynchronizer](http://ovn0i3kdg.bkt.clouddn.com/AbstractQueuedSynchronized_.png?imageView/2/w/500)


### 框架
该类维护了一个共享资源和一个FIFO线程等待队列（多线程争用资源被阻塞时会进入此队列），下面这个图可以用来描述这个关系：
![共享资源和等待队列](https://images2015.cnblogs.com/blog/721070/201705/721070-20170504110246211-10684485.png)

如图中所示，该类用一个变量`volatile int state`来表示这个被争用的资源，队头和队尾由两个变量来记录：
```java
private transient volatile Node head;
private transient volatile Node tail;
```

另外从图中可以看到，这个队列叫做“CLH队列”。"CLH"即“Craig, Landin, and Hagersten  locks”，即自旋锁，什么是自旋锁？自旋锁采用让当前线程不停地的在循环体内执行实现，当循环的条件被其他线程改变时 才能进入临界区。关于自旋锁的更多内容参见各类锁的辨析。

对于共享资源，就是这个state，AQS定义三种访问方式——`getState`、`setState`和`compareAndSetState`。

对于共享资源的占用方式，该类定义了两种方式：
1. Exclusive，即独占式的，每次只能有一个线程能够执行。实现这种方式的锁例如`ReentrantLock`。
1. Share，即共享式的，多个线程能够同时执行。实现这种方式的锁例如`Semaphore`、`CountDownLatch`。

AQS是最基础的锁的框架定义，具体的同步争用器实现争用方式是不同的，它们借在AQS的线程等待队列的维护功能（如获取资源失败入队/唤醒出队等）基础上，只需实现各自的共享资源state获取和释放功能的实现，也就是实现了各具特色的锁。


自定义的同步争用器主要需要实现以下几种方法：

| 方法名 |作用  |
| :------------- | :------------- |
|    isHeldExclusively()    | 该线程是否正在独占资源。只有用到condition才需要去实现它。     |
|tryAcquire(int)   |  独占方式。尝试获取资源，成功则返回true，失败则返回false。 |
|  tryRelease(int) | 独占方式。尝试释放资源，成功则返回true，失败则返回false。|
|tryAcquireShared(int)   | 共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。  |
|tryReleaseShared(int)   |  共享方式。尝试释放资源，如果释放后允许唤醒后续等待结点返回true，否则返回false。 |



例如`ReentrantLock`，以`ReentrantLock`为例，`state`初始化为0，表示未锁定状态。A线程`lock()`时，会调用`tryAcquire()`独占该锁并将`state` + 1。此后，其他线程再`tryAcquire()`时就会失败，直到A线程`unlock()`到`state = 0`（即释放锁）为止，其它线程才有机会获取该锁。当然，释放锁之前，A线程自己是可以重复获取此锁的（`state`会累加），这就是可重入的概念。但要注意，获取多少次就要释放多么次，这样才能保证`state`是能回到零态的。

例如`CountDownLatch`，任务分为`N`个子线程去执行，`state`也初始化为`N`（注意`N`要与线程个数一致）。这`N`个子线程是并行执行的，每个子线程执行完后`countDown()`一次，`state`会`CAS`减1。等到所有子线程都执行完后(即`state = 0`)，会`unpark()`主调用线程，然后主调用线程就会从`await()`函数返回，继续后余动作。

一般来说，自定义同步器要么是独占方法，要么是共享方式，他们也只需实现`tryAcquire-tryRelease`、`tryAcquireShared-tryReleaseShare`d中的一种即可。但AQS也支持自定义同步器同时实现独占和共享两种方式，如`ReentrantReadWriteLock`。


以上大概就是AQS对于JUC所作的贡献了，它之所以重要，就是提供了一个最基础、也是最重要的线程等待队列。

下面就从源码角度来分析一下AQS对于线程等待队列的实现。

### public abstract class AbstractQueuedSynchronizer extends AbstractOwnableSynchronizer implements java.io.Serializable
类声明，这是一个抽象类，继承的抽象类`AbstractOWnableSynchronizer`类也是JUC中的一个重要的类，从名字就可以看出，“拥有某个锁的同步器”，其实就是排他锁。类结构如下：

![AbstractOwnableSynchronizer](http://ovn0i3kdg.bkt.clouddn.com/AbstractOwnableSynchronizer.png)

### protected AbstractQueuedSynchronizer() { }
只有一个构造方法，且私有。那么如何初始化呢？这种与底层实现息息相关的类，一定不能让开发人员随便构造的，那么JVM总得初始化吧，如果实现？静态初始块。

```java

private static final Unsafe unsafe = Unsafe.getUnsafe();
private static final long stateOffset;
private static final long headOffset;
private static final long tailOffset;
private static final long waitStatusOffset;
private static final long nextOffset;

static {
    try {
        stateOffset = unsafe.objectFieldOffset
            (AbstractQueuedSynchronizer.class.getDeclaredField("state"));
        headOffset = unsafe.objectFieldOffset
            (AbstractQueuedSynchronizer.class.getDeclaredField("head"));
        tailOffset = unsafe.objectFieldOffset
            (AbstractQueuedSynchronizer.class.getDeclaredField("tail"));
        waitStatusOffset = unsafe.objectFieldOffset
            (Node.class.getDeclaredField("waitStatus"));
        nextOffset = unsafe.objectFieldOffset
            (Node.class.getDeclaredField("next"));

    } catch (Exception ex) { throw new Error(ex); }
}

private transient volatile Node head;
private transient volatile Node tail;
private volatile int state;
```
这里进行了初始化的操作，`Unsafe`这个类封装了`CAS`的操作，是用本地方法实现的，我们不用去管到底如何实现。总之，它获取到了共享资源state、共享队列的头head和共享队列的尾tail，第一个等待节点和其后一个节点的地址，就此完成了初始化工作。

### public final void acquire(int arg){...}
这个方法是**独占模式**下，线程获取共享资源的顶层入口，如果线程获取到了锁，那么线程直接返回，否则进入等待队列，直到获取到资源。**整个过程忽略中断的影响**，源码如下：
```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```
它的执行过程分为以下四个步骤：
1. `tryAcquire()`：尝试直接去获取资源，如果成功则直接返回；
2. `addWaiter()`：将该线程加入等待队列的尾部，并标记为独占模式；
3. `acquireQueued()`：使线程在等待队列中获取资源，一直获取到资源后才返回。如果在整个等待过程中被中断过，则返回true，否则返回false。
4. 如果线程在等待过程中被中断过，它是不响应的。只是获取资源后才再进行自我中断`selfInterrupt()`，将中断补上。

下面就每个过程进行具体的讲解：
1. tryAcquire(int)
此方法尝试去获取独占资源。如果获取成功，则直接返回true，否则直接返回false。这也正是`tryLock()`的语义，还是那句话，当然不仅仅只限于tryLock()。如下是`tryAcquire()`的源码：
```java
protected boolean tryAcquire(int arg) {
    throw new UnsupportedOperationException();
}
```
诶？？？说好的获取资源呢？为什么抛出异常了？好吧，之前说到过，AQS只是一个框架，具体的实现还是依赖了具体的同步器的实现，这里只是提供了一个接口而已。诶，那为什么不用抽象方法？那是因为这只是独占模式下的实现，而共享模式下不用实现这个方法，如果写成抽象方法，那么采取共享模式的同步器也必须实现这个方法，这是不合理的。

2. addWaiter(Node)
这个方法是将当前线程加入到等待队伍的队尾，并且返回当前线程所在的节点。
```java
private Node addWaiter(Node mode) {
    //以给定模式构造结点。mode有两种：EXCLUSIVE（独占）和SHARED（共享）
    Node node = new Node(Thread.currentThread(), mode);

    //尝试快速方式直接放到队尾。
    Node pred = tail;
    if (pred != null) {
        node.prev = pred;
        if (compareAndSetTail(pred, node)) {
            pred.next = node;
            return node;
        }
    }

    //上一步失败则通过enq入队。
    enq(node);
    return node;
}
```
其中`enq`方法定义如下：
```java
private Node enq(final Node node) {
    //CAS"自旋"，直到成功加入队尾
    for (;;) {
        Node t = tail;
        if (t == null) { // 队列为空，创建一个空的标志结点作为head结点，并将tail也指向它。
            if (compareAndSetHead(new Node()))
                tail = head;
        } else {//正常流程，放入队尾
            node.prev = t;
            if (compareAndSetTail(t, node)) {
                t.next = node;
                return t;
            }
        }
    }
}
```
如果你看过`AtomicInteger.getAndIncrement()`函数源码，那么相信你一眼便看出这段代码的精华。**CAS自旋volatile变量**，是一种很经典的用法。还不太了解的，自己去百度一下吧。



参考
* [java锁的种类以及辨析（一）：自旋锁](http://ifeve.com/java_lock_see1/)
* [Java并发之AQS详解](https://www.cnblogs.com/waterystone/p/4920797.html)
