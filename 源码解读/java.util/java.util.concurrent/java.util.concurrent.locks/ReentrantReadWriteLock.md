# ReentrantReadWriteLock

JUC中各种锁（比如`ReentrantLock`），基本上都是排他锁，即同一时刻只能有一个线程进行访问，对资源的占有采取的是独占策略。

而读写锁允许**同一时刻有多个读线程访问，但是在写线程访问时，其他读线程和写线程都会被阻塞。** 所以说，读写锁同时采取了独占策略和分享策略。

在没有读写锁支持的（Java 5 之前）时候，如果需要完成上述工作就要使用Java的等待通知机制，就是当写操作开始时，所有晚于写操作的读操作均会进入等待状态，只有写操作完成并进行通知之后，所有等待的读操作才能继续执行（写操作之间依靠`synchronized`关键字进行同步），这样做的目的是使读操作都能读取到正确的数据，而不会出现脏读。

改用读写锁实现上述功能，只需要在读操作时获取读锁，而写操作时获取写锁即可，当写锁被获取到时，后续（非当前写操作线程）的读写操作都会被阻塞，写锁释放之后，所有操作继续执行，编程方式相对于使用等待通知机制的实现方式而言，变得简单明了。

读写锁维护了一对锁，读锁和写锁，通过锁的分离，使得并发性比一般的排他锁有了很大的提高。`ReentrantReadWriteLock`就是这样的Java提供的可重入的读写锁。其结果如下：

![ReentrantReadWriteLock](http://ovn0i3kdg.bkt.clouddn.com/ReentrantReadWriteLock.png?imageView/2/w/400 )

`ReentrantReadWriteLock`支持如下性能：
1. 公平性：支持非公平（默认）和公平性锁获取方式，前者吞吐量大于后者
1. 锁重入性：其内部的`WriteLock`可以获取`ReadLock`，但是反过来`ReadLock`想要获得`WriteLock`则永远都不要想。
2. **锁降级**： `WriteLock`可以降级为`ReadLock`，顺序是：先获得`WriteLock`再获得`ReadLock`，然后释放`WriteLock`，这时候线程将保持`Readlock`的持有。反过来`ReadLock`想要升级为`WriteLock`则不可能，为什么？参上一条。
3. ` ReadLock`可以被多个线程持有并且在作用时排斥任何的`WriteLock`，而`WriteLock`则是完全的互斥。这一特性最为重要，因为对于**高读取频率而相对较低写入**的数据结构，使用此类锁同步机制则可以提高并发量。
4. 不管是`ReadLock`还是`WriteLock`都支持`Interrupt`，语义与`ReentrantLock`一致。
5. `WriteLock`支持`Condition`并且与`ReentrantLock`语义一致，而`ReadLock`则不能使用`Condition`，否则抛出`UnsupportedOperationException`异常。

我们首先了解一下可重入读写锁的基本工作过程。这个过程在博文 http://blog.csdn.net/qyp199312/article/details/70598480 中图文并茂的展示了公平可重入读写锁和非公平可重入读写锁的工作过程，生动形象，可以参考。


我们还是从源码角度看看这些性能是如何实现的。

## public class ReentrantReadWriteLock implements ReadWriteLock, java.io.Serializable
类声明，可见`ReentrantReadWriteLock`实现`ReadWriteLock`接口，而该接口的结构如下：

![ReadWriteLock](http://ovn0i3kdg.bkt.clouddn.com/ReadWriteLock.png)

接口定义了两个锁，一个读锁，一个写锁。注意，`ReentrantReadWriteLock`并没有继承`Lock`类，也没有继承AQS。回想一下`ReentrantLock`也没有继承`Lock`类，而是利用内部类实现了AQS。这一点和`ReentrantLock`一样。


## 构造方法
`ReentrantReadWriteLock`提供了两个构造方法。
```java
public ReentrantReadWriteLock() {
   this(false);
}

public ReentrantReadWriteLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
    readerLock = new ReadLock(this);
    writerLock = new WriteLock(this);
}
```
区别在于前者默认构造的是非公平锁，而后者可指定公平性。和`ReentrantLock`一样，`ReentrantLock`中定义了一个抽象内部类`Sync`，该类继承了AQS抽象类，并且定义了`Sync`的两个实现类：`NonfairSync`和`FairSync`，两者重新定义了`tryAcquire`和`tryRelease`方法。

`Sync`内部抽象类中重要的是`tryAcquire`方法、`tryRelease`方法、`tryReleaseShare`方法、`tryReadLock`方法和`tryWriteLock`方法。我们逐个来看一下这些方法的实现：
#### tryAcquire
```Java
protected final boolean tryAcquire(int acquires) {
    /*
     * Walkthrough:
     * 1. If read count nonzero or write count nonzero
     *    and owner is a different thread, fail.
     * 2. If count would saturate, fail. (This can only
     *    happen if count is already nonzero.)
     * 3. Otherwise, this thread is eligible for lock if
     *    it is either a reentrant acquire or
     *    queue policy allows it. If so, update state
     *    and set owner.
     */
    Thread current = Thread.currentThread();
    int c = getState();
    int w = exclusiveCount(c);
    if (c != 0) {
        // (Note: if c != 0 and w == 0 then shared count != 0)
        if (w == 0 || current != getExclusiveOwnerThread())
            return false;
        if (w + exclusiveCount(acquires) > MAX_COUNT)
            throw new Error("Maximum lock count exceeded");
        // Reentrant acquire
        setState(c + acquires);
        return true;
    }
    if (writerShouldBlock() ||
        !compareAndSetState(c, c + acquires))
        return false;
    setExclusiveOwnerThread(current);
    return true;
}
```
代码开头的注解解释了这个方法的含义：
1. 如果当前写锁非0和读锁非0，并且其拥有者并不是当前的线程，则获取失败。
2. 如果当前锁已经饱和，即已经达到上限了，则获取失败。
3. 否则，如果该线程是可重入获取或队列策略允许，则该线程可以锁定。 如果是这样，更新状态并设置所有者。

#### tryRelease
```Java
/*
 * Note that tryRelease and tryAcquire can be called by
 * Conditions. So it is possible that their arguments contain
 * both read and write holds that are all released during a
 * condition wait and re-established in tryAcquire.
 */

protected final boolean tryRelease(int releases) {
    if (!isHeldExclusively())
        throw new IllegalMonitorStateException();
    int nextc = getState() - releases;
    boolean free = exclusiveCount(nextc) == 0;
    if (free)
        setExclusiveOwnerThread(null);
    setState(nextc);
    return free;
}
```
`tryRelease`方法用来释放锁。在注释中写到`tryRelease`和`tryAcquire`可以被Condition调用。


#### tryReadLock
```Java
/**
 * Performs tryLock for read, enabling barging in both modes.
 * This is identical in effect to tryAcquireShared except for
 * lack of calls to readerShouldBlock.
 */
final boolean tryReadLock() {
    Thread current = Thread.currentThread();
    for (;;) {
        int c = getState();
        if (exclusiveCount(c) != 0 &&
            getExclusiveOwnerThread() != current)
            return false;
        int r = sharedCount(c);
        if (r == MAX_COUNT)
            throw new Error("Maximum lock count exceeded");
        if (compareAndSetState(c, c + SHARED_UNIT)) {
            if (r == 0) {
                firstReader = current;
                firstReaderHoldCount = 1;
            } else if (firstReader == current) {
                firstReaderHoldCount++;
            } else {
                HoldCounter rh = cachedHoldCounter;
                if (rh == null || rh.tid != getThreadId(current))
                    cachedHoldCounter = rh = readHolds.get();
                else if (rh.count == 0)
                    readHolds.set(rh);
                rh.count++;
            }
            return true;
        }
    }
}
```
尝试获取读锁。

#### tryWriteLock
```Java
/**
 * Performs tryLock for write, enabling barging in both modes.
 * This is identical in effect to tryAcquire except for lack
 * of calls to writerShouldBlock.
 */
final boolean tryWriteLock() {
    Thread current = Thread.currentThread();
    int c = getState();
    if (c != 0) {
        int w = exclusiveCount(c);
        if (w == 0 || current != getExclusiveOwnerThread())
            return false;
        if (w == MAX_COUNT)
            throw new Error("Maximum lock count exceeded");
    }
    if (!compareAndSetState(c, c + 1))
        return false;
    setExclusiveOwnerThread(current);
    return true;
}
```
尝试获取写锁。这个很好办了，只有当前没有读锁和写锁都没有被获取，获取写锁是自己，这时候自己才能获取写锁。


另外，由于继承`ReadWriteLock`，所以`ReentrantReadWriteLock`必须提供获取读写锁的方法。该类定义分别定义了读锁和写锁两个静态内部类。这两个类都实现了`Lock`接口。

从源码中可以看到，读写锁采取的默认策略也是非公平。

## WriteLock和ReadLock
`ReentrantReadWriteLock`中定义了两个内部类：`WriteLock`和 `ReadLock`。这两个类都实现Lock接口，即实现了`lock`、`tryLock`和`lockInterruptibly`等方法，而这些方法中，真正的实现仍旧是`Syn`中的各种方法。

```Java
/**
* The lock returned by method {@link ReentrantReadWriteLock#readLock}.
*/
public static class ReadLock implements Lock, java.io.Serializable {
   private static final long serialVersionUID = -5992448646407690164L;
   private final Sync sync;

   /**
    * Constructor for use by subclasses
    *
    * @param lock the outer lock object
    * @throws NullPointerException if the lock is null
    */
   protected ReadLock(ReentrantReadWriteLock lock) {
       sync = lock.sync;
   }

   /**
    * Acquires the read lock.
    *
    * <p>Acquires the read lock if the write lock is not held by
    * another thread and returns immediately.
    *
    * <p>If the write lock is held by another thread then
    * the current thread becomes disabled for thread scheduling
    * purposes and lies dormant until the read lock has been acquired.
    */
   public void lock() {
       sync.acquireShared(1);
   }

   /**
    * Acquires the read lock unless the current thread is
    * {@linkplain Thread#interrupt interrupted}.
    *
    * <p>Acquires the read lock if the write lock is not held
    * by another thread and returns immediately.
    *
    * <p>If the write lock is held by another thread then the
    * current thread becomes disabled for thread scheduling
    * purposes and lies dormant until one of two things happens:
    *
    * <ul>
    *
    * <li>The read lock is acquired by the current thread; or
    *
    * <li>Some other thread {@linkplain Thread#interrupt interrupts}
    * the current thread.
    *
    * </ul>
    *
    * <p>If the current thread:
    *
    * <ul>
    *
    * <li>has its interrupted status set on entry to this method; or
    *
    * <li>is {@linkplain Thread#interrupt interrupted} while
    * acquiring the read lock,
    *
    * </ul>
    *
    * then {@link InterruptedException} is thrown and the current
    * thread's interrupted status is cleared.
    *
    * <p>In this implementation, as this method is an explicit
    * interruption point, preference is given to responding to
    * the interrupt over normal or reentrant acquisition of the
    * lock.
    *
    * @throws InterruptedException if the current thread is interrupted
    */
   public void lockInterruptibly() throws InterruptedException {
       sync.acquireSharedInterruptibly(1);
   }

   /**
    * Acquires the read lock only if the write lock is not held by
    * another thread at the time of invocation.
    *
    * <p>Acquires the read lock if the write lock is not held by
    * another thread and returns immediately with the value
    * {@code true}. Even when this lock has been set to use a
    * fair ordering policy, a call to {@code tryLock()}
    * <em>will</em> immediately acquire the read lock if it is
    * available, whether or not other threads are currently
    * waiting for the read lock.  This &quot;barging&quot; behavior
    * can be useful in certain circumstances, even though it
    * breaks fairness. If you want to honor the fairness setting
    * for this lock, then use {@link #tryLock(long, TimeUnit)
    * tryLock(0, TimeUnit.SECONDS) } which is almost equivalent
    * (it also detects interruption).
    *
    * <p>If the write lock is held by another thread then
    * this method will return immediately with the value
    * {@code false}.
    *
    * @return {@code true} if the read lock was acquired
    */
   public boolean tryLock() {
       return sync.tryReadLock();
   }

   /**
    * Acquires the read lock if the write lock is not held by
    * another thread within the given waiting time and the
    * current thread has not been {@linkplain Thread#interrupt
    * interrupted}.
    *
    * <p>Acquires the read lock if the write lock is not held by
    * another thread and returns immediately with the value
    * {@code true}. If this lock has been set to use a fair
    * ordering policy then an available lock <em>will not</em> be
    * acquired if any other threads are waiting for the
    * lock. This is in contrast to the {@link #tryLock()}
    * method. If you want a timed {@code tryLock} that does
    * permit barging on a fair lock then combine the timed and
    * un-timed forms together:
    *
    *  <pre> {@code
    * if (lock.tryLock() ||
    *     lock.tryLock(timeout, unit)) {
    *   ...
    * }}</pre>
    *
    * <p>If the write lock is held by another thread then the
    * current thread becomes disabled for thread scheduling
    * purposes and lies dormant until one of three things happens:
    *
    * <ul>
    *
    * <li>The read lock is acquired by the current thread; or
    *
    * <li>Some other thread {@linkplain Thread#interrupt interrupts}
    * the current thread; or
    *
    * <li>The specified waiting time elapses.
    *
    * </ul>
    *
    * <p>If the read lock is acquired then the value {@code true} is
    * returned.
    *
    * <p>If the current thread:
    *
    * <ul>
    *
    * <li>has its interrupted status set on entry to this method; or
    *
    * <li>is {@linkplain Thread#interrupt interrupted} while
    * acquiring the read lock,
    *
    * </ul> then {@link InterruptedException} is thrown and the
    * current thread's interrupted status is cleared.
    *
    * <p>If the specified waiting time elapses then the value
    * {@code false} is returned.  If the time is less than or
    * equal to zero, the method will not wait at all.
    *
    * <p>In this implementation, as this method is an explicit
    * interruption point, preference is given to responding to
    * the interrupt over normal or reentrant acquisition of the
    * lock, and over reporting the elapse of the waiting time.
    *
    * @param timeout the time to wait for the read lock
    * @param unit the time unit of the timeout argument
    * @return {@code true} if the read lock was acquired
    * @throws InterruptedException if the current thread is interrupted
    * @throws NullPointerException if the time unit is null
    */
   public boolean tryLock(long timeout, TimeUnit unit)
           throws InterruptedException {
       return sync.tryAcquireSharedNanos(1, unit.toNanos(timeout));
   }

   /**
    * Attempts to release this lock.
    *
    * <p>If the number of readers is now zero then the lock
    * is made available for write lock attempts.
    */
   public void unlock() {
       sync.releaseShared(1);
   }

   /**
    * Throws {@code UnsupportedOperationException} because
    * {@code ReadLocks} do not support conditions.
    *
    * @throws UnsupportedOperationException always
    */
   public Condition newCondition() {
       throw new UnsupportedOperationException();
   }

   /**
    * Returns a string identifying this lock, as well as its lock state.
    * The state, in brackets, includes the String {@code "Read locks ="}
    * followed by the number of held read locks.
    *
    * @return a string identifying this lock, as well as its lock state
    */
   public String toString() {
       int r = sync.getReadLockCount();
       return super.toString() +
           "[Read locks = " + r + "]";
   }
}
```


```Java
/**
 * The lock returned by method {@link ReentrantReadWriteLock#writeLock}.
 */
public static class WriteLock implements Lock, java.io.Serializable {
    private static final long serialVersionUID = -4992448646407690164L;
    private final Sync sync;

    /**
     * Constructor for use by subclasses
     *
     * @param lock the outer lock object
     * @throws NullPointerException if the lock is null
     */
    protected WriteLock(ReentrantReadWriteLock lock) {
        sync = lock.sync;
    }

    /**
     * Acquires the write lock.
     *
     * <p>Acquires the write lock if neither the read nor write lock
     * are held by another thread
     * and returns immediately, setting the write lock hold count to
     * one.
     *
     * <p>If the current thread already holds the write lock then the
     * hold count is incremented by one and the method returns
     * immediately.
     *
     * <p>If the lock is held by another thread then the current
     * thread becomes disabled for thread scheduling purposes and
     * lies dormant until the write lock has been acquired, at which
     * time the write lock hold count is set to one.
     */
    public void lock() {
        sync.acquire(1);
    }

    /**
     * Acquires the write lock unless the current thread is
     * {@linkplain Thread#interrupt interrupted}.
     *
     * <p>Acquires the write lock if neither the read nor write lock
     * are held by another thread
     * and returns immediately, setting the write lock hold count to
     * one.
     *
     * <p>If the current thread already holds this lock then the
     * hold count is incremented by one and the method returns
     * immediately.
     *
     * <p>If the lock is held by another thread then the current
     * thread becomes disabled for thread scheduling purposes and
     * lies dormant until one of two things happens:
     *
     * <ul>
     *
     * <li>The write lock is acquired by the current thread; or
     *
     * <li>Some other thread {@linkplain Thread#interrupt interrupts}
     * the current thread.
     *
     * </ul>
     *
     * <p>If the write lock is acquired by the current thread then the
     * lock hold count is set to one.
     *
     * <p>If the current thread:
     *
     * <ul>
     *
     * <li>has its interrupted status set on entry to this method;
     * or
     *
     * <li>is {@linkplain Thread#interrupt interrupted} while
     * acquiring the write lock,
     *
     * </ul>
     *
     * then {@link InterruptedException} is thrown and the current
     * thread's interrupted status is cleared.
     *
     * <p>In this implementation, as this method is an explicit
     * interruption point, preference is given to responding to
     * the interrupt over normal or reentrant acquisition of the
     * lock.
     *
     * @throws InterruptedException if the current thread is interrupted
     */
    public void lockInterruptibly() throws InterruptedException {
        sync.acquireInterruptibly(1);
    }

    /**
     * Acquires the write lock only if it is not held by another thread
     * at the time of invocation.
     *
     * <p>Acquires the write lock if neither the read nor write lock
     * are held by another thread
     * and returns immediately with the value {@code true},
     * setting the write lock hold count to one. Even when this lock has
     * been set to use a fair ordering policy, a call to
     * {@code tryLock()} <em>will</em> immediately acquire the
     * lock if it is available, whether or not other threads are
     * currently waiting for the write lock.  This &quot;barging&quot;
     * behavior can be useful in certain circumstances, even
     * though it breaks fairness. If you want to honor the
     * fairness setting for this lock, then use {@link
     * #tryLock(long, TimeUnit) tryLock(0, TimeUnit.SECONDS) }
     * which is almost equivalent (it also detects interruption).
     *
     * <p>If the current thread already holds this lock then the
     * hold count is incremented by one and the method returns
     * {@code true}.
     *
     * <p>If the lock is held by another thread then this method
     * will return immediately with the value {@code false}.
     *
     * @return {@code true} if the lock was free and was acquired
     * by the current thread, or the write lock was already held
     * by the current thread; and {@code false} otherwise.
     */
    public boolean tryLock( ) {
        return sync.tryWriteLock();
    }

    /**
     * Acquires the write lock if it is not held by another thread
     * within the given waiting time and the current thread has
     * not been {@linkplain Thread#interrupt interrupted}.
     *
     * <p>Acquires the write lock if neither the read nor write lock
     * are held by another thread
     * and returns immediately with the value {@code true},
     * setting the write lock hold count to one. If this lock has been
     * set to use a fair ordering policy then an available lock
     * <em>will not</em> be acquired if any other threads are
     * waiting for the write lock. This is in contrast to the {@link
     * #tryLock()} method. If you want a timed {@code tryLock}
     * that does permit barging on a fair lock then combine the
     * timed and un-timed forms together:
     *
     *  <pre> {@code
     * if (lock.tryLock() ||
     *     lock.tryLock(timeout, unit)) {
     *   ...
     * }}</pre>
     *
     * <p>If the current thread already holds this lock then the
     * hold count is incremented by one and the method returns
     * {@code true}.
     *
     * <p>If the lock is held by another thread then the current
     * thread becomes disabled for thread scheduling purposes and
     * lies dormant until one of three things happens:
     *
     * <ul>
     *
     * <li>The write lock is acquired by the current thread; or
     *
     * <li>Some other thread {@linkplain Thread#interrupt interrupts}
     * the current thread; or
     *
     * <li>The specified waiting time elapses
     *
     * </ul>
     *
     * <p>If the write lock is acquired then the value {@code true} is
     * returned and the write lock hold count is set to one.
     *
     * <p>If the current thread:
     *
     * <ul>
     *
     * <li>has its interrupted status set on entry to this method;
     * or
     *
     * <li>is {@linkplain Thread#interrupt interrupted} while
     * acquiring the write lock,
     *
     * </ul>
     *
     * then {@link InterruptedException} is thrown and the current
     * thread's interrupted status is cleared.
     *
     * <p>If the specified waiting time elapses then the value
     * {@code false} is returned.  If the time is less than or
     * equal to zero, the method will not wait at all.
     *
     * <p>In this implementation, as this method is an explicit
     * interruption point, preference is given to responding to
     * the interrupt over normal or reentrant acquisition of the
     * lock, and over reporting the elapse of the waiting time.
     *
     * @param timeout the time to wait for the write lock
     * @param unit the time unit of the timeout argument
     *
     * @return {@code true} if the lock was free and was acquired
     * by the current thread, or the write lock was already held by the
     * current thread; and {@code false} if the waiting time
     * elapsed before the lock could be acquired.
     *
     * @throws InterruptedException if the current thread is interrupted
     * @throws NullPointerException if the time unit is null
     */
    public boolean tryLock(long timeout, TimeUnit unit)
            throws InterruptedException {
        return sync.tryAcquireNanos(1, unit.toNanos(timeout));
    }

    /**
     * Attempts to release this lock.
     *
     * <p>If the current thread is the holder of this lock then
     * the hold count is decremented. If the hold count is now
     * zero then the lock is released.  If the current thread is
     * not the holder of this lock then {@link
     * IllegalMonitorStateException} is thrown.
     *
     * @throws IllegalMonitorStateException if the current thread does not
     * hold this lock
     */
    public void unlock() {
        sync.release(1);
    }

    /**
     * Returns a {@link Condition} instance for use with this
     * {@link Lock} instance.
     * <p>The returned {@link Condition} instance supports the same
     * usages as do the {@link Object} monitor methods ({@link
     * Object#wait() wait}, {@link Object#notify notify}, and {@link
     * Object#notifyAll notifyAll}) when used with the built-in
     * monitor lock.
     *
     * <ul>
     *
     * <li>If this write lock is not held when any {@link
     * Condition} method is called then an {@link
     * IllegalMonitorStateException} is thrown.  (Read locks are
     * held independently of write locks, so are not checked or
     * affected. However it is essentially always an error to
     * invoke a condition waiting method when the current thread
     * has also acquired read locks, since other threads that
     * could unblock it will not be able to acquire the write
     * lock.)
     *
     * <li>When the condition {@linkplain Condition#await() waiting}
     * methods are called the write lock is released and, before
     * they return, the write lock is reacquired and the lock hold
     * count restored to what it was when the method was called.
     *
     * <li>If a thread is {@linkplain Thread#interrupt interrupted} while
     * waiting then the wait will terminate, an {@link
     * InterruptedException} will be thrown, and the thread's
     * interrupted status will be cleared.
     *
     * <li> Waiting threads are signalled in FIFO order.
     *
     * <li>The ordering of lock reacquisition for threads returning
     * from waiting methods is the same as for threads initially
     * acquiring the lock, which is in the default case not specified,
     * but for <em>fair</em> locks favors those threads that have been
     * waiting the longest.
     *
     * </ul>
     *
     * @return the Condition object
     */
    public Condition newCondition() {
        return sync.newCondition();
    }

    /**
     * Returns a string identifying this lock, as well as its lock
     * state.  The state, in brackets includes either the String
     * {@code "Unlocked"} or the String {@code "Locked by"}
     * followed by the {@linkplain Thread#getName name} of the owning thread.
     *
     * @return a string identifying this lock, as well as its lock state
     */
    public String toString() {
        Thread o = sync.getOwner();
        return super.toString() + ((o == null) ?
                                   "[Unlocked]" :
                                   "[Locked by thread " + o.getName() + "]");
    }

    /**
     * Queries if this write lock is held by the current thread.
     * Identical in effect to {@link
     * ReentrantReadWriteLock#isWriteLockedByCurrentThread}.
     *
     * @return {@code true} if the current thread holds this lock and
     *         {@code false} otherwise
     * @since 1.6
     */
    public boolean isHeldByCurrentThread() {
        return sync.isHeldExclusively();
    }

    /**
     * Queries the number of holds on this write lock by the current
     * thread.  A thread has a hold on a lock for each lock action
     * that is not matched by an unlock action.  Identical in effect
     * to {@link ReentrantReadWriteLock#getWriteHoldCount}.
     *
     * @return the number of holds on this lock by the current thread,
     *         or zero if this lock is not held by the current thread
     * @since 1.6
     */
    public int getHoldCount() {
        return sync.getWriteHoldCount();
    }
}
```

哎，读这个源码还是好吃力。

## 读写锁的用途
读写锁是为了提高并发数，特别适用于读多写少的场景。下面是一个读写锁的应用场景：
```Java
package com.thread;

import java.util.Random;
import java.util.concurrent.locks.ReentrantReadWriteLock;

public class ReadWriteLockTest {
    public static void main(String[] args) {
        final Queue3 q3 = new Queue3();
        for(int i=0;i<3;i++)
        {
            new Thread(){
                public void run(){
                    while(true){
                        q3.get();                        
                    }
                }

            }.start();
        }
        for(int i=0;i<3;i++)
        {        
            new Thread(){
                public void run(){
                    while(true){
                        q3.put(new Random().nextInt(10000));
                    }
                }            

            }.start();    
        }
    }
}

class Queue3{
    private Object data = null;//共享数据，只能有一个线程能写该数据，但可以有多个线程同时读该数据。
    private ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();
    public void get(){
        rwl.readLock().lock();//上读锁，其他线程只能读不能写
        System.out.println(Thread.currentThread().getName() + " be ready to read data!");
        try {
            Thread.sleep((long)(Math.random()*1000));
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + "have read data :" + data);        
        rwl.readLock().unlock(); //释放读锁，最好放在finnaly里面
    }

    public void put(Object data){

        rwl.writeLock().lock();//上写锁，不允许其他线程读也不允许写
        System.out.println(Thread.currentThread().getName() + " be ready to write data!");                    
        try {
            Thread.sleep((long)(Math.random()*1000));
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        this.data = data;        
        System.out.println(Thread.currentThread().getName() + " have write data: " + data);                    

        rwl.writeLock().unlock();//释放写锁    
    }
}
```
下面的代码使用读写锁实现缓存器：
```Java
package com.thread;

import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

public class CacheDemo {
    private Map<String, Object> map = new HashMap<String, Object>();//缓存器
    private ReadWriteLock rwl = new ReentrantReadWriteLock();
    public static void main(String[] args) {

    }
    public Object get(String id){
        Object value = null;
        rwl.readLock().lock();//首先开启读锁，从缓存中去取
        try{
            value = map.get(id);
            if(value == null){  //如果缓存中没有释放读锁，上写锁
                rwl.readLock().unlock();
                rwl.writeLock().lock();
                try{
                    if(value == null){
                        value = "aaa";  //此时可以去数据库中查找，这里简单的模拟一下
                    }
                }finally{
                    rwl.writeLock().unlock(); //释放写锁
                }
                rwl.readLock().lock(); //然后再上读锁
            }
        }finally{
            rwl.readLock().unlock(); //最后释放读锁
        }
        return value;
    }
}
```
> 参考[ReentrantReadWriteLock读写锁的使用](https://www.cnblogs.com/liuling/archive/2013/08/21/2013-8-21-03.html)


参考
* [轻松掌握java读写锁(ReentrantReadWriteLock)的实现原理](http://blog.csdn.net/yanyan19880509/article/details/52435135)
* [ReentrantReadWriteLock读写锁的使用](https://www.cnblogs.com/liuling/archive/2013/08/21/2013-8-21-03.html)
