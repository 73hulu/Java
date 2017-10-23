# Lock

这个接口及相关实现类是在JDK1.5中加入的。先看下接口：

![Lock](http://ovn0i3kdg.bkt.clouddn.com/Lock.png)



###  void lock();
获取锁。
如果该锁没有被另一个线程保持，则获取该锁并立即返回，将锁的保持计数设置为 1。
如果当前线程已经保持该锁，则将保持计数加 1，并且该方法立即返回。
如果该锁被另一个线程持有，则该线程不可被调度（disabled for thread scheduling purposes）（即阻塞状态，CPU不会给该线程分配时间片）直到该线程获取到该锁，并且在获取到锁后，将保持计数设置为1



###  void lockInterruptibly() throws InterruptedException;

1. 如果当前线程未被中断，则获取锁。
2. 如果该锁没有被另一个线程保持，则获取该锁并立即返回，将锁的保持计数器置为1。
3. 如果当前线程已经保持该锁，则将保持计数器加1，并将该方法返回。
4. 如果锁被另一个线程保持，则处于线程调度的目的，禁用当前线程，并且在发生以下两种情况之一以前，该线程一直处于休眠状态：
  1. 锁由当前线程获得；或者
  1. 其他某个线程中断当前线程。
5. 如果当前线程获得该锁，则将锁保持器置为1.
如果当前线程：
  1. 在进入方法时已经设置了该线程的终端状态；或者
  1. 在等待获取锁的同时被终端。
则抛出`InterruptedException`，并且清除当前线程的已终端状态。
6. 在此实现中，因为此方法是一个显式中断点，所以要优先考虑响应中断，而不是响应普通锁的获取或冲入获取。

抛出：`InterruptedException` 如果单钱线程已经中断


### boolean tryLock();

仅在调用时锁未被另一个线程保持的情况下，才获取该锁。

1. 如果该锁没有被另一个线程保持，并且立即返回 true 值，则将锁的保持计数设置为 1。
即使已将此锁设置为使用公平排序策略，但是调用 tryLock() 仍将 立即获取锁（如果有可用的），
而不管其他线程当前是否正在等待该锁。在某些情况下，此“闯入”行为可能很有用，即使它会打破公
平性也如此。如果希望遵守此锁的公平设置，则使用`tryLock(0, TimeUnit.SECONDS)`
，它几乎是等效的（也检测中断）。

2. 如果当前线程已经保持此锁，则将保持计数加 1，该方法将返回 true。

3. 如果锁被另一个线程保持，则此方法将立即返回 false 值。

返回：如果锁是自由的并且被当前线程获取，或者当前线程已经保持该锁，则返回 `true`；否则返回`false`。

一个典型的使用场景是：
```java
Lock lock = ...;
if (lock.tryLock()) {
  try {
    // manipulate protected state
  } finally {
    lock.unlock();
  }
} else {
  // perform alternative actions
}
```


### boolean tryLock(long time, TimeUnit unit) throws InterruptedException;

---
和无参数的`tryLock`方法功能相同。但是允许在指定的时间内尝试，拿不到锁的情况下就等待一段时间，超出时间后再返回结果，比较聪明的做法。`tryLock()`和`tryLock(0, TimeUnit.SECONDS)`是等效的。

上面讲了三种`Lock`接口中获取锁的方式：`lock()`、`lockInterruptibly`和`tryLock`，有什么不同？

* `lock()`：调用后一直阻塞到获得锁，拿不到锁誓不罢休。忽略`interrupt`。
* `lockInterruptibly`：调用后一直阻塞到获得锁，但是会响应中断，这个方法优先考虑响应中断，而不是响应锁的普通获取或重入获取。
* `tryLock`：立即返回，获得锁就返回true，没有就返回false。

关于线程中断有很多要讲的，这里略过，讲下线程的打扰机制。每个线程都有一个打扰标志，这里分为两种情况：
1. **线程在sleep或wait或join**，此时如果别的进程调用此进程的`interrupt()`方法，此线程会被唤醒并要求处理`InterruptedException`。具体见`Thread`API。
2. **如果此线程在运行中**，则不会收到提醒。但是线程的“打扰标志”会被设置，可以通过`isInterrupted()`查看并作出处理。

`lockInterruptibly()`和上面的第一种情况是一样的，线程在请求`lock`并且被阻塞的时候，如果被`interrupt`，则“此线程会被要求处理`InterruptedException`查看并作出处理”

### void unlock();
释放锁

### Condition newCondition();
返回绑定到此`Lock`实例的新`Condition`实例。
在等待条件之前，锁必须由当前线程保持。 对`Condition.await（）`的调用将在等待之前将原子地释放锁，并在等待返回之前重新获取锁。



参考：
* [Java中Lock，tryLock，lockInterruptibly有什么区别？](https://www.zhihu.com/question/36771163)
* [Java中的锁-Lock接口解析](http://blog.csdn.net/canot/article/details/52050633)
