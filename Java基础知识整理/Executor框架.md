# Executor框架

`Executor`是Java提供的有关于线程的框架，在JDK1.5中被提出。在此之前，线程即是工作单元也是执行机制。而`Executor`框架将线程的工作单元和执行机制相互分离，其中`Runnable`和`Callable`是工作单元，而`Executor`框架则提供了线程的执行机制。

`Executor`框架主要包含三个部分

| 部分 | 内容 |
| :------------- | :------------- |
| 任务 | 包括`Runnable`和`Callable`，其中`Runnable`可以表示一个异步执行的任务，而`Callable`表示一个会产生结果的任务 |
|任务的执行   |包括核心接口`Executor`及其子接口`ExecutorService`，有两个类——`ThreadPoolExecutor`和`ScheduledThreadPollExecutor`实现了`ExecutorService`接口  |
|异步计算的结果   |  包括接口`Future`及其实现了`FutureTask`|

## 为什么线程池

首先我们要弄清楚为什么要用线程池？

之前我们用继承`Thread`类和继承`Runnable`接口的方法来创建线程，完全能够运作起来，没有什么不妥。但是试想这样一种情况：许多服务器应用程序都面向处理来自远程来源的多而小的任务，应用程序需要为每一个任务都创建一个线程，然后让线程执行完程序之后，再进行销毁，如此反复，导致造成这样一种局面：创建和销毁线程带来的开销远大于处理业务的开销。另外，只要线程活动着，它就会消耗资源。所以，如果系统创建了过多的线程，在一个JVM中创建了太多的线程将可能导致系统由于过度消耗内存而用完内存或“切换过度”。为了防止资源不足，应用程序需要一些办法来限制任何给定时刻处理的请求数目。

基于以上的考虑，线程池应运而生。它的基本原理是**线程的重用**：当请求到达时，由于线程已经存在，所以也就不用再创建线程，也不用等创建成功后再进行操作，既节省了资源空间，也提高了响应速度。另外，线程池的线程数目是有上线的，当一个任务到来但是已无线程可用的时候，需要强制使新到的任务等待，一直等到其获取线程为止，这样就可以避免资源不足的问题。（说到这里，想到了Semaphore的应用）

总的来说，以下原因促使线程池出现：
1. 创建/销毁线程伴随着系统的开销，过于频繁的创建/销毁线程将影响处理效率。
2. 线程并发量过多，抢占系统资源从而导致阻塞
3. 对线程进行一些简单的管理，比如延迟执行，定时循环执行等。

Java在JUC中提供了`Executor`框架，框架中重要类的继承树如下：


![Executor框架](http://img.blog.csdn.net/20151027091552190)

其实这其中重要的一个是`Executors`工厂类。

## Executor框架
### Executor接口
`Executor`接口是`Executor`框架最重要的根本的接口，里面定义了`execute`方法，用来执行任务。

### ExecutorService接口
`ExecutorService`直接继承了`Executor`接口，另外还提供了任务的管理办法，比如`shutdownNow`来立即停止一切任务，`shutdown`来停止接受新任务，并等待之前提交的任务完成，`submit`方法用来提交任务，注意该方法的返回值是异步结果`Future`。

### ThreadPoolExecutor
这个类非常非常非常重要，可以说是Executor框架的核心。它定义一个线程的处理流程，具体如下：

![ThreadPoolExecutor具体工作流](https://upload-images.jianshu.io/upload_images/2177145-33c7b5ff75cf2bf7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

我们需要熟悉几个参数的含义：
* `corePoolSize`： 核心线程的数量
* `maxPoolSize`：线程池的最大数量
* `keepAliveTime`： 线程空闲时间数量
* `workQueue`：阻塞队列
* `handler`：拒绝策略，包括四种(AbortPolicy|DiscardPolicy|DiscardOldestPolicy|CallerRunsPolicy)，默认AbortPolicy。

上图中的流程可以这样解释：
当一个任务被执行后，它首先需要判断当前线程数量是否小于`corePoolSize`，如果是，则新建线程运行，如果不是，则考虑放到`workQueue`中。如果此时`workQueue`还有空间，那么就放到阻塞队列中，如果已经满了，那么就看当前线程数量是否小于`maxPoolSize`，如果是，则创建线程，如果不是，则使用拒绝策略。

这就是`ThreadPoolExecutor`这个类最重要的地方。一般情况下，我们使用的是`Executors`工厂类为我们创建的四种线程池。而他们本质上，是使用了不同的参数来创建的`ThreadPoolExecutor`。不同的参数导致了不同类型的线程池。

### Executors
`Executors`是`Executor`框架中的工具类，它提供了很多线程操作的支持。具体如下：
#### 对于ExecutorService的支持
`Executors`中提供了不同的线程池的获取，

|方法名称 | newFixedThreadPool |  newCachedThreadPool| newSingleThreadExecutor|
| :------------- | :------------- | :--|
|线程池特点 |   固定线程数量，无界队列  | 可缓存线程池，| 线程池固定大小为1，无界队列|
|  处理方式 | 每次提交一个任务就创建一个线程，直到线程达到线程池的最大大小。线程池的大小一旦达到最大值就会保持不变，如果某个线程因为执行异常而结束，那么线程池会补充一个新线程。  | 如果线程池的大小超过了处理任务所需要的线程，那么就会回收部分空闲（60秒不执行任务）的线程，当任务数增加时，此线程池又可以智能的添加新线程来处理任务。不会对线程池大小做限制，线程池大小完全依赖于操作系统（或者说JVM）能够创建的最大线程大小。  |  这个线程池只有一个线程在工作，也就是相当于单线程串行执行所有任务。如果这个唯一的线程因为异常结束，那么会有一个新的线程来替代它。此线程池保证所有任务的执行顺序按照任务的提交顺序执行。 |
|corePoolSize   |  自定义 | 0| 1|
|maximumPoolSize   |  自定义，与corePoolSize保持一致 |  Integer.MAX_VALUE| 1|
|keepAliveTime   |  0 |  60分钟| 0|
|workQueue类型   |   LinkedBlockingQueue| SynchronousQueue|  LinkedBlockingQueue|
|threadFactory   | DefaultThreadFactory  |  DefaultThreadFactory |  DefaultThreadFactory |
|handler   | AbortPolicy  |AbortPolicy| AbortPolicy|
|备注   |   |   |  使用`FinalizableDelegatedExecutorService`对`ThreadPoolExecutor`进行包装 |
|优点   | 保证新任务不会被拒绝  |   |  保证新任务不会被拒绝  |
|缺点   |  当任务处理无线等待的是会造成内存问题 |   | 当任务处理无线等待的是会造成内存问题  |
|适用于   |   |     |逻辑上需要单线程处理任务的场景  |


#### 对于ThreadFactory的支持
主要定义了`DefaultThreadFactory`和`PrivilegedThreadFactory`用于返回创建新线程的线程工厂。前者是默认定义，主要是约定了线程名字为`pool-[线程池编号]-thread-[该线程池的线程编号]`和线程优先级固定为`Thread.NORM_PRIORIT`。后者继承前者，约定线程的优先级与当前线程具有同样的优先级。

#### 对于Callable的支持
特别注意的是`callable(Runnable task, T result)`方法，它用`Runable`类型的对象作为参数，但是内部转成了`callable`并返回结果，其中的原因是使用了适配器`RunnableAdapter`。
定义如下：
```JAVA

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

## 使用Executors来创建线程池
官方建议使用`Executors`来创建线程池，但是最近阿里发布的Java开发手册中强调线程池的建立禁止使用`Executors`，而是通过 `ThreadPoolExecutor `的方式，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险。

这里，我们还是需要知道`Executors`如何创建线程池，并运行线程。下面是一个用`newCachedThreadPool`l来创建无界限线程池的例子：
```JAVA
public class FixedThreadPoolTest {
    public static void main(String[] args) {
        ExecutorService executor = Executors.newCachedThreadPool();

        executor.execute(new Runnable() {
            @Override
            public void run() {
                try {
                    System.out.println("Runnable1 begin : " + System.currentTimeMillis());

                    Thread.sleep(10000);

                    System.out.println("A");
                    System.out.println("Runnable1 end:" + System.currentTimeMillis());

                }catch (InterruptedException e){
                    e.printStackTrace();
                }
            }
        });

        executor.execute(new Runnable() {
            @Override
            public void run() {
                try {
                    System.out.println("Runnable2 begin : " + System.currentTimeMillis());

                    Thread.sleep(10000);

                    System.out.println("A");
                    System.out.println("Runnable2 end:" + System.currentTimeMillis());

                }catch (InterruptedException e){
                    e.printStackTrace();
                }
            }
        });
    }
}
```
执行效果如下：
```
Runnable1 begin : 1520171243755
Runnable2 begin : 1520171243756
A
Runnable1 end:1520171253758
A
Runnable2 end:1520171253760
```

下面的代码是一个程序复用的例子：
```JAVA
public class MyRunnable implements Runnable {
    private String username;
    public MyRunnable(String username) {
        super();
        this.username = username;
    }
    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + " username=" + username + " begin:" + System.currentTimeMillis());
        System.out.println(Thread.currentThread().getName() + " username=" + username + " end:" + System.currentTimeMillis());
    }
}

public class Main {
    public static void main(String[] args) throws InterruptedException {
        ExecutorService executorService = Executors.newCachedThreadPool();
        for (int i = 0; i < 5; i++) {
            executorService.execute(new MyRunnable("" + i));
        }
        Thread.sleep(2000);
        System.out.println();
        for (int i = 0; i < 5; i++) {
            executorService.execute(new MyRunnable("" + i));
        }
    }
}
```
执行效果如下：
```
pool-1-thread-1 username=0 begin:1470229448635
pool-1-thread-4 username=3 begin:1470229448635
pool-1-thread-3 username=2 begin:1470229448635
pool-1-thread-2 username=1 begin:1470229448635
pool-1-thread-3 username=2 end:1470229448635
pool-1-thread-4 username=3 end:1470229448635
pool-1-thread-5 username=4 begin:1470229448635
pool-1-thread-1 username=0 end:1470229448635
pool-1-thread-5 username=4 end:1470229448636
pool-1-thread-2 username=1 end:1470229448635

pool-1-thread-1 username=2 begin:1470229450637
pool-1-thread-3 username=4 begin:1470229450637
pool-1-thread-3 username=4 end:1470229450637
pool-1-thread-2 username=0 begin:1470229450637
pool-1-thread-2 username=0 end:1470229450638
pool-1-thread-5 username=1 begin:1470229450637
pool-1-thread-4 username=3 begin:1470229450637
pool-1-thread-4 username=3 end:1470229450638
pool-1-thread-5 username=1 end:1470229450638
pool-1-thread-1 username=2 end:1470229450637
```
由打印结果可见，第一次for循环中创建了5个线程对象分别是pool-1-thread-1到pool-1-thread-5，第二次for循环中没有创建新的线程对象，复用了第一次for循环中创建的线程对象。

下面是一个使用自定义线程工厂创建无界线程池的例子：
```JAVA
public class MyThreadFactory implements ThreadFactory {
    @Override
    public Thread newThread(Runnable r) {
        Thread thread = new Thread(r);
        thread.setName("定制池中线程对象的名称" + Math.random());
        return thread;
    }
}

public class Run {
    public static void main(String[] args) {
        MyThreadFactory myThreadFactory = new MyThreadFactory();
        ExecutorService executorService = Executors.newCachedThreadPool(myThreadFactory);
        executorService.execute(new Runnable() {

            @Override
            public void run() {
                System.out.println(Thread.currentThread().getName() + "运行：" + System.currentTimeMillis());
            }
        });
    }
}
```
程序执行结果如下：
```
定制池中线程对象的名称0.2671917944865071运行：1470230269473
```

下面使用`newFixedThreadPool(int)`方法创建有界线程池：
```JAVA
public class MyRunnable implements Runnable {
    String username;
    public MyRunnable(String username) {
        super();
        this.username = username;
    }
    @Override
    public void run() {
        try {
            System.out.println(Thread.currentThread().getName() + " username=" + username + " begin:" + System.currentTimeMillis());
            Thread.sleep(2000);
            System.out.println(Thread.currentThread().getName() + " username=" + username + " end:" + System.currentTimeMillis());
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

public class Run {
    public static void main(String[] args) {
        ExecutorService executorService = Executors.newFixedThreadPool(3);
        for (int i = 0; i < 3; i++) {
            executorService.execute(new MyRunnable("" + i));
        }
        for (int i = 0; i < 3; i++) {
            executorService.execute(new MyRunnable("" + i));
        }
    }
}
```
线程执行结果如下：
```JAVA
pool-1-thread-1 username=0 begin:1470230865037
pool-1-thread-3 username=2 begin:1470230865037
pool-1-thread-2 username=1 begin:1470230865037
pool-1-thread-3 username=2 end:1470230867043
pool-1-thread-1 username=0 end:1470230867042
pool-1-thread-3 username=0 begin:1470230867043
pool-1-thread-1 username=1 begin:1470230867043
pool-1-thread-2 username=1 end:1470230867043
pool-1-thread-2 username=2 begin:1470230867043
pool-1-thread-3 username=0 end:1470230869047
pool-1-thread-1 username=1 end:1470230869047
pool-1-thread-2 username=2 end:1470230869047
```
此时线程池中最多有三个线程。

下面是使用使用`newSingleThreadExecutor()`方法创建单一线程池
```JAVA
public class MyRunnable implements Runnable {
    String username;
    public MyRunnable(String username) {
        super();
        this.username = username;
    }
    @Override
    public void run() {
        try {
            System.out.println(Thread.currentThread().getName() + " username=" + username + " begin:" + System.currentTimeMillis());
            Thread.sleep(2000);
            System.out.println(Thread.currentThread().getName() + " username=" + username + " end:" + System.currentTimeMillis());
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

public class Run {
    public static void main(String[] args) {
        ExecutorService executorService = Executors.newSingleThreadExecutor();
        for (int i = 0; i < 3; i++) {
            executorService.execute(new MyRunnable("" + i));
        }
    }
}
```
程序执行结果如下：
```JAVA
pool-1-thread-1 username=0 begin:1470231470978
pool-1-thread-1 username=0 end:1470231472978
pool-1-thread-1 username=1 begin:1470231472978
pool-1-thread-1 username=1 end:1470231474982
pool-1-thread-1 username=2 begin:1470231474982
pool-1-thread-1 username=2 end:1470231476984
```
可以看到，此时线程池中只有一个线程。



参考
* [Java并发编程核心方法与框架-Executors的使用](http://blog.csdn.net/umgsai/article/details/54288405)
