# AbstractQueuedSynchronizer

> AQS是干什么的？总结成一句话： AQS利用CAS原子操作维护自身的状态，结合LockSupport对线程进行阻塞和唤醒从而实现更为灵活的同步操作。


`AbstractQueuedSynchronizer`翻译过来就是“抽象队列同步器”，它定义了一套多线程访问共同资源的同步器框架，之后的很多同步实现类都依赖于它，比如`ReentrantLock`、`ReentrantReadWriteLock`、`Semaphore`、`CountDownLatch`等等。可以说，`AbstractQueuedSynchronizer`，简称`AQS`，是整个`java.util.concurrent`的核心。

其结构如下：

![AbstractQueuedSynchronizer](http://ovn0i3kdg.bkt.clouddn.com/AbstractQueuedSynchronizer.png?imageView/2/w/500)
![AbstractQueuedSynchronizer](http://ovn0i3kdg.bkt.clouddn.com/AbstractQueuedSynchronized_.png?imageView/2/w/500)


## 框架
该类维护了一个共享资源和一个FIFO线程等待队列（多线程争用资源被阻塞时会进入此队列），下面这个图可以用来描述这个关系：
![共享资源和等待队列](http://ovn0i3kdg.bkt.clouddn.com/%E5%85%B1%E4%BA%AB%E8%B5%84%E6%BA%90%E5%92%8C%E7%AD%89%E5%BE%85%E9%98%9F%E5%88%97.png)

> AQS里面的CLH队列是CLH同步锁的一种变形。其主要从两方面进行了改造：节点的结构与节点等待机制。在结构上引入了头结点和尾节点，他们分别指向队列的头和尾，尝试获取锁、入队列、释放锁等实现都与头尾节点相关，并且每个节点都引入前驱节点和后后续节点的引用；在等待机制上由原来的自旋改成阻塞唤醒。其结构如下


如图中所示，该类用一个变量`volatile int state`来表示这个被争用的资源，队头和队尾由两个变量来记录：
```java
private transient volatile Node head;
private transient volatile Node tail;
```

而其中数据结构`Node`实际上是对`Thread`的一个包装，其定义如下：
```java
static final class Node {
    /** Marker to indicate a node is waiting in shared mode */
    static final Node SHARED = new Node();
    /** Marker to indicate a node is waiting in exclusive mode */
    static final Node EXCLUSIVE = null;

    /** waitStatus value to indicate thread has cancelled */
    /** 因为超时或者中断，结点会被设置为取消状态，被取消状态的结点不应该去竞争锁，只能保持取消状态不变，不能转换为其他状态。处于这种状态的结点会被踢出队列，被GC回收 */
    static final int CANCELLED =  1;
    /** waitStatus value to indicate successor's thread needs unparking */
    /** 表示这个结点的继任结点被阻塞了，到时需要通知它 */
    static final int SIGNAL    = -1;
    /** waitStatus value to indicate thread is waiting on condition */
    /** 表示这个结点在条件队列中，因为等待某个条件而被阻塞 */
    static final int CONDITION = -2;
    /**
     * waitStatus value to indicate the next acquireShared should
     * unconditionally propagate
     */
     /**使用在共享模式头结点有可能牌处于这种状态，表示锁的下一次获取可以无条件传播*/
    static final int PROPAGATE = -3;

    /*当前node对象的等待状态，注意该状态并不是描述当前对象而是描述下一个节点的状态，
     * 从而来决定是否唤醒下一个节点，该节点总共有四个取值：
     * a. CANCELLED = 1：因为超时或者中断，结点会被设置为取消状态，被取消状态的结点不应该去竞争锁，
     * 只能保持取消状态不变，不能转换为其他状态。处于这种状态的结点会被踢出队列，被GC回收；
     * b. SIGNAL = -1：表示这个结点的继任结点被阻塞了，到时需要通知它；
     * c. CONDITION = -2：表示这个结点在条件队列中，因为等待某个条件而被阻塞；
     * d. PROPAGATE = -3：使用在共享模式头结点有可能牌处于这种状态，表示锁的下一次获取可以无条件传播；
     * e. 0： None of the above，新结点会处于这种状态。
     *
     * 非负值标识节点不需要被通知（唤醒）。
    */
    volatile int waitStatus;

     //当前节点的上一个节点，如果是头节点那么值为null
    volatile Node prev;

    //当前节点的下一个节点
    volatile Node next;

    //与Node绑定的线程对象
    volatile Thread thread;

    //下一个等待条件（Condition）的节点，由于Condition是独占模式，因此这里有一个简单的队列来描述Condition上的线程节点。
    Node nextWaiter;

    /**
     * Returns true if node is waiting in shared mode.
     */
    final boolean isShared() {
        return nextWaiter == SHARED;
    }

    /**
     * Returns previous node, or throws NullPointerException if null.
     * Use when predecessor cannot be null.  The null check could
     * be elided, but is present to help the VM.
     *
     * @return the predecessor of this node
     */
    final Node predecessor() throws NullPointerException {
        Node p = prev;
        if (p == null)
            throw new NullPointerException();
        else
            return p;
    }

    Node() {    // Used to establish initial head or SHARED marker
    }

    Node(Thread thread, Node mode) {     // Used by addWaiter
        this.nextWaiter = mode;
        this.thread = thread;
    }

    Node(Thread thread, int waitStatus) { // Used by Condition
        this.waitStatus = waitStatus;
        this.thread = thread;
    }
}
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

## public abstract class AbstractQueuedSynchronizer extends AbstractOwnableSynchronizer implements java.io.Serializable
类声明，这是一个抽象类，继承的抽象类`AbstractOWnableSynchronizer`类也是JUC中的一个重要的类，从名字就可以看出，“拥有某个锁的同步器”，其实就是排他锁。类结构如下：

![AbstractOwnableSynchronizer](http://ovn0i3kdg.bkt.clouddn.com/AbstractOwnableSynchronizer.png)

## protected AbstractQueuedSynchronizer() { }
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

## public final void acquire(int arg){...}
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
//获取锁失败后，将其包装成节点
/**
* addWaiter:
* 1. 尝试将新节点以最快的方式设置为尾节点，如果CAS设置尾节点成功，返回附加着当前线程的节点。
* 2. 如果CAS操作失败，则调用enq方法，循环入队直到成功。
*/
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
//循环插入队尾直到CAS成功。
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
3. acquireQueued(Node, int)
OK，通过`tryAcquire()`和`addWaiter()`，该线程获取资源失败，已经被放入等待队列尾部了，但是节点插入队尾后不会直接挂起，因为可能在插入的时候占有锁的线程已经运行结束了，所以会通过自旋进行锁的竞争。


那么下一步的工作是：进入等待状态休息，直到其他线程彻底释放资源后唤醒自己，自己再拿到资源，然后就可以去干自己想干的事了。没错，就是这样！是不是跟医院排队拿号有点相似~~`acquireQueued()`就是干这件事：在等待队列中排队拿号（中间没其它事干可以休息），直到拿到号后再返回。这个函数非常关键，还是上源码吧：
```java
//自旋获取锁，直至异常退出或获取锁成功，但是并不是busy acquire，因为当获取失败后会被挂起，由前驱节点释放锁时将其唤醒。
//同时由于唤醒的时候可能有其他线程竞争，所以还需要进行尝试获取锁，体现的非公平锁的精髓。
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;//标记是否成功拿到资源
    try {
        boolean interrupted = false;//标记等待过程中是否被中断过

        //又是一个“自旋”！
        for (;;) {
            final Node p = node.predecessor();//拿到前驱
            //如果前驱是head，即该结点已成老二，那么便有资格去尝试获取资源（可能是老大释放完资源唤醒自己的，当然也可能被interrupt了）。
            if (p == head && tryAcquire(arg)) {////如果node的前驱节点是head节点，尝试获取锁，如果获取锁成功，说明head节点已经释放锁了，将node设为head开始运行。
               setHead(node);
                setHead(node);//拿到资源后，将head指向该结点。所以head所指的标杆结点，就是当前获取到资源的那个结点或null。
                p.next = null; // setHead中node.prev已置为null，此处再将head.next置为null，就是为了方便GC回收以前的head结点。也就意味着之前拿完资源的结点出队了！
                failed = false;
                return interrupted;//返回等待过程中是否被中断过
            }

            //如果自己可以休息了，就进入waiting状态，直到被unpark()
            if (shouldParkAfterFailedAcquire(p, node) && //判断当前node在获取锁失败后是否可以挂起，通过pred的状态判断
                parkAndCheckInterrupt())////挂起线程，等待node的前驱节点唤醒。
                interrupted = true;//如果等待过程中被中断过，哪怕只有那么一次，就将interrupted标记为true
        }
    } finally {//中间出现异常，导致获取锁失败，取消当前锁的自旋尝试获取锁
        if (failed)
            cancelAcquire(node);
    }
}
```
其中`shouldParkAfterFailedAcquire`这个方法主要用于检查状态，看看自己是否真的可以去休息了，即进入waiting状态，源码如下：
```java
//更新前驱节点的状态，是否可以挂起当前获取锁失败的节点。
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    int ws = pred.waitStatus;//拿到前驱的状态
    if (ws == Node.SIGNAL)
    //pred节点的状态为signal，说明node无法获取锁，可以安全挂起。
        return true;
    if (ws > 0) {
        /*
         * 如果前驱放弃了，那就一直往前找，直到找到最近一个正常等待的状态，并排在它的后边。
         * 注意：那些放弃的结点，由于被自己“加塞”到它们前边，它们相当于形成一个无引用链，稍后就会被保安大叔赶走了(GC回收)！
         */
        do {
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0);
        pred.next = node;
    } else {
      compareAndSetWaitStatus(pred, ws, Node.SIGNAL);//将前驱节点的状态设置为SIGNAL，代表node需要被运行。
    //更新可能失败，所以也不能够直接返回true。
    }
    return false;
}
```
整个流程中，如果前驱结点的状态不是`SIGNAL`，那么自己就不能安心去休息，需要去找个安心的休息点，同时可以再尝试下看有没有机会轮到自己拿号。
另外`parkAndCheckInterrupt()`方法的意思是如果线程找好安全休息点后，那就可以安心去休息了。此方法就是让线程去休息，真正进入等待状态。源码如下：
```java
//挂起线程
private final boolean parkAndCheckInterrupt() {
     LockSupport.park(this);//调用park()使线程进入waiting状态
     return Thread.interrupted();//如果被唤醒，查看自己是不是被中断的。
}
```
`park()`会让当前线程进入`waiting`状态。在此状态下，有两种途径可以唤醒该线程：1）被unpark()；2）被interrupt()。

  现在重新回到`acquireQueued()`方法，总结下该函数的具体流程：
  1. 结点进入队尾后，检查状态，找到安全休息点；
  2. 调用`park()`进入`waiting`状态，等待`unpark()`或`interrupt()`唤醒自己；
  3. 被唤醒后，看自己是不是有资格能拿到号。如果拿到，head指向当前结点，并返回从入队到拿到号的整个过程中是否被中断过；如果没拿到，继续流程1。

现在再重新回到`acquire()`方法，总结下流程：
1. 调用自定义同步器的`tryAcquire()`尝试直接去获取资源，如果成功则直接返回；
2. 没成功，则`addWaiter()`将该线程加入等待队列的尾部，并标记为独占模式；
3. `acquireQueued()`使线程在等待队列中休息，有机会时（轮到自己，会被`unpark()`）会去尝试获取资源。获取到资源后才返回。如果在整个等待过程中被中断过，则返回true，否则返回false。
4. 如果线程在等待过程中被中断过，它是不响应的。只是获取资源后才再进行自我中断`selfInterrupt()`，将中断补上。

这个方法非常重要，用下面这张图可以总结一下：

![acquire](http://ovn0i3kdg.bkt.clouddn.com/AQS%E4%B8%ADacquire.png)


## public final boolean release(int arg){...}
`release`是`acquire`的反操作，此方法是**独占模式**下线程释放共享资源的顶层入口。它会释放指定量的资源，如果彻底释放了（即`state=0`）,它会唤醒等待队列里的其他线程来获取资源。这也正是`unlock()`的语义，当然不仅仅只限于`unlock()`。下面是release()的源码：
```java
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;//找到头结点
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);//唤醒等待队列里的下一个线程
        return true;
    }
    return false;
}
```
逻辑并不复杂。它调用tryRelease()来释放资源。有一点需要注意的是，**它是根据`tryRelease()`的返回值来判断该线程是否已经完成释放掉资源了！所以自定义同步器在设计`tryRelease()`的时候要明确这一点！！** 下面同样进行分步讲解：
1. tryRelease(int)
此方法尝试去释放指定量的资源。下面是tryRelease()的源码：
```java
protected boolean tryRelease(int arg) {
    throw new UnsupportedOperationException();
}
```
和`tryAcquire`一样，`tryRelease`同样需要自定义的同步器去实现，正常来说，`tryRelease()`都会成功的，因为这是独占模式，该线程来释放资源，那么它肯定已经拿到独占资源了，直接减掉相应量的资源即可(`state-=arg`)，也不需要考虑线程安全的问题。但要注意它的返回值，上面已经提到了，** `release()`是根据`tryRelease()`的返回值来判断该线程是否已经完成释放掉资源了！** 所以自义定同步器在实现时，如果已经彻底释放资源(state=0)，要返回true，否则返回false。

2. unparkSuccessor(Node)
此方法用于唤醒等待队列中下一个线程。下面是源码：
```java
private void unparkSuccessor(Node node) {
  /*
   * If status is negative (i.e., possibly needing signal) try
   * to clear in anticipation of signalling.  It is OK if this
   * fails or if status is changed by waiting thread.
   *
   * 如果waitStatus小于0，那么将下一个节点设置成头节点
   */
    int ws = node.waitStatus;
    if (ws < 0)//置零当前线程所在的结点状态，允许失败。
        compareAndSetWaitStatus(node, ws, 0);

  /*
   * Thread to unpark is held in successor, which is normally
   * just the next node.  But if cancelled or apparently null,
   * traverse backwards from tail to find the actual
   * non-cancelled successor.
   *
   * 如果waitStatus大于0，或者下一个节点为null，那么从后往前找
   * 之前与同学讨论的时候不是很明白为什么要从后往前找，现在看了下doc瞬间明白了：
   * 下一个节点有可能因为任务被取消了，节点有可能变为null
   */
    Node s = node.next;//找到下一个需要唤醒的结点s
    if (s == null || s.waitStatus > 0) {//如果为空或已取消
        s = null;
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)//从这里可以看出，<=0的结点，都是还有效的结点。
                s = t;
    }
    if (s != null)
        LockSupport.unpark(s.thread);//唤醒
}
```
这个函数并不复杂。一句话概括：**用`unpark()`唤醒等待队列中最前边的那个未放弃线程**，这里我们也用s来表示吧。此时，再和`acquireQueued()`联系起来，s被唤醒后，进入`if (p == head && tryAcquire(arg))`的判断（即使`p!=head`也没关系，它会再进入`shouldParkAfterFailedAcquire()`寻找一个安全点。这里既然s已经是等待队列中最前边的那个未放弃线程了，那么通过`shouldParkAfterFailedAcquire()`的调整，s也必然会跑到`head`的`next`结点，下一次自旋`p==head`就成立啦），然后s把自己设置成head标杆结点，表示自己已经获取到资源了，`acquire()`也返回了！！And then, DO what you WANT!

总结一下：`release()`是独占模式下线程释放共享资源的顶层入口。它会释放指定量的资源，如果彻底释放了（即`state=0`）,它会唤醒等待队列里的其他线程来获取资源。


## public final void acquireShared(int arg) {...}
此方法是共享模式下线程获取共享资源的顶层入口。它会获取指定量的资源，获取成功则直接返回，获取失败则进入等待队列，直到获取到资源为止，整个过程忽略中断。下面是`acquireShared()`的源码：
```java
public final void acquireShared(int arg) {
    if (tryAcquireShared(arg) < 0)
        doAcquireShared(arg);
}
```
这里`tryAcquireShared()`依然需要自定义同步器去实现。但是`AQS`已经把其返回值的语义定义好了：负值代表获取失败；0代表获取成功，但没有剩余资源；正数表示获取成功，还有剩余资源，其他线程还可以去获取。所以这里`acquireShared()`的流程就是：
1. `tryAcquireShared()`尝试获取资源，成功则直接返回；
2. 失败则通过`doAcquireShared()`进入等待队列，直到获取到资源为止才返回。

下面进行分步讲解：
1. doAcquireShared(int)
此方法用于将当前线程加入等待队列尾部休息，直到其他线程释放资源唤醒自己，自己成功拿到相应量的资源后才返回。下面是`doAcquireShared()`的源码：
```java
private void doAcquireShared(int arg) {
    final Node node = addWaiter(Node.SHARED);//加入队列尾部
    boolean failed = true;//是否成功标志
    try {
        boolean interrupted = false;//等待过程中是否被中断过的标志
        for (;;) {
            final Node p = node.predecessor();//前驱
            if (p == head) {//如果到head的下一个，因为head是拿到资源的线程，此时node被唤醒，很可能是head用完资源来唤醒自己的
                int r = tryAcquireShared(arg);//尝试获取资源
                if (r >= 0) {//成功
                    setHeadAndPropagate(node, r);//将head指向自己，还有剩余资源可以再唤醒之后的线程
                    p.next = null; // help GC
                    if (interrupted)//如果等待过程中被打断过，此时将中断补上。
                        selfInterrupt();
                    failed = false;
                    return;
                }
            }      
            //判断状态，寻找安全点，进入waiting状态，等着被unpark()或interrupt()
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```
有木有觉得跟`acquireQueued()`很相似？对，其实流程并没有太大区别。只不过这里将补中断的`selfInterrupt()`放到`doAcquireShared()`里了，而独占模式是放到`acquireQueued()`之外，其实都一样，不知道Doug Lea是怎么想的。
跟独占模式比，还有一点需要注意的是，这里只有线程是`head.next`时（“老二”），才会去尝试获取资源，有剩余的话还会唤醒之后的队友。那么问题就来了，假如老大用完后释放了5个资源，而老二需要6个，老三需要1个，老四需要2个。老大先唤醒老二，老二一看资源不够，他是把资源让给老三呢，还是不让？答案是否定的！老二会继续park()等待其他线程释放资源，也更不会去唤醒老三和老四了。独占模式，同一时刻只有一个线程去执行，这样做未尝不可；但共享模式下，多个线程是可以同时执行的，现在因为老二的资源需求量大，而把后面量小的老三和老四也都卡住了。当然，这并不是问题，只是AQS保证严格按照入队顺序唤醒罢了（保证公平，但降低了并发）。
2. setHeadAndPropagate(Node, int)
```java
private void setHeadAndPropagate(Node node, int propagate) {
    Node h = head;
    setHead(node);//head指向自己
     //如果还有剩余量，继续唤醒下一个邻居线程
    if (propagate > 0 || h == null || h.waitStatus < 0) {
        Node s = node.next;
        if (s == null || s.isShared())
            doReleaseShared();
    }
}
```
此方法在`setHead()`的基础上多了一步，就是自己苏醒的同时，如果条件符合（比如还有剩余资源），还会去唤醒后继结点，毕竟是共享模式！`doReleaseShared()`我们留着下一小节的`releaseShared()`里来讲。


现在总结一下`acquireShared()`的流程：
1. `tryAcquireShared()`尝试获取资源，成功则直接返回；
2. 失败则通过`doAcquireShared()`进入等待队列`park()`，直到被`unpark()`/`interrupt()`并成功获取到资源才返回。整个等待过程也是忽略中断的。

其实跟`acquire()`的流程大同小异，只不过多了个自己拿到资源后，还会去唤醒后继队友的操作（这才是共享嘛）。

## public final boolean releaseShared(int arg) {...}
此方法是共享模式下线程释放共享资源的顶层入口。它会释放指定量的资源，如果成功释放且允许唤醒等待线程，它会唤醒等待队列里的其他线程来获取资源。下面是`releaseShared()`的源码：
```java
public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {//尝试释放资源
        doReleaseShared();//唤醒后继结点
        return true;
    }
    return false;
}
```
此方法的流程也比较简单，一句话：**释放掉资源后，唤醒后继**。跟独占模式下的`release()`相似，但有一点稍微需要注意：独占模式下的`tryRelease()`在完全释放掉资源（`state=0`）后，才会返回true去唤醒其他线程，这主要是基于独占下可重入的考量；而共享模式下的`releaseShared()`则没有这种要求，共享模式实质就是控制一定量的线程并发执行，那么拥有资源的线程在释放掉部分资源时就可以唤醒后继等待结点。例如，资源总量是13，A（5）和B（7）分别获取到资源并发运行，C（4）来时只剩1个资源就需要等待。A在运行过程中释放掉2个资源量，然后`tryReleaseShared(2)`返回true唤醒C，C一看只有3个仍不够继续等待；随后B又释放2个，`tryReleaseShared(2)`返回true唤醒C，C一看有5个够自己用了，然后C就可以跟A和B一起运行。而`ReentrantReadWriteLock`读锁的`tryReleaseShared()`只有在完全释放掉资源（`state=0`）才返回true，所以自定义同步器可以根据需要决定`tryReleaseShared()`的返回值。

其中`doReleaseShared()`的方法定义如下：
```java
private void doReleaseShared() {
    for (;;) {
        Node h = head;
        if (h != null && h != tail) {
            int ws = h.waitStatus;
            if (ws == Node.SIGNAL) {
                if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                    continue;
                unparkSuccessor(h);//唤醒后继
            }
            else if (ws == 0 &&
                     !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                continue;
        }
        if (h == head)// head发生变化
            break;
    }
}
```
此方法主要用于唤醒后继。


参考
* [java锁的种类以及辨析（一）：自旋锁](http://ifeve.com/java_lock_see1/)
* [Java并发之AQS详解](https://www.cnblogs.com/waterystone/p/4920797.html)
* [关于AQS的一点总结](http://blog.csdn.net/mian_CSDN/article/details/61913200)
* [AbstractQueuedSynchronizer-源码走读](https://zhuanlan.zhihu.com/p/28843912)
* [40个Java多线程问题总结](https://zhuanlan.zhihu.com/p/26441926)
