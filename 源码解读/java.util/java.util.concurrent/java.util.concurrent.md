# java.util.concurrent

这个包是JDK5引入的新包，`java.util.current`包（常常成为JUC）中提供了对线程优化、管理的各项操作，使得线程的使用变得的心应手。该包提供了线程的运行，线程池的创建，线程生命周期的控制，线程间的协作等功能。如果一些类名看起来相似，可能是因为`java.util.concurrent`中的许多概念源自 `Doug Lea `的 `util.concurrent` 库。

> 作者Doug Lea大概是世界上对Java影响最大的人了。可以看下它关于并发文章，译文见 http://ifeve.com/doug-lea/

![java.util.concurrent](http://ovn0i3kdg.bkt.clouddn.com/java.util.concurrent.locks.png)

JDK 5.0 中的并发改进可以分为三组：
* **JVM 级别更改**。大多数现代处理器对并发对某一硬件级别提供支持，通常以 compare-and-swap （CAS）指令形式。CAS 是一种低级别的、细粒度的技术，它允许多个线程更新一个内存位置，同时能够检测其他线程的冲突并进行恢复。它是许多高性能并发算法的基础。在 JDK 5.0 之前，Java 语言中用于协调线程之间的访问的惟一原语是同步，同步是更重量级和粗粒度的。CAS操作都封装在java不公开的类库`sun.misc.Unsafe`，这个类包含了对院子操作的封装，具体用本地代码实现。公开 CAS 可以开发高度可伸缩的并发 Java 类。这些更改主要由 JDK 库类使用，而不是由开发人员使用。
* **低级实用程序类 -- 锁定和原子类**。使用 CAS 作为并发原语，`ReentrantLock` 类提供与 `synchronized` 原语相同的锁定和内存语义，然而这样可以更好地控制锁定（如计时的锁定等待、锁定轮询和可中断的锁定等待）和提供更好的可伸缩性（竞争时的高性能）。大多数开发人员将不再直接使用 `ReentrantLock` 类，而是使用在 `ReentrantLock` 类上构建的高级类。
* **高级实用程序类**。这些类实现并发构建块，每个计算机科学文本中都会讲述这些类 -- 信号、互斥、闩锁、屏障、交换程序、线程池和线程安全集合类等。大部分开发人员都可以在应用程序中用这些类，来替换许多（如果不是全部）同步、wait() 和 notify() 的使用，从而提高性能、可读性和正确性。

JUC大致上又可以分为三个子包：

| 子包 | 作用     |
| :------------- | :------------- |
| Atomic     | 原子数据的构建     |
|Locks   | 基本锁的实现，最重要的是AQS框架和LockSupport  |
|Concurrent   |  构建的一些高级工具，比如线程池、并发队列等 |


`Atomic`和`Locks`是真正存在的子包，最后一个实际上并不存在这个子包，有一些类散落在JUC中。它们都是一些高级的使用程序类，主要的重要的类有下面这些：


| 类名 | 功能 |
| :------------- | :------------- |
|  Executors |  线程池的工厂类|
|ConcurrentHashMap   |  |
|ConcurrentLinkedDeque   |   |
|ConcurrentLinkedQueue   |   |
|CopyOnWriteArrayList   |   |
|CopyOnWriteArratSet   |   |
|CountDownLatch   |   |
|DelayQueue   |   |
|LinkedBlockingDeque   |   |
|LinkedBlockingQueue   |   |
|PriorityBlockingQueue   |   |


定义的接口有下面这些

| 接口 | 功能 |
| :------------- | :------------- |
|Executor | 启动了该线程      |
|ExecutorService   | 控制线程的执行和取消   |
|ScheduledExecutorService   |  提供了线程启动时机的控制 |
|Callable  | 提供了对线程的控制  |
|Future   | 用于控制线程的执行  |

> [Java 8  API—— JUC](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/package-summary.html)



参考
* [Doug Lea并发编程文章全部译文](http://ifeve.com/doug-lea/)
