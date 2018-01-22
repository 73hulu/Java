# FutureTask

`FutureTask`是最常用的`Future`的实现类，其主要的功能就是`Future`的功能，这里不多说。

需要特别注意的是，`FutureTask`可以用来包装`Callable`和`Runnable`对象，并且由于`FutureTask`实现了`Runnable`接口，所以它可以提交给`Executor`执行。

该类的结构如下：

![FutureTask](http://ovn0i3kdg.bkt.clouddn.com/FutureTask.png?imageView/2/w/500)


### public class FutureTask<V> implements RunnableFuture<V>
类声明。前面说到`FutureTask`实现了`Runnable`接口，实际上，并不是直接实现的，是间接实现的。该类直接实现`RunnableFuture`接口，而该接口继承了`Runnable`和`Future`接口：
```java
public interface RunnableFuture<V> extends Runnable, Future<V> {
    /**
     * Sets this Future to the result of its computation
     * unless it has been cancelled.
     */
    void run();
}
```
所以，`FutureTask`需要实现run方法。

### 构造函数
重载了两个构造函数。
### public FutureTask(Callable<V> callable){...}
接受一个`Callable`类型的参数，这里是对callable做了一层包装。定义如下：
```java
public FutureTask(Callable<V> callable) {
    if (callable == null)
        throw new NullPointerException();
    this.callable = callable;
    this.state = NEW;       // ensure visibility of callable
}
/** The underlying callable; nulled out after running */
private Callable<V> callable;

private volatile int state;
```
可以看到，`FutureTask`保存了这个任务，然后将`state`实例变量为`NEW`。`state`变量用来表示任务的状态，有以下几种取值：
```java
private static final int NEW          = 0; // 新建
private static final int COMPLETING   = 1; //计算中
private static final int NORMAL       = 2; //计算成功
private static final int EXCEPTIONAL  = 3; //抛出异常
private static final int CANCELLED    = 4; //被取消
private static final int INTERRUPTING = 5; //正在被中断
private static final int INTERRUPTED  = 6; //已经被中断
```
那么任务的状态变化过程可能存在以下几种情况：
```java
 NEW -> COMPLETING -> NORMAL
 NEW -> COMPLETING -> EXCEPTIONAL
 NEW -> CANCELLED
 NEW -> INTERRUPTING -> INTERRUPTED
```

### public FutureTask(Runnable runnable, V result) {..}
这里接受`Runnable`类型参数和预定的结果的类型。诶，不是说`FutureTask`会接受任务的执行结果返回值么？但是`Runnable`的run方法没有返回值啊？没错，但是`Executor`框架中的工具类`Executors`提供了一种转化方法，可以将`Runnable`类型对象转为`Callable`类型对象，该构造函数定义如下：
```java
public FutureTask(Runnable runnable, V result) {
    this.callable = Executors.callable(runnable, result);
    this.state = NEW;       // ensure visibility of callable
}
```
> java中有很多有"-s"和没有"-s"的非常相似的类，一般来讲，有"-s"的都是没有"-s"的工具类，比如`Objects -  Object`、`Arrays - Array` 、`Collections - Collection`和`Executors -  Executor`。

### public boolean cancel(boolean mayInterruptIfRunning){...}
用来取消任务，正如下`Future`接口中所说一样，如果在任务开始前就调用该方法，那么任务根本不会执行。如果任务已经结束，那么方法将返回false。否则将由`mayInterruptIfRunning`参数控制是否取消任务，即true表示取消任务，而false表示不取消。一般来说，调用这个方法都是为了取消任务的吧，谁那么无聊用false作为参数，虚晃一枪。该方法定义如下：
```java
public boolean cancel(boolean mayInterruptIfRunning) {
   if (!(state == NEW &&
         UNSAFE.compareAndSwapInt(this, stateOffset, NEW,
             mayInterruptIfRunning ? INTERRUPTING : CANCELLED)))
       return false;
   try {    // in case call to interrupt throws exception
       if (mayInterruptIfRunning) {
           try {
               Thread t = runner;
               if (t != null)
                   t.interrupt();
           } finally { // final state
               UNSAFE.putOrderedInt(this, stateOffset, INTERRUPTED);
           }
       }
   } finally {
       finishCompletion();
   }
   return true;
}
```
方法结构比较清晰，在finally中调用的`finishCompletion`方法定义如下：
```java
private void finishCompletion() {
    // assert state > COMPLETING;
    for (WaitNode q; (q = waiters) != null;) {
        if (UNSAFE.compareAndSwapObject(this, waitersOffset, q, null)) {
            for (;;) {
                Thread t = q.thread;
                if (t != null) {
                    q.thread = null;
                    LockSupport.unpark(t);
                }
                WaitNode next = q.next;
                if (next == null)
                    break;
                q.next = null; // unlink to help gc
                q = next;
            }
            break;
        }
    }

    done();

    callable = null;        // to reduce footprint
}
```
额，没看懂。

### public boolean isCancelled() {...}
判断任务是否被取消。定义如下：
```java
public boolean isCancelled() {
    return state >= CANCELLED;
}
```
注意，这里判断当前任务状态是否大于`CANCELLED`，对照前面的状态常数，如果任务的状态正处于`CANCELLED`、`INTERRUPTING`或`INTERRUPTED`，则方法返回true。原来这个状态常量的设计并不是随便写的。

###  public boolean isDone() {..}
我理解的`isDone`含义是检查任务是否已经完成，这种完成不定是`NORMAL`，可以是除了`NEW`之外的其他状态。源码定义如下：
```java
public boolean isDone() {
    return state != NEW;
}
```

### get方法
重载了另个`get`方法，区别在于一个会一直等下去或者当前线程在等待状态被中断，而另一个在有限时间内等待，超时了就抛出异常，道理是一样的，我们只需要看前一种计即可。
```java
public V get() throws InterruptedException, ExecutionException {
    int s = state;
    if (s <= COMPLETING)
        s = awaitDone(false, 0L);
    return report(s);
}
```
只有当任务处于`NEW`或者`COMPLETING`状态，`get`方法才会尝试进行获取结果，这种尝试依靠`awaitDone`方法实现，定义如下：
```java
private int awaitDone(boolean timed, long nanos)
    throws InterruptedException {
    final long deadline = timed ? System.nanoTime() + nanos : 0L;
    WaitNode q = null;
    boolean queued = false;
    for (;;) {
        if (Thread.interrupted()) {
            removeWaiter(q);
            throw new InterruptedException();
        }

        int s = state;
        if (s > COMPLETING) {
            if (q != null)
                q.thread = null;
            return s;
        }
        else if (s == COMPLETING) // cannot time out yet
            Thread.yield();
        else if (q == null)
            q = new WaitNode();
        else if (!queued)
            queued = UNSAFE.compareAndSwapObject(this, waitersOffset,
                                                 q.next = waiters, q);
        else if (timed) {
            nanos = deadline - System.nanoTime();
            if (nanos <= 0L) {
                removeWaiter(q);
                return state;
            }
            LockSupport.parkNanos(this, nanos);
        }
        else
            LockSupport.park(this);
    }
}
```
其核心还是自旋。如果当前线程被中断了，则退出等待，并抛出异常。剩下的...有点看不懂了。

###  public void run() {}
`FutureTask`间接实现了`Runnable`接口，所以必须是实现`run`方法。定义如下：
```java
public void run() {
    if (state != NEW ||
        !UNSAFE.compareAndSwapObject(this, runnerOffset,
                                     null, Thread.currentThread()))
        return;
    try {
        Callable<V> c = callable;
        if (c != null && state == NEW) {
            V result;
            boolean ran;
            try {
                result = c.call();
                ran = true;
            } catch (Throwable ex) {
                result = null;
                ran = false;
                setException(ex);
            }
            if (ran)
                set(result);
        }
    } finally {
        // runner must be non-null until state is settled to
        // prevent concurrent calls to run()
        runner = null;
        // state must be re-read after nulling runner to prevent
        // leaked interrupts
        int s = state;
        if (s >= INTERRUPTING)
            handlePossibleCancellationInterrupt(s);
    }
}
```
本质上执行的仍旧是`Callable`的实例方法的`call`方法。

### 使用`FutureTask`
利用`FutureTask`来获取任务的执行结果的基本流程是： 定义一个 `Callable`或`Runnable`实例（一般用匿名类实现），以此实例构造`FutureTask`的实例（注意不是`Future`的实例），然后以此实例构造`Thread`实例，调用`Thread`对象的`start`方法。

下面是一个`FutureTask`的最基础的使用方法：
```java
public class CallableAndFuture {
    public static void main(String[] args) {
        Callable callable = new Callable<Integer>() {
            @Override
            public Integer call() throws Exception {
                return new Random().nextInt();
            }
        };

        FutureTask<Integer> future = new FutureTask<Integer>(callable); //注意这里future不能声明为`Future<Integer>`,虽然这句话不会报错，但是下一句话`new Thread(future)`会编译报错，因为`Future`接口本身没有继承`Runnable`接口，只有`FutureTask`实现了`Runnable`接口
        new Thread(future).start();

        try{
            Thread.sleep(5000); //主线程休眠，让任务执行线程有足够的时间执行任务
            System.out.println(future.get());
        }catch (InterruptedException e){

        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }
}
```

程序执行将打印出一个任意的整数。

以上方法是我们手动创建一个线程用来执行任务。在JDK1.5之后，创建线程的办法更加多样化，我们可以将任务执行交给线程池来完成。
```java
public class CallableAndFutureTest {

    public static void main(String[] args) {
        ExecutorService threadPoll = Executors.newSingleThreadExecutor();

        Future<Integer> future = threadPoll.submit(new Callable<Integer>() { // 这里可以将future声明为Future
            @Override
            public Integer call() throws Exception {
                return new Random().nextInt();
            }
        });

        try {
            Thread.sleep(5000);
            System.out.println(future.get());
        }catch (InterruptedException e){
            e.printStackTrace();
        }catch (Exception e){
            e.printStackTrace();
        }
    }
}
```
这样看着代码也简单了很多，我们无需取手动触发`start`，这样不用自己去管理线程的生命周期，非常方面，在1.5之后，线程池继续是线程操作的首选。

如果有多个执行结果，可以采取以下方法：
```java
public class CallableAndFutureTest1 {
    public static void main(String[] args) {
        ExecutorService threadPool = Executors.newSingleThreadExecutor();
        CompletionService<Integer> cs = new ExecutorCompletionService<Integer>(threadPool);
        for(int i = 1; i < 5; i++) {
            final int taskID = i;
            cs.submit(new Callable<Integer>() {
                public Integer call() throws Exception {
                    return taskID;
                }
            });
        }
        // 可能做一些事情
        for(int i = 1; i < 5; i++) {
            try {
                System.out.println(cs.take().get());
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (ExecutionException e) {
                e.printStackTrace();
            }
        }
    }
}  
```

参考
* [ Java线程(七)：Callable和Future](http://blog.csdn.net/ghsau/article/details/7451464)
