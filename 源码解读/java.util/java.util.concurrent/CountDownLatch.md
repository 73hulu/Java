# CountDownLatch

`CountDown`的意思是“倒计时”，“latch”的意思是"门栓"。所以`CountDownLatch`的意思是“倒计时后占有”？实际上，这个类的用法确实如此。它是一个同步辅助类，允许一个或多个线程等待，直到一组在其它线程中的操作执行完成。什么意思呢？比如有一个任务A，它要等待其他4个任务执行完毕之后才能执行，此时就可以利用CountDownLatch来实现这种功能了。


该类的定义如下：

![CountDownLatch](http://ovn0i3kdg.bkt.clouddn.com/CountDownLatch.png)

又在里面见到了熟悉的老朋友`Sync`


## public class CountDownLatch
类声明，非常干净。这可不是一个锁。

> 对于线程的理解总能感觉乱糟糟的，总觉得什么都是锁，线程就是通过各种各样的锁实现的。这种想法是不对的。

## private static final class Sync extends AbstractQueuedSynchronizer {..}
又是基于AQS的实现，定义如下：
```Java
/**
 * Synchronization control For CountDownLatch.
 * Uses AQS state to represent count.
 */
private static final class Sync extends AbstractQueuedSynchronizer {
    private static final long serialVersionUID = 4982264981922014374L;

    Sync(int count) {
        setState(count);
    }

    int getCount() {
        return getState();
    }

    protected int tryAcquireShared(int acquires) {
        return (getState() == 0) ? 1 : -1;
    }

    protected boolean tryReleaseShared(int releases) {
        // Decrement count; signal when transition to zero
        for (;;) {
            int c = getState();
            if (c == 0)
                return false;
            int nextc = c-1;
            if (compareAndSetState(c, nextc))
                return nextc == 0;
        }
    }
}
```
可以看到，我们仍旧可以指定资源state的数量。

## 构造方法
只有一个构造方法：
```Java
public CountDownLatch(int count) {
    if (count < 0) throw new IllegalArgumentException("count < 0");
    this.sync = new Sync(count);
}
```
其中count就是倒计时的值，从本质来讲，这个值执行的是AQS中state的值。

## await方法
`await`方法是`CountDownLatch`中最重要的方法之一了，它的目的在于调用await()方法的线程会被挂起，直到条件满足了才能继续进行。满足什么条件呢？`await`有两个重载，这两个方法继续进行的满足条件是不一样的。

### public void await() throws InterruptedException{..}
无参数的`await`方法。定义如下：
```Java
public void await() throws InterruptedException {
    sync.acquireSharedInterruptibly(1);
}
```
调用这个方法的线程将被挂起，知道count的值减到0才能继续。

### public boolean await(long timeout, TimeUnit unit) throws InterruptedException {..}
两个参数，一个是指时间，另一个是指时间单位。它与无参的`await`方法作用类似，不同的是，当count到达0之后，它还需要再等待一段时间才能继续。
```Java
public boolean await(long timeout, TimeUnit unit)
        throws InterruptedException {
    return sync.tryAcquireSharedNanos(1, unit.toNanos(timeout));
}
```
## public void countDown(){..}
`CountDownLatch`另外一个重要的方法，其作用是倒计时，即count减去1。定义如下：
```Java
public void countDown() {
    sync.releaseShared(1);
}
```

## public long getCount(){..}
取得当前count的大小，很简单：
```Java
public long getCount() {
    return sync.getCount();
}
```

##  public String toString(){..}
返回的有用信息仍旧是当前的count量。定义如下：
```Java
public String toString() {
    return super.toString() + "[Count = " + sync.getCount() + "]";
}
```

## 使用场景
`CountDownLatch`常常被使用到的场景：线程A需要等到其他几个线程完成之后才能进行，如果需要等待的线程的个数为n，那么就创建一个count=n的计数器，线程A需要等待，其他几个线程每完成一个线程后就将倒计时，等所有的线程都完成之后线程A继续。下面是一个使用示例：
```Java
public class Test {
     public static void main(String[] args) {   
         final CountDownLatch latch = new CountDownLatch(2);

         new Thread(){
             public void run() {
                 try {
                     System.out.println("子线程"+Thread.currentThread().getName()+"正在执行");
                    Thread.sleep(3000);
                    System.out.println("子线程"+Thread.currentThread().getName()+"执行完毕");
                    latch.countDown();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
             };
         }.start();

         new Thread(){
             public void run() {
                 try {
                     System.out.println("子线程"+Thread.currentThread().getName()+"正在执行");
                     Thread.sleep(3000);
                     System.out.println("子线程"+Thread.currentThread().getName()+"执行完毕");
                     latch.countDown();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
             };
         }.start();

         try {
             System.out.println("等待2个子线程执行完毕...");
            latch.await();
            System.out.println("2个子线程已经执行完毕");
            System.out.println("继续执行主线程");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
     }
}
```
程序输出结果是：
```Java
线程Thread-0正在执行
线程Thread-1正在执行
等待2个子线程执行完毕...
线程Thread-0执行完毕
线程Thread-1执行完毕
2个子线程已经执行完毕
继续执行主线程
```
