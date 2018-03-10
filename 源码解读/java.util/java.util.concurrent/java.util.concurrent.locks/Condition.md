# Condition

`Conditon`用来是实现进程的同步。在1.5之前，我们使用的是`Object`的`wait`、`notify`和`notifyAll`方法，配合`synchronied`关键字来进行线程的同步。

而`Conditon`接口也提供了类似于Object监视器的方法，通过与Lock来实现等待-通知模式。


他们两者的比较如下表：

![Object监视器和Condition比较](http://ovn0i3kdg.bkt.clouddn.com/Object%E7%9B%91%E8%A7%86%E5%99%A8%E5%92%8CCondition%E5%AF%B9%E6%AF%94.png)



![Conditon](http://ovn0i3kdg.bkt.clouddn.com/Condition.png)

API说明如下：

| 方法 | 说明 |
| :------------- | :------------- |
| void await() throws InterruptedException |当前线程进入等待状态，直到被通知（signal）或者被中断时，当前线程进入运行状态，从await()返回；       |
|void awaitUninterruptibly()   |  当前线程进入等待状态，直到被通知，对中断不做响应；|
|long awaitNanos(long nanosTimeout) throws InterruptedException  |   在接口1的返回条件基础上增加了超时响应，返回值表示当前剩余的时间，如果在nanosTimeout之前被唤醒，返回值 = nanosTimeout - 实际消耗的时间，返回值 <= 0表示超时；|
|boolean await(long time, TimeUnit unit) throws InterruptedException   |  同样是在接口1的返回条件基础上增加了超时响应，与接口3不同的是：①可以自定义超时时间单位；②返回值返回true/false，在time之前被唤醒，返回true，超时返回false。|Â
|  boolean awaitUntil(Date deadline) throws InterruptedException|  当前线程进入等待状态直到将来的指定时间被通知，如果没有到指定时间被通知返回true，否则，到达指定时间，返回false； |
|   void signal()|  唤醒一个等待在Condition上的线程； |
|void signalAll()   |   唤醒等待在Condition上所有的线程。|

下面是源码注释中给出的一个例子：
```Java

class BoundedBuffer {
   final Lock lock = new ReentrantLock();
   final Condition notFull  = lock.newCondition();
   final Condition notEmpty = lock.newCondition();

   final Object[] items = new Object[100];
   int putptr, takeptr, count;

   public void put(Object x) throws InterruptedException {
     lock.lock();
     try {
       while (count == items.length)
         notFull.await();
       items[putptr] = x;
       if (++putptr == items.length) putptr = 0;
       ++count;
       notEmpty.signal();
     } finally {
       lock.unlock();
     }
   }

   public Object take() throws InterruptedException {
     lock.lock();
     try {
       while (count == 0)
         notEmpty.await();
       Object x = items[takeptr];
       if (++takeptr == items.length) takeptr = 0;
       --count;
       notFull.signal();
       return x;
     } finally {
       lock.unlock();
     }
   }
 }
```
这里定义了一个缓冲区类`Buffer`，缓冲区为空的时候读操作要收到阻塞，缓冲区为满的是的时候写操作收到阻塞。上面代码中分别定义了`notFull`和`notEmpty`来控制阻塞和唤醒。

类似的，我们可以利用`Conditon`来创建一个有界队列，当队列为空的时候出队操作阻塞，当队列为满的时候入队操作组设。（其实和上面是一个意思，换汤不换药）



![有界队列](https://upload-images.jianshu.io/upload_images/3994601-30a0a3470435c2ad.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)



`AQS`的内部类`ConditionObject`是`Condition`的实现类。
