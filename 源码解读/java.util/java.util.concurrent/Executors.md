# Executors

`Executor`和`Executors`又是相差一个字符s，想到之前也有三对同样的命名：`Array`和`Arrays`、`Collection`和`Collections`、`Object`和`Objects`（这三对在之前说了说了很多遍了，今天终于来了新成员）。

`Executor`是一个接口，实际上是一个执行器，而`Executors`是一个工具类，专门为执行器提供服务的。所以里面有很多的静态方法，结果如下图所示：

![Executors](http://ovn0i3kdg.bkt.clouddn.com/Executors.png)

这个类非常重要，因为它提供了生成线程池的一些常用方法。


## public class Executors
类声明，很干净，因为它只是一个工具类，所以不存在任何的继承和实现。

## 构造函数
和大部分的工具类一样，`Executors`是不能实例化的，所以就将其构造函数私有化。略。

## 关于ExecutorService的支持
`Executors`是一个工厂类，生产的是各类的线程池。它们可能在容量、等待时间、阻塞队列选择、抛弃策略上的选择有所不同。实际上，创建它们的方法很简单，就是调用了`ThreadPoolExecutor`的不同的构造函数，赋予了不同的参数而已。下面来看看有哪些不同的线程池。

### newFixedThreadPool
`newFixedThreadPool`的目的是创建一个可重用**固定线程数**的线程池,以共享的**无界队列**方式来运行这些线程。有两个重定向方法。
```java
/**
 * Creates a thread pool that reuses a fixed number of threads
 * operating off a shared unbounded queue.  At any point, at most
 * {@code nThreads} threads will be active processing tasks.
 * If additional tasks are submitted when all threads are active,
 * they will wait in the queue until a thread is available.
 * If any thread terminates due to a failure during execution
 * prior to shutdown, a new one will take its place if needed to
 * execute subsequent tasks.  The threads in the pool will exist
 * until it is explicitly {@link ExecutorService#shutdown shutdown}.
 *
 * @param nThreads the number of threads in the pool
 * @return the newly created thread pool
 * @throws IllegalArgumentException if {@code nThreads <= 0}
 */
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}

/**
 * Creates a thread pool that reuses a fixed number of threads
 * operating off a shared unbounded queue, using the provided
 * ThreadFactory to create new threads when needed.  At any point,
 * at most {@code nThreads} threads will be active processing
 * tasks.  If additional tasks are submitted when all threads are
 * active, they will wait in the queue until a thread is
 * available.  If any thread terminates due to a failure during
 * execution prior to shutdown, a new one will take its place if
 * needed to execute subsequent tasks.  The threads in the pool will
 * exist until it is explicitly {@link ExecutorService#shutdown
 * shutdown}.
 *
 * @param nThreads the number of threads in the pool
 * @param threadFactory the factory to use when creating new threads
 * @return the newly created thread pool
 * @throws NullPointerException if threadFactory is null
 * @throws IllegalArgumentException if {@code nThreads <= 0}
 */
public static ExecutorService newFixedThreadPool(int nThreads, ThreadFactory threadFactory) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>(),
                                  threadFactory);
}
```
后面一个方法比前一个方法多了一个`ThreadFactory`参数的指定，其他都一样。
总的来说，`newFixedThreadPool`的参数如下：

| 参数 | 默认值 |
| :------------- | :------------- |
|  corePoolSize       | 自定义，nThreads    |
|maximumPoolSize   |  自定义，和corePoolSize保持一致 |
|keepAliveTime   |0   |
|wordQueue 类型  |LinkedBlockingQueue类型，默认长度为`Integer.MAX_VALUE`，是无界队列 |
|threadFactory   |  默认的defaultFactory或者自定义 |
|handler   | 默认`AbortPolicy`  |


对于`newFixedThreadPool`，有几个常见的问题：
1. 为什么要将`corePoolSize`和`maximumPoolSize`设置成一样？
因为`newFixedThreadPool`中用的是`LinkedBlockingQueue`（是无界队列），只要当前线程大于等于`corePoolSize`来的任务就直接加入到无界队列中，所以线程数不会超过`corePoolSize`，这样**`maximumPoolSize`** 没有用。例如，在 Web 页服务器中。这种排队可用于处理瞬态突发请求，当命令以超过队列所能处理的平均数连续到达时，此策略允许无界线程具有增长的可能性。
2. 为什么队列使用`LinkedBlockingQueue`?
设置的`corePoolSize`和`maximumPoolSize`相同，则创建的线程池是大小固定的，要保证线程池大小固定则需要`LinkedBlockingQueue`（无界队列）来保证来的任务能够放到任务队列中，不至于触发拒绝策略。
3. 为什么`keepAliveTime`会设置成0？因为`corePoolSize和maximumPoolSize`一样大，`KeepAliveTime`设置的时间会失效，所以设置为0。

`FixedThreadPool`的优点是能够保证所有的任务都被执行，永远不会拒绝新的任务；同时缺点是队列数量没有限制，在任务执行时间无限延长的这种极端情况下会造成内存问题。

### newCachedThreadPool
`newCachedThreadPool`创建了可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。
```java
/**
 * Creates a thread pool that creates new threads as needed, but
 * will reuse previously constructed threads when they are
 * available.  These pools will typically improve the performance
 * of programs that execute many short-lived asynchronous tasks.
 * Calls to {@code execute} will reuse previously constructed
 * threads if available. If no existing thread is available, a new
 * thread will be created and added to the pool. Threads that have
 * not been used for sixty seconds are terminated and removed from
 * the cache. Thus, a pool that remains idle for long enough will
 * not consume any resources. Note that pools with similar
 * properties but different details (for example, timeout parameters)
 * may be created using {@link ThreadPoolExecutor} constructors.
 *
 * @return the newly created thread pool
 */
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}

/**
 * Creates a thread pool that creates new threads as needed, but
 * will reuse previously constructed threads when they are
 * available, and uses the provided
 * ThreadFactory to create new threads when needed.
 * @param threadFactory the factory to use when creating new threads
 * @return the newly created thread pool
 * @throws NullPointerException if threadFactory is null
 */
public static ExecutorService newCachedThreadPool(ThreadFactory threadFactory) {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>(),
                                  threadFactory);
}
```
后面的方法比前面多了指定`ThreadFactory`参数的功能，其他都一样。

总的来说，`newCachedThreadPool`创建的线程池的参数如下：

| 参数 | 默认值 |
| :------------- | :------------- |
| corePoolSize | 0 |
|maximumPoolSize    |  Integer.MAX_VALUE |
|keepAliveTime    |  60分钟 |
|workQueue    |  SynchronousQueue类型，默认大小是1 |
|threadFactory   |  默认的defaultFactory或者自定义 |
|handler   | 默认`AbortPolicy`  |

可以看到，由于`corePoolSize`为0，所以每次新的新任务到来的时候，总是先缓存到阻塞队伍中。而这个组设队伍是`SynchronousQueue`类型，这个队列类型非常特殊，是因为队列只能存储1个元素，生产者线程对其的插入操作put必须等待消费者的移除操作take，反过来也一样，消费者移除数据操作必须等待生产者的插入。这个队列用于那些有依赖的线程，比如A线程必须在B线程之前进行，那么A作为生产者，B作为消费者。B将入队。而`maximumPoolSize`的大小是无限的，所以这就是为什么说`newCachedThreadPool`创建“缓存”线程池，可正式如此，`CachedTheadPool`对于对任务的处理策略是提交的任务会立即分配一个线程进行执行，线程池中线程数量会随着任务数的变化自动扩张和缩减，在任务执行时间无限延长的极端情况下会创建过多的线程。


同样，关于`newCachedThreadPool`也有一些问题：
1. 为什么要将`corePoolSize`设置成0？
答:因为队列使用`SynchronousQueue`，队列中只能存放一个任务，保证所有任务会先入队列，用于那些互相依赖的线程，比如线程A必须在线程B之前先执行。
2. 队列使用`SynchronousQueue`？
答：线程数会随着任务数量变化自动扩张和缩减，可以灵活回收空闲线程，用`SynchronousQueue`队列整好保证了`CachedTheadPool`的特点。


### newSingleThreadExecutor
```java
/**
 * Creates an Executor that uses a single worker thread operating
 * off an unbounded queue. (Note however that if this single
 * thread terminates due to a failure during execution prior to
 * shutdown, a new one will take its place if needed to execute
 * subsequent tasks.)  Tasks are guaranteed to execute
 * sequentially, and no more than one task will be active at any
 * given time. Unlike the otherwise equivalent
 * {@code newFixedThreadPool(1)} the returned executor is
 * guaranteed not to be reconfigurable to use additional threads.
 *
 * @return the newly created single-threaded Executor
 */
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}

/**
 * Creates an Executor that uses a single worker thread operating
 * off an unbounded queue, and uses the provided ThreadFactory to
 * create a new thread when needed. Unlike the otherwise
 * equivalent {@code newFixedThreadPool(1, threadFactory)} the
 * returned executor is guaranteed not to be reconfigurable to use
 * additional threads.
 *
 * @param threadFactory the factory to use when creating new
 * threads
 *
 * @return the newly created single-threaded Executor
 * @throws NullPointerException if threadFactory is null
 */
public static ExecutorService newSingleThreadExecutor(ThreadFactory threadFactory) {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>(),
                                threadFactory));
}
```
总的来说`newSingleThreadExecutor`的参数设置如下：

| 参数 | 默认值 |
| :------------- | :------------- |
| corePoolSize | 1 |
|maximumPoolSize    |  1 |
|keepAliveTime   |  0 |
|workQueue   | LinkedBlockingQueue， 长度默认为Integer.MAX_VALUE  |
|threadFactory   |  默认的defaultFactory或者自定义 |
|handler   | 默认`AbortPolicy`  |

可见`SingleThreadExecutor`的线程池固定大小为1，任务队列无界限。特别注意的是，这里用`FinalizableDelegatedExecutorService`对`ThreadPoolExecutor`进行了包装，这个类定义如下：
```java
static class FinalizableDelegatedExecutorService
    extends DelegatedExecutorService {
    FinalizableDelegatedExecutorService(ExecutorService executor) {
        super(executor);
    }
    protected void finalize() {
        super.shutdown();
    }
}

static class DelegatedExecutorService extends AbstractExecutorService {
    private final ExecutorService e;
    DelegatedExecutorService(ExecutorService executor) { e = executor; }
    public void execute(Runnable command) { e.execute(command); }
    public void shutdown() { e.shutdown(); }
    public List<Runnable> shutdownNow() { return e.shutdownNow(); }
    public boolean isShutdown() { return e.isShutdown(); }
    public boolean isTerminated() { return e.isTerminated(); }
    public boolean awaitTermination(long timeout, TimeUnit unit)
        throws InterruptedException {
        return e.awaitTermination(timeout, unit);
    }
    public Future<?> submit(Runnable task) {
        return e.submit(task);
    }
    public <T> Future<T> submit(Callable<T> task) {
        return e.submit(task);
    }
    public <T> Future<T> submit(Runnable task, T result) {
        return e.submit(task, result);
    }
    public <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
        throws InterruptedException {
        return e.invokeAll(tasks);
    }
    public <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks,
                                         long timeout, TimeUnit unit)
        throws InterruptedException {
        return e.invokeAll(tasks, timeout, unit);
    }
    public <T> T invokeAny(Collection<? extends Callable<T>> tasks)
        throws InterruptedException, ExecutionException {
        return e.invokeAny(tasks);
    }
    public <T> T invokeAny(Collection<? extends Callable<T>> tasks,
                           long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException {
        return e.invokeAny(tasks, timeout, unit);
    }
}
```
那么`SingleThreadExecutor`为什么要用`FinalizableDelegatedExecutorService`对`ThreadPoolExecutor`进行包装呢？

因为`SingleThreadExecutor`是单线程化线程池，用`DelegatedExecutorService`包装为了屏蔽`ThreadPoolExecutor`动态修改线程数量的功能，仅保留`Executor`中的方法。

`SingleThreadExecutor` 适用于在逻辑上需要单线程处理任务的场景，同时无界的`LinkedBlockingQueue`保证新任务都能够放入队列，不会被拒绝；缺点和`FixedThreadPool`相同，当处理任务无限等待的时候会造成内存问题。

### newScheduledThreadPool
创建定时任务线线程池：
```java

/**
 * Creates a thread pool that can schedule commands to run after a
 * given delay, or to execute periodically.
 * @param corePoolSize the number of threads to keep in the pool,
 * even if they are idle
 * @return a newly created scheduled thread pool
 * @throws IllegalArgumentException if {@code corePoolSize < 0}
 */
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
    return new ScheduledThreadPoolExecutor(corePoolSize);
}

/**
 * Creates a thread pool that can schedule commands to run after a
 * given delay, or to execute periodically.
 * @param corePoolSize the number of threads to keep in the pool,
 * even if they are idle
 * @param threadFactory the factory to use when the executor
 * creates a new thread
 * @return a newly created scheduled thread pool
 * @throws IllegalArgumentException if {@code corePoolSize < 0}
 * @throws NullPointerException if threadFactory is null
 */
public static ScheduledExecutorService newScheduledThreadPool(
        int corePoolSize, ThreadFactory threadFactory) {
    return new ScheduledThreadPoolExecutor(corePoolSize, threadFactory);
}
```

### newWorkStealingPool
```java
/**
 * Creates a thread pool that maintains enough threads to support
 * the given parallelism level, and may use multiple queues to
 * reduce contention. The parallelism level corresponds to the
 * maximum number of threads actively engaged in, or available to
 * engage in, task processing. The actual number of threads may
 * grow and shrink dynamically. A work-stealing pool makes no
 * guarantees about the order in which submitted tasks are
 * executed.
 *
 * @param parallelism the targeted parallelism level
 * @return the newly created thread pool
 * @throws IllegalArgumentException if {@code parallelism <= 0}
 * @since 1.8
 */
public static ExecutorService newWorkStealingPool(int parallelism) {
    return new ForkJoinPool
        (parallelism,
         ForkJoinPool.defaultForkJoinWorkerThreadFactory,
         null, true);
}

/**
 * Creates a work-stealing thread pool using all
 * {@link Runtime#availableProcessors available processors}
 * as its target parallelism level.
 * @return the newly created thread pool
 * @see #newWorkStealingPool(int)
 * @since 1.8
 */
public static ExecutorService newWorkStealingPool() {
    return new ForkJoinPool
        (Runtime.getRuntime().availableProcessors(),
         ForkJoinPool.defaultForkJoinWorkerThreadFactory,
         null, true);
}
```
### 线程池的大小选择问题

如何去选择线程池？线程池应该设置多大？没有固定的答案，只有适合的答案

* 关于线程池大小问题，可以参考这个公式，仅仅是参考而已。
```
启动线程数 = [ 任务执行时间 / ( 任务执行时间 - IO等待时间 ) ] x CPU内核数
```
计算示例可以参考 https://www.cnblogs.com/waytobestcoder/p/5323130.html
* 在控制线程池大小的基础上，尽量使用有界队列并且设置大小，避免OOM。
* 设置合理的驳回策略，适用于你的业务。
## 关于ThreadFactory的支持

`ThreadFactory`是函数式接口，顾名思义，是“线程工厂”，使用了工厂创建的设计模式，一般配合线程池使用。**主要用来控制创建新线程时的一些行为，比如设置线程的优先级，名字等等。** 接口定义如下：
```java
public interface ThreadFactory {

    /**
     * Constructs a new {@code Thread}.  Implementations may also initialize
     * priority, name, daemon status, {@code ThreadGroup}, etc.
     *
     * @param r a runnable to be executed by new thread instance
     * @return constructed thread, or {@code null} if the request to
     *         create a thread is rejected
     */
    Thread newThread(Runnable r);
}
```
就是用来创建线程的。该接口的实现类都散落在不同的类中，如下：

![ThreadFactory](http://ovn0i3kdg.bkt.clouddn.com/ThreadFactory%E7%9A%84%E5%AE%9E%E7%8E%B0%E7%B1%BB.png)


### DefaultThreadFactory
在`Executes`中定义了一个`ThreadFactory`的直接实现类`DefaultThreadFactory`：
```java
/**
 * The default thread factory
 */
static class DefaultThreadFactory implements ThreadFactory {
    private static final AtomicInteger poolNumber = new AtomicInteger(1);
    private final ThreadGroup group;
    private final AtomicInteger threadNumber = new AtomicInteger(1);
    private final String namePrefix;

    DefaultThreadFactory() {
        SecurityManager s = System.getSecurityManager();
        group = (s != null) ? s.getThreadGroup() :
                              Thread.currentThread().getThreadGroup();
        namePrefix = "pool-" +
                      poolNumber.getAndIncrement() +
                     "-thread-";
    }

    public Thread newThread(Runnable r) {
        Thread t = new Thread(group, r,
                              namePrefix + threadNumber.getAndIncrement(),
                              0);
        if (t.isDaemon())
            t.setDaemon(false);
        if (t.getPriority() != Thread.NORM_PRIORITY)
            t.setPriority(Thread.NORM_PRIORITY);
        return t;
    }
}
```
`DefaultThreadFactory`创建的线程的名字采用下面的格式
> pool-[线程池编号]-thread-[该线程池的线程编号]

然后优先级为`Thread.NORM_PRIORITY`。

### PrivilegedThreadFactory
在`Executors`中，还定义了一个`DefaultThreadFactory`的子类`PrivilegedThreadFactory`，用于返回用于创建新线程的线程工厂，这些新线程与**当前线程**x具有相同的权限。


```java
/**
 * Thread factory capturing access control context and class loader
 */
static class PrivilegedThreadFactory extends DefaultThreadFactory {
    private final AccessControlContext acc;
    private final ClassLoader ccl;

    PrivilegedThreadFactory() {
        super();
        SecurityManager sm = System.getSecurityManager();
        if (sm != null) {
            // Calls to getContextClassLoader from this class
            // never trigger a security check, but we check
            // whether our callers have this permission anyways.
            sm.checkPermission(SecurityConstants.GET_CLASSLOADER_PERMISSION);

            // Fail fast
            sm.checkPermission(new RuntimePermission("setContextClassLoader"));
        }
        this.acc = AccessController.getContext();
        this.ccl = Thread.currentThread().getContextClassLoader();
    }

    public Thread newThread(final Runnable r) {
        return super.newThread(new Runnable() {
            public void run() {
                AccessController.doPrivileged(new PrivilegedAction<Void>() {
                    public Void run() {
                        Thread.currentThread().setContextClassLoader(ccl);
                        r.run();
                        return null;
                    }
                }, acc);
            }
        });
    }
}
```
从源码看出，`PrivilegedThreadFactory extends DefaultThreadFactory`从而具有与 `defaultThreadFactory()` 相同设置的线程。但增加了两个特性：`ClassLoader`和`AccessControlContext`，从而使运行在此类线程中的任务具有与当前线程相同的访问控制和类加载器。

## 关于Callable的支持
### callable
重载了多个方法
```java
// 返回 Callable 对象，调用它时可运行给定的任务并返回给定的结果。callable(task)等价于callable(task, null)。
public static <T> Callable<T> callable(Runnable task, T result) {
    if (task == null)
        throw new NullPointerException();
    return new RunnableAdapter<T>(task, result);
}

// RunnableAdapter类
/**
 * A callable that runs given task and returns given result
 */
static final class RunnableAdapter<T> implements Callable<T> {
    final Runnable task;
    final T result;
    RunnableAdapter(Runnable  task, T result) {
        this.task = task;
        this.result = result;
    }
    public T call() {
        task.run();
        return result;
    }
}
```

```java
// 返回 Callable 对象，调用它时可运行给定特权的操作并返回其结果。
// callable(PrivilegedExceptionAction action)和其类似，唯一的区别在于，前者可抛出异常。
public static Callable<Object> callable(final PrivilegedAction<?> action) {
    if (action == null)
        throw new NullPointerException();
    return new Callable<Object>() {
    public Object call() { return action.run(); }};
}

//PrivilegedAction接口
public interface PrivilegedAction<T> {
    T run();
}
```

### privilegedCallable
```Java
// 返回 Callable 对象，调用它时可在当前的访问控制上下文中执行给定的 callable 对象。
Callable privilegedCallable(Callable callable)
```
### privilegedCallableUsingCurrentClassLoader
```java
// 返回 Callable 对象，调用它时可在当前的访问控制上下文中，使用当前上下文类加载器作为上下文类加载器来执行给定的 callable 对象。
Callable privilegedCallableUsingCurrentClassLoader(Callable callable)
```



参考
* [对线程池简单理解](https://www.cnblogs.com/ScarecrowAnBird/p/6801975.html)
* [再聊线程池](http://blog.csdn.net/ghsau/article/details/53538303)
* [多线程之线程池newFixedThreadPool（二）](http://blog.csdn.net/zknxx/article/details/53057683)
* [java定时任务接口ScheduledExecutorService](https://www.cnblogs.com/chenmo-xpw/p/5555931.html)
* [ThreadPoolExecutor线程池参数设置技巧](https://www.cnblogs.com/waytobestcoder/p/5323130.html)
* [理解ThreadPoolExecutor源码(一)线程池的corePoolSize、maximumPoolSize和poolSize](http://blog.csdn.net/aitangyong/article/details/38822505)
* [深入理解java线程池—ThreadPoolExecutor](https://www.jianshu.com/p/ade771d2c9c0)
* [ExecutorService引发的血案（二）ExecutorService使用](http://blog.csdn.net/u010940300/article/details/50251841)
* [Java线程池 ExecutorService](http://blog.csdn.net/suifeng3051/article/details/49443835)
* [Java线程池ExecutorService](https://www.cnblogs.com/Steven0805/p/6393443.html)
* [Java并发包学习三ThreadFactory介绍](http://blog.csdn.net/hechurui/article/details/49508357)
* [ Java线程(五)：Executors、ThreadFactory](http://blog.csdn.net/a355586533/article/details/78427871)
