# ThreadPoolExecutor


线程池的使用解决了两个问题：
1. 减少了每个任务的调用开销，所以在执行大量异步任务的是偶提供了改进的性能。
2. 提供了一种限制和管理资源（包括线程）的方法，每个`ThreadPoolExecutor`还维护了一些基本统计信息，例如已经完成任务的数量。

`ThreadPoolExecutor`是我们最常用的线程池，其结果如下：

![ThreadPoolExecutor](http://ovn0i3kdg.bkt.clouddn.com/ThreadPoolExecutor_1.png)
![ThreadPoolExecutor](http://ovn0i3kdg.bkt.clouddn.com/ThreadPoolExecutor_2.png)
![ThreadPoolExecutor](http://ovn0i3kdg.bkt.clouddn.com/ThreadPoolExecutor_3.png)


这个类中提供了很多可调参数和可扩展性的钩子，但是实际上，我们最常使用的是使用`Executors`工厂类中的三种方法：
* `Executors.newCachedThreadPool()`：创建无界线程池，带自动线程回收。
* `Executors.newFixedThreadPool（int)`：创建固定大小的线程池。
* `Executors.newSingleThreadExecutor（）`：创建单线程话线程池。

这些是我们常见的线程池的配置预设，极少情况下才需要我们手动配置和调整。配置和调整什么呢？可以有以下选择（以下内容翻译自[Java 8 doc API](https://docs.oracle.com/javase/8/docs/api/)中`ThreadPoolExecutor`的说明）：
* 核心和最大池大小
`ThreadPoolExecutor`将根据`corePoolSize`（参阅`getCorePoolSize`）和`maximumPoolSize`（请参阅`getMaximumPoolSize`）设置的边界自动调整线程池大小（参阅`getPoolSize`）。当方法`execute(Runnable)`中提交新任务并且少于`corePoolSize`线程正在处于运行时，即使其他工作线程处于空闲状态，也会创建一个新线程来处理该请求。如果有多余`corePoolSize`但是少于`maximumPoolSize`线程正在运行，则仅当队列已满时才会创建新的线程。通过设置`corePoolSize`和`maximumPoolSize`相同，我们可以创建一个固定大小的线程池；通过`maximumPoolSize`设置为基本无界的值（比如`Integer.MAX_VALUE`），可以让线程池容纳任意数量的并发任务。通常，核心和最大池大小只在构建时设置，但是也可以通过`setCorePoolSize(int)`和`setMaxmumPoolSize(int)`来动态更改。

* 按需建设
默认情况下，即使核心线程最初是在新任务到达时创建的，但是可以使用方法`prestartCoreThread()`或`prestartAllCoreThreads()`来动态重写。如果使用非空队列构建池，则可能需要预先启动线程。

* 创建新线程
没有特别说明，使用一个`Executors.defaultThreadFactory()`，它创建的线程都在同一个`ThreadGroup`中，并且具有相同的`NORM_PRIORITY`优先级和守护进程状态。通过提供不同的`ThreadFactory`，可以更改线程的名称、线程组、优先级、守护进程状态等。如果`ThreadFactory`在从`newThread`返回`null`时未能创建线程，则执行程序将继续，但可能无法执行任何任务。线程拥有`modifyThread` RuntimePermission。如果工作线程或使用该池的其他线程不具有此权限，则服务可能降级，配置更改可能无法及时生效，并且关闭池可能会保持可终止但尚未完成状态。
* 线程活跃数量
如果池当前拥有多个`corePoolSize`线程，那么多余的线程将被终止，如果他们的空闲时间已经超过了`keepAliveTime`（请参阅`getKeepAliveTime(TimeUnit)`）。这提供了一种在不积极使用池时减少资源消耗的方法。如果池在以后变的更加活跃，则将构建新线程。此参数也可以使用方法`setKeepAliveTime(long, TimeUnit)`动态更改，使用`Long.MAX_VALUE`值、`TimeUnit.NANOSECONDS`可有效地禁用在关闭之前终止空闲线程。默认情况下，保持活动策略仅适用于存在多余`corePoolSize`线程的情况。但是，只要`keepAliveTime`值不为0，方法`allowCoreThreadTimeOut(boolean)`也可用于将此超时策略应用于核心线程。
* 队列
任何`BlockingQueue`都可以用来传输和保存提交的任务。此队列的使用与池大小进行交互：
  - 如果少于`corePoolSize`线程正在运行，那么`Executor`总是倾向于添加新线程而不是排队。
  - 如果`corePoolSize`或更多线程正在运行，那么`Executor`总是倾向于排队请求而不是添加新线程。
  - 如果请求不能排队，则会创建一个新线程，除非这会超多`maximumPoolSize`，在这种情况下，该任务将被拒绝。
有三个通用的排队策略：
  - 直接切换。对于工作队列来说，一个很好的默认选择是`SynchronousQueue`，它将任务交给下城而不需要保留。这里，如果没有线程立即可用来运行它，那么排队任务的尝试将失败，因此将构建新的新城。此策略咋处理可能具有内部依赖关系的请求集时避免锁定。直接切换通常需要无限制的`maximumPoolSize`来避免拒绝新提交的任务。这反过来又承认，当命令持续以平均速度超多可处理速度时，线程的无线增长称为可能。
  - 无界队列。在所有`corePoolSize`线程繁忙时，使用无界队列（例如没有预定义容量的`LinkedBlockingQueue`）将倒置新任务在队列中等待。因此，不会创建`corePoolSize`线程。（因为`maximumPoolSize`的值没有任何作用。）当每个任务完全独立于其他任务时，这可能是适当的，因此任务不能影响其他任何执行。例如，在网页服务器中，虽然这种排队方式可以有效地消除瞬时突发请求，但是它承认当命令持续以平均速度快于处理的速度到达时，无限制的工作队列增长的可能性。
  - 有界队列。有界队列（例如，`ArrayBlockingQueue`）有助于防止在使用有限的`maximumPoolSize`时资源耗尽，但可有更难以调整和控制。队列的大小和最大池大小可以彼此交换；使用大队列和小池可以最大限度地减少CPU的使用率，操作系统资源和上下文切换开销，但会导致任务的低吞吐量，如果任务经常被阻塞（例如，如果它们是I/O绑定的），那么系统可能会安排更多线程的时间，而不是我们设定的时间。使用小队列通常需要更大的池，这会使得CPU更加繁忙，但可能会遇到不可接受的调度开销，这也会降低吞吐量。

* 拒绝任务
当执行程序关闭时，以及执行程序对最大线程和工作队列容量使用有限边界并且达到饱和时，在方法execute（Runnable）中提交的新任务将被拒绝。 无论哪种情况，执行方法都会调用`RejectedExecutionHandler`的 `RejectedExecutionHandler.rejectedExecution（Runnable，ThreadPoolExecutor）`方法。 提供四个预定义的处理程序策略:
  * 在默认的`ThreadPoolExecutor.AbortPolicy`中，该处理程序在拒绝时抛出运行时`RejectedExecutionException`。
  * 在`ThreadPoolExecutor.CallerRunsPolicy`中，调用自身执行的线程运行该任务。 这提供了一个简单的反馈控制机制，可以减慢提交新任务的速度。
  * 在`ThreadPoolExecutor.DiscardPolicy`中，无法执行的任务将被删除。
  * 在`ThreadPoolExecutor.DiscardOldestPolicy`中，如果执行程序未关闭，则会删除工作队列头部的任务，然后重试执行（可能会再次失败，从而导致重复执行）。
可以定义和使用其他种类的`RejectedExecutionHandler`类。 这样做需要特别注意，特别是在政策旨在仅在特定容量或排队策略下工作时。
* 钩子方法
该类提供了在执行每个任务之前和之后调用的受保护的可覆盖`beforeExecute（Thread，Runnable）`和`afterExecute（Runnable，Throwable）`方法。 这些可以用来操纵执行环境; 例如，重新初始化`ThreadLocals`，收集统计信息或添加日志条目。 另外，可以重写方法`terminate（）`来执行`Executor`完全终止后需要完成的任何特殊处理。
如果钩子或回调方法抛出异常，则内部工作者线程可能会失败并突然终止。
* 队列维护
方法`getQueue（）`允许访问工作队列以用于监视和调试。 强烈建议不要将此方法用于任何其他目的。 提供两种提供的方法`remove（Runnable）`和`purge（）`可用于在大量排队的任务被取消时协助存储回收。
* Finalization
一个程序中不再被引用且没有剩余线程的池将自动关闭。 如果您希望确保即使用户忘记调用`shutdown（）`，也可以回收未引用的池，那么您必须通过设置适当的保持活动时间，使用零核心线程的下限和/或设置`allowCoreThreadTimeOut（boolean）`。


上面是对`ThreadPoolExecutor`的说明，讲了很多但是一头雾水，还是看看源码是如何实现的吧。
## public class ThreadPoolExecutor extends AbstractExecutorService
类声明，直接继承自`AbstractExecutorService`，而这个抽象类又实现了`ExecutorService`接口，类结果如下：

![AbstractExecutorService](http://ovn0i3kdg.bkt.clouddn.com/AbstractExecutorService.png)


## 构造方法
重载了四种构造方法。核心是下面这个
```java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler) {
    if (corePoolSize < 0 ||
        maximumPoolSize <= 0 ||
        maximumPoolSize < corePoolSize ||
        keepAliveTime < 0)
        throw new IllegalArgumentException();
    if (workQueue == null || threadFactory == null || handler == null)
        throw new NullPointerException();
    this.acc = System.getSecurityManager() == null ?
            null :
            AccessController.getContext();
    this.corePoolSize = corePoolSize;
    this.maximumPoolSize = maximumPoolSize;
    this.workQueue = workQueue;
    this.keepAliveTime = unit.toNanos(keepAliveTime);
    this.threadFactory = threadFactory;
    this.handler = handler;
}
/* The context to be used when executing the finalizer, or null. */
 private final AccessControlContext acc;

 /**
 * Core pool size is the minimum number of workers to keep alive
 * (and not allow to time out etc) unless allowCoreThreadTimeOut
 * is set, in which case the minimum is zero.
 */
private volatile int corePoolSize;

/**
 * Maximum pool size. Note that the actual maximum is internally
 * bounded by CAPACITY.
 */
private volatile int maximumPoolSize;

/**
* The queue used for holding tasks and handing off to worker
* threads.  We do not require that workQueue.poll() returning
* null necessarily means that workQueue.isEmpty(), so rely
* solely on isEmpty to see if the queue is empty (which we must
* do for example when deciding whether to transition from
* SHUTDOWN to TIDYING).  This accommodates special-purpose
* queues such as DelayQueues for which poll() is allowed to
* return null even if it may later return non-null when delays
* expire.
*/
private final BlockingQueue<Runnable> workQueue;

/**
 * Timeout in nanoseconds for idle threads waiting for work.
 * Threads use this timeout when there are more than corePoolSize
 * present or if allowCoreThreadTimeOut. Otherwise they wait
 * forever for new work.
 */
private volatile long keepAliveTime;


/**
* Factory for new threads. All threads are created using this
* factory (via method addWorker).  All callers must be prepared
* for addWorker to fail, which may reflect a system or user's
* policy limiting the number of threads.  Even though it is not
* treated as an error, failure to create threads may result in
* new tasks being rejected or existing ones remaining stuck in
* the queue.
*
* We go further and preserve pool invariants even in the face of
* errors such as OutOfMemoryError, that might be thrown while
* trying to create threads.  Such errors are rather common due to
* the need to allocate a native stack in Thread.start, and users
* will want to perform clean pool shutdown to clean up.  There
* will likely be enough memory available for the cleanup code to
* complete without encountering yet another OutOfMemoryError.
*/
private volatile ThreadFactory threadFactory;

/**
 * Handler called when saturated or shutdown in execute.
 */
private volatile RejectedExecutionHandler handler;
```

这个构造方法中的参数与线程的管理关系密切，我们必须弄明白各自的含义
### corePoolSize
线程池的**基本大小**，即在没有任务需要执行的时候线程池的大小，并且只有在工作队列满了的情况下才会创建超出这个数量的线程。这里需要注意的是：在刚刚创建`ThreadPoolExecutor`的时候，线程并不会立即启动，而是要等到有任务提交时才会启动，除非调用了`prestartCoreThread/prestartAllCoreThreads`事先启动核心线程。再考虑到`keepAliveTime`和`allowCoreThreadTimeOut`超时参数的影响，所以没有任务需要执行的时候，线程池的大小不一定是`corePoolSize`。
> 什么是核心线程呢？就是预先创建出来的，一直存活的线程。即使没有任务需要执行也一直活着，"一直活着"这种说法不准确，至少如果设置`allowCoreThreadTimeOut=true`的时候，核心线程超时后是会退出的。

### maximumPoolSize
线程池中允许的**最大线程数**，线程池中的当前线程数目不会超过该值。如果队列中任务已满，并且当前线程个数小于`maximumPoolSize`，那么会创建新的线程来执行任务。这里值得一提的是`largestPoolSize`，该变量记录了线程池在整个生命周期中**曾经**出现的最大线程个数。为什么说是曾经呢？因为线程池创建之后，可以调用`setMaximumPoolSize()`改变运行的最大线程的数目。

线程池中还有一个线程池大小的概念`poolSize`，这个量是指线程池中当前线程的实际数量，当该值为0的时候，意味着没有任何线程，线程池会终止；同一时刻，`poolSize`不会超过`maximumPoolSize`。

### workQueue
`workQueue`是`BlockingQueue`类型，这是一个阻塞队列，当新任务条件之后，发现`corePool`的数量大于等于`corePoolSize`的时候，将会将任务放到阻塞队列中。`BlockingQueue`是一个接口，在API中推荐使用三种该接口的实现类：`SynchronousQueue`、`LinkedBlockingQueue`和`ArrayBlockingQueue`。这些具体说明可以参考其对应的源码解决的博文。

### keepAliveTime和unit
如果一个线程处在**空闲状态**的时间超过了该属性值，就会因为超时而退出。举个例子，如果线程池的核心大小`corePoolSize=5`，而当前大小`poolSize =8`，那么超出核心大小的线程，会按照`keepAliveTime`的值判断是否会超时退出。如果线程池的核心大小`corePoolSize=5`，而当前大小`poolSize =5`，那么线程池中所有线程都是核心线程，这个时候线程是否会退出，取决于`allowCoreThreadTimeOut`。

`allowCoreThreadTimeOut`该属性用来控制是否允许**核心线程**超时退出。如果该值为false，那么核心线程保持为存活状态，不管是不是空闲，如果为true，那么核心线程的空闲时间如果达到了`keepAliveTime`，那么就会推出。如果线程池的大小已经达到了corePoolSize，不管有没有任务需要执行，线程池都会保证这些核心线程处于存活状态。可以知道：该属性只是用来控制核心线程的。

### handler
`handler`是`RejectedExecutionHandler`类型的变量，表示的是线程拒绝策略。`RejectedExecutionHandler`是一个接口，在`ThreadPoolExecutor`中有四种该接口的实现类：
#### AbortPolicy
对拒绝任务抛弃处理，并且抛出异常。
```java
/**
* A handler for rejected tasks that throws a
* {@code RejectedExecutionException}.
*/
public static class AbortPolicy implements RejectedExecutionHandler {
    /**
     * Creates an {@code AbortPolicy}.
     */
    public AbortPolicy() { }

    /**
     * Always throws RejectedExecutionException.
     *
     * @param r the runnable task requested to be executed
     * @param e the executor attempting to execute this task
     * @throws RejectedExecutionException always
     */
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
        throw new RejectedExecutionException("Task " + r.toString() +
                                             " rejected from " +
                                             e.toString());
    }
}
```

#### CallerRunsPolicy
这个策略重试添加当前的任务，他会自动重复调用 execute() 方法，直到成功。
```Java
/**
 * A handler for rejected tasks that runs the rejected task
 * directly in the calling thread of the {@code execute} method,
 * unless the executor has been shut down, in which case the task
 * is discarded.
 */
public static class CallerRunsPolicy implements RejectedExecutionHandler {
    /**
     * Creates a {@code CallerRunsPolicy}.
     */
    public CallerRunsPolicy() { }

    /**
     * Executes task r in the caller's thread, unless the executor
     * has been shut down, in which case the task is discarded.
     *
     * @param r the runnable task requested to be executed
     * @param e the executor attempting to execute this task
     */
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
        if (!e.isShutdown()) {
            r.run();
        }
    }
}
```

### DiscardPolicy
对拒绝任务直接无声抛弃，没有异常信息。
```java
/**
 * A handler for rejected tasks that silently discards the
 * rejected task.
 */
public static class DiscardPolicy implements RejectedExecutionHandler {
    /**
     * Creates a {@code DiscardPolicy}.
     */
    public DiscardPolicy() { }

    /**
     * Does nothing, which has the effect of discarding task r.
     *
     * @param r the runnable task requested to be executed
     * @param e the executor attempting to execute this task
     */
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
    }
}
```
### DiscardOldestPolicy
对拒绝任务不抛弃，而是抛弃队列里面等待最久的一个线程，然后把拒绝任务加到队列。
```java
/**
 * A handler for rejected tasks that discards the oldest unhandled
 * request and then retries {@code execute}, unless the executor
 * is shut down, in which case the task is discarded.
 */
public static class DiscardOldestPolicy implements RejectedExecutionHandler {
    /**
     * Creates a {@code DiscardOldestPolicy} for the given executor.
     */
    public DiscardOldestPolicy() { }

    /**
     * Obtains and ignores the next task that the executor
     * would otherwise execute, if one is immediately available,
     * and then retries execution of task r, unless the executor
     * is shut down, in which case task r is instead discarded.
     *
     * @param r the runnable task requested to be executed
     * @param e the executor attempting to execute this task
     */
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
        if (!e.isShutdown()) {
            e.getQueue().poll();
            e.execute(r);
        }
    }
}
```

以上，各类参数基本塑造了整个线程池的轮廓。在学习各种细节之前，我们用下面这张图来串联以下`ThreadPoolExecutor`的工作流程：
![ThreadPoolExecutor工作流程](https://upload-images.jianshu.io/upload_images/2177145-33c7b5ff75cf2bf7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

当调用`ThreadPoolExecutor`的`execute`提交任务后：
1. 首先检查CorePool中线程的个数，如果此时CorePool中线程的数目于`corePoolSize`，那么**创建新线程**来执行任务。注意，这里是创建新线程，即使有空闲的核心线程，线程池也会优先创建新线程处理。
2. 如果当前CorePool内的线程大于等于`CorePoolSize`，那么将线程尝试加入到`BlockingQueue`。
3. 如果不能加入`BlockingQueue`，在小于`MaxPoolSize`的情况下创建线程执行任务。
4. 如果线程数大于等于`MaxPoolSize`，那么执行拒绝策略。


对于线程池的工作过程，《编写高质量代码 改善Java程序的151个建议》这本书里举的这个例子很形象：
![关于线程池的一个比喻](http://img.blog.csdn.net/20161107130709586?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
![关于线程池的一个比喻](http://img.blog.csdn.net/20161107130723836?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

## public void execute(Runnable command){..}
这是线程池提交新任务的方法，定义如下
```java
/**
 * Executes the given task sometime in the future.  The task
 * may execute in a new thread or in an existing pooled thread.
 *
 * If the task cannot be submitted for execution, either because this
 * executor has been shutdown or because its capacity has been reached,
 * the task is handled by the current {@code RejectedExecutionHandler}.
 *
 * @param command the task to execute
 * @throws RejectedExecutionException at discretion of
 *         {@code RejectedExecutionHandler}, if the task
 *         cannot be accepted for execution
 * @throws NullPointerException if {@code command} is null
 */
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();
    /*
     * Proceed in 3 steps:
     *
     * 1. If fewer than corePoolSize threads are running, try to
     * start a new thread with the given command as its first
     * task.  The call to addWorker atomically checks runState and
     * workerCount, and so prevents false alarms that would add
     * threads when it shouldn't, by returning false.
     *
     * 2. If a task can be successfully queued, then we still need
     * to double-check whether we should have added a thread
     * (because existing ones died since last checking) or that
     * the pool shut down since entry into this method. So we
     * recheck state and if necessary roll back the enqueuing if
     * stopped, or start a new thread if there are none.
     *
     * 3. If we cannot queue task, then we try to add a new
     * thread.  If it fails, we know we are shut down or saturated
     * and so reject the task.
     */
    int c = ctl.get();
    if (workerCountOf(c) < corePoolSize) {
        if (addWorker(command, true))
            return;
        c = ctl.get();
    }
    if (isRunning(c) && workQueue.offer(command)) {
        int recheck = ctl.get();
        if (! isRunning(recheck) && remove(command))
            reject(command);
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);
    }
    else if (!addWorker(command, false))
        reject(command);
}

/**
 * The main pool control state, ctl, is an atomic integer packing
 * two conceptual fields
 *   workerCount, indicating the effective number of threads
 *   runState,    indicating whether running, shutting down etc
 *
 * In order to pack them into one int, we limit workerCount to
 * (2^29)-1 (about 500 million) threads rather than (2^31)-1 (2
 * billion) otherwise representable. If this is ever an issue in
 * the future, the variable can be changed to be an AtomicLong,
 * and the shift/mask constants below adjusted. But until the need
 * arises, this code is a bit faster and simpler using an int.
 *
 * The workerCount is the number of workers that have been
 * permitted to start and not permitted to stop.  The value may be
 * transiently different from the actual number of live threads,
 * for example when a ThreadFactory fails to create a thread when
 * asked, and when exiting threads are still performing
 * bookkeeping before terminating. The user-visible pool size is
 * reported as the current size of the workers set.
 *
 * The runState provides the main lifecycle control, taking on values:
 *
 *   RUNNING:  Accept new tasks and process queued tasks
 *   SHUTDOWN: Don't accept new tasks, but process queued tasks
 *   STOP:     Don't accept new tasks, don't process queued tasks,
 *             and interrupt in-progress tasks
 *   TIDYING:  All tasks have terminated, workerCount is zero,
 *             the thread transitioning to state TIDYING
 *             will run the terminated() hook method
 *   TERMINATED: terminated() has completed
 *
 * The numerical order among these values matters, to allow
 * ordered comparisons. The runState monotonically increases over
 * time, but need not hit each state. The transitions are:
 *
 * RUNNING -> SHUTDOWN
 *    On invocation of shutdown(), perhaps implicitly in finalize()
 * (RUNNING or SHUTDOWN) -> STOP
 *    On invocation of shutdownNow()
 * SHUTDOWN -> TIDYING
 *    When both queue and pool are empty
 * STOP -> TIDYING
 *    When pool is empty
 * TIDYING -> TERMINATED
 *    When the terminated() hook method has completed
 *
 * Threads waiting in awaitTermination() will return when the
 * state reaches TERMINATED.
 *
 * Detecting the transition from SHUTDOWN to TIDYING is less
 * straightforward than you'd like because the queue may become
 * empty after non-empty and vice versa during SHUTDOWN state, but
 * we can only terminate if, after seeing that it is empty, we see
 * that workerCount is 0 (which sometimes entails a recheck -- see
 * below).
 */
 private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
 private static final int COUNT_BITS = Integer.SIZE - 3;
 private static final int CAPACITY   = (1 << COUNT_BITS) - 1;

 // runState is stored in the high-order bits
 private static final int RUNNING    = -1 << COUNT_BITS;
 private static final int SHUTDOWN   =  0 << COUNT_BITS;
 private static final int STOP       =  1 << COUNT_BITS;
 private static final int TIDYING    =  2 << COUNT_BITS;
 private static final int TERMINATED =  3 << COUNT_BITS;

 // Packing and unpacking ctl
 private static int runStateOf(int c)     { return c & ~CAPACITY; }
 private static int workerCountOf(int c)  { return c & CAPACITY; }
 private static int ctlOf(int rs, int wc) { return rs | wc; }
```
说实话具体的没怎么看懂，但是大概能理解，讲的就是我们上面画的那张图的工作流程。

## public void shutdown() {..}
```java
/**
 * Initiates an orderly shutdown in which previously submitted
 * tasks are executed, but no new tasks will be accepted.
 * Invocation has no additional effect if already shut down.
 *
 * <p>This method does not wait for previously submitted tasks to
 * complete execution.  Use {@link #awaitTermination awaitTermination}
 * to do that.
 *
 * @throws SecurityException {@inheritDoc}
 */
public void shutdown() {
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        checkShutdownAccess();
        advanceRunState(SHUTDOWN);
        interruptIdleWorkers();
        onShutdown(); // hook for ScheduledThreadPoolExecutor
    } finally {
        mainLock.unlock();
    }
    tryTerminate();
}

/**
 * Lock held on access to workers set and related bookkeeping.
 * While we could use a concurrent set of some sort, it turns out
 * to be generally preferable to use a lock. Among the reasons is
 * that this serializes interruptIdleWorkers, which avoids
 * unnecessary interrupt storms, especially during shutdown.
 * Otherwise exiting threads would concurrently interrupt those
 * that have not yet interrupted. It also simplifies some of the
 * associated statistics bookkeeping of largestPoolSize etc. We
 * also hold mainLock on shutdown and shutdownNow, for the sake of
 * ensuring workers set is stable while separately checking
 * permission to interrupt and actually interrupting.
 */
private final ReentrantLock mainLock = new ReentrantLock();

/**
 * If there is a security manager, makes sure caller has
 * permission to shut down threads in general (see shutdownPerm).
 * If this passes, additionally makes sure the caller is allowed
 * to interrupt each worker thread. This might not be true even if
 * first check passed, if the SecurityManager treats some threads
 * specially.
 */
private void checkShutdownAccess() {
    SecurityManager security = System.getSecurityManager();
    if (security != null) {
        security.checkPermission(shutdownPerm);
        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
            for (Worker w : workers)
                security.checkAccess(w.thread);
        } finally {
            mainLock.unlock();
        }
    }
}

/**
 * Transitions runState to given target, or leaves it alone if
 * already at least the given target.
 *
 * @param targetState the desired state, either SHUTDOWN or STOP
 *        (but not TIDYING or TERMINATED -- use tryTerminate for that)
 */
private void advanceRunState(int targetState) {
    for (;;) {
        int c = ctl.get();
        if (runStateAtLeast(c, targetState) ||
            ctl.compareAndSet(c, ctlOf(targetState, workerCountOf(c))))
            break;
}
}

/**
* Common form of interruptIdleWorkers, to avoid having to
* remember what the boolean argument means.
*/
private void interruptIdleWorkers() {
interruptIdleWorkers(false);
}

/**
* Interrupts threads that might be waiting for tasks (as
* indicated by not being locked) so they can check for
* termination or configuration changes. Ignores
* SecurityExceptions (in which case some threads may remain
* uninterrupted).
*
* @param onlyOne If true, interrupt at most one worker. This is
* called only from tryTerminate when termination is otherwise
* enabled but there are still other workers.  In this case, at
* most one waiting worker is interrupted to propagate shutdown
* signals in case all threads are currently waiting.
* Interrupting any arbitrary thread ensures that newly arriving
* workers since shutdown began will also eventually exit.
* To guarantee eventual termination, it suffices to always
* interrupt only one idle worker, but shutdown() interrupts all
* idle workers so that redundant workers exit promptly, not
* waiting for a straggler task to finish.
*/
private void interruptIdleWorkers(boolean onlyOne) {
  final ReentrantLock mainLock = this.mainLock;
  mainLock.lock();
  try {
  for (Worker w : workers) {
      Thread t = w.thread;
      if (!t.isInterrupted() && w.tryLock()) {
          try {
              t.interrupt();
          } catch (SecurityException ignore) {
          } finally {
              w.unlock();
          }
      }
      if (onlyOne)
          break;
  }
  } finally {
  mainLock.unlock();
  }
}

/**
 * Performs any further cleanup following run state transition on
 * invocation of shutdown.  A no-op here, but used by
 * ScheduledThreadPoolExecutor to cancel delayed tasks.
 */
void onShutdown() {
}

/**
 * Transitions to TERMINATED state if either (SHUTDOWN and pool
 * and queue empty) or (STOP and pool empty).  If otherwise
 * eligible to terminate but workerCount is nonzero, interrupts an
 * idle worker to ensure that shutdown signals propagate. This
 * method must be called following any action that might make
 * termination possible -- reducing worker count or removing tasks
 * from the queue during shutdown. The method is non-private to
 * allow access from ScheduledThreadPoolExecutor.
 */
final void tryTerminate() {
    for (;;) {
        int c = ctl.get();
        if (isRunning(c) ||
            runStateAtLeast(c, TIDYING) ||
            (runStateOf(c) == SHUTDOWN && ! workQueue.isEmpty()))
            return;
        if (workerCountOf(c) != 0) { // Eligible to terminate
            interruptIdleWorkers(ONLY_ONE);
            return;
        }

        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
            if (ctl.compareAndSet(c, ctlOf(TIDYING, 0))) {
                try {
                    terminated();
                } finally {
                    ctl.set(ctlOf(TERMINATED, 0));
                    termination.signalAll();
                }
                return;
            }
        } finally {
            mainLock.unlock();
        }
        // else retry on failed CAS
    }
}
```
`shutdown`只是将空闲的线程`interrupt()` 了， 因此在`shutdown（）`之前提交的任务可以继续执行直到结束。

## public List<Runnable> shutdownNow() {..}
与`shutdown`方法不同的是，`shutdownNow`方法将`interrupt`**所有** 线程， 因此大部分线程将立刻被中断。之所以是大部分，而不是全部 ，是因为`interrupt()`方法能力有限。但是调用该方法并不代表线程池就一定立即就能退出，它可能必须要等待所有正在执行的任务都执行完成了才能退出。
```java
/**
 * Attempts to stop all actively executing tasks, halts the
 * processing of waiting tasks, and returns a list of the tasks
 * that were awaiting execution. These tasks are drained (removed)
 * from the task queue upon return from this method.
 *
 * <p>This method does not wait for actively executing tasks to
 * terminate.  Use {@link #awaitTermination awaitTermination} to
 * do that.
 *
 * <p>There are no guarantees beyond best-effort attempts to stop
 * processing actively executing tasks.  This implementation
 * cancels tasks via {@link Thread#interrupt}, so any task that
 * fails to respond to interrupts may never terminate.
 *
 * @throws SecurityException {@inheritDoc}
 */
public List<Runnable> shutdownNow() {
    List<Runnable> tasks;
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        checkShutdownAccess();
        advanceRunState(STOP);
        interruptWorkers();
        tasks = drainQueue();
    } finally {
        mainLock.unlock();
    }
    tryTerminate();
    return tasks;
}
```

> 剩下的方法其实基本上都不太用到，比如修改`corePoolSize`的大小、修改`maximumPoolSize`的大小，这些量其实在创建池的时候已经定好了，很少改变。另外，更多的情况，我们是使用的是`Executors`工厂类来创建线程池。所以重点还是放在`Executors`上吧。其他方法如果有兴趣后面再整理。


参考
* [理解ThreadPoolExecutor源码(一)线程池的corePoolSize、maximumPoolSize和poolSize](http://blog.csdn.net/aitangyong/article/details/38822505)
* [深入理解java线程池—ThreadPoolExecutor](https://www.jianshu.com/p/ade771d2c9c0)
* [Java 线程池 ThreadPoolExecutor.(包含拒绝策略CallerRunsPolicy,AbortPolicy,DiscardPolicy,DiscardOldestPolicy )](http://blog.csdn.net/psiitoy/article/details/38587293)
* [Java多线程研究05-ThreadPoolExecutor中workQueue、threadFactory和handle](http://blog.csdn.net/wfzczangpeng/article/details/52015866)
* [ThreadPoolExecutor线程池参数设置技巧](https://www.cnblogs.com/waytobestcoder/p/5323130.html)
* [多线程之线程池newFixedThreadPool（二）](http://blog.csdn.net/zknxx/article/details/53057683)
