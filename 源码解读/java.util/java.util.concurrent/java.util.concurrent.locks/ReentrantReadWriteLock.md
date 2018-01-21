# ReentrantReadWriteLock

JUC中各种锁（比如`ReentrantLock`），基本上都是排他锁，即同一时刻只能有一个线程进行访问。而读写锁允许**同一时刻有多个读线程访问，但是在写线程访问时，其他读线程和写线程都会被阻塞。**

在没有读写锁支持的（Java 5 之前）时候，如果需要完成上述工作就要使用Java的等待通知机制，就是当写操作开始时，所有晚于写操作的读操作均会进入等待状态，只有写操作完成并进行通知之后，所有等待的读操作才能继续执行（写操作之间依靠`synchronized`关键字进行同步），这样做的目的是使读操作都能读取到正确的数据，而不会出现脏读。

改用读写锁实现上述功能，只需要在读操作时获取读锁，而写操作时获取写锁即可，当写锁被获取到时，后续（非当前写操作线程）的读写操作都会被阻塞，写锁释放之后，所有操作继续执行，编程方式相对于使用等待通知机制的实现方式而言，变得简单明了。

读写锁维护了一对锁，读锁和写锁，通过锁的分离，使得并发性比一般的排他锁有了很大的提高。`ReentrantReadWriteLock`就是这样的Java提供的可重入的读写锁。其结果如下：

![ReentrantReadWriteLock](http://ovn0i3kdg.bkt.clouddn.com/ReentrantReadWriteLock.png?imageView/2/w/400 )

`ReentrantReadWriteLock`支持如下性能：
1. 公平性选择：支持非公平（默认）和公平性锁获取方式，前者吞吐量大于后者
2. 锁重入：读线程在获取了读锁之后，能够再次获取读锁。而写线程在获取了写锁之后能够再次获取写锁，同时也可以获取读锁。
3. **锁降级**：遵循获取写锁、获取读锁再释放写锁的次序，写锁能够降级成为读锁


我们还是从源码角度看看这些性能是如何实现的。


### public class ReentrantReadWriteLock implements ReadWriteLock, java.io.Serializable
类声明，可见`ReentrantReadWriteLock`实现`ReadWriteLock`接口，而该接口的结构如下：

![ReadWriteLock](http://ovn0i3kdg.bkt.clouddn.com/ReadWriteLock.png)

接口定义了两个锁，一个读锁，一个写锁。注意，`ReentrantReadWriteLock`并没有继承`Lock`类，也没有继承AQS。回想一下`ReentrantLock`也没有继承`Lock`类，而是利用内部类实现了AQS。


### 构造方法
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

另外，由于继承`ReadWriteLock`，所以`ReentrantReadWriteLock`必须提供获取读写锁的方法。该类定义分别定义了读锁和写锁两个静态内部类。这两个类都实现了`Lock`接口。

关于读写锁的一个比较形象的介绍可以参考 http://blog.csdn.net/yanyan19880509/article/details/52435135



参考
* [轻松掌握java读写锁(ReentrantReadWriteLock)的实现原理](http://blog.csdn.net/yanyan19880509/article/details/52435135)
