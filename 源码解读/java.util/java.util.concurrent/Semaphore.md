# Semaphore

"Semaphore"翻译过来是“信号量”的意思。

“信号量”、“互斥”、“同步”...这些名词在并发编程中经常听到，它们之间有什么联系和区别呢？
## 一些概念的区别
### 同步和互斥
首先是“互斥”和“同步”。

“互斥”是指控制同一个时刻只有一个线程可以对其进行操作，这种情况下，多个线程之间是互相排斥的。这种场景非常常见，比如10个售票窗口就有10个线程，而100张票是他们共同访问的资源，要控制每个时刻只能有一个线程能对其进行操作，这样才能保证不出现重复卖票的情况。互斥的实现是通过各种锁实现的，比如内置锁、1.5新加入的`ReenrantLock`

“同步”的概念是用于线程间的通信，只要是进行线程间的配合工作，最常见的情况就是“生产者-消费者”模式。多个生产者和多个消费者就是多条执行线程，他们共同操作一个数据结构中的数据，数据结构中有时是没有数据的，这个时候消费者应该处于等待状态而不是不断的去访问这个数据结构。这里就涉及到线程间通信（当然此处还涉及到互斥，这里暂不考虑这一点），消费者线程一次消费后发现数据结构空了，就应该处于等待状态，生产者生产数据后，就去唤醒消费者线程开始消费。生产者线程某次生产后发现数据结构已经满了，也应该处于等待状态，消费者消费一条数据后，就去唤醒生产者继续生产。而线程同步可以通过`Object`的`wait`、`notify`和`notifyAll`以及1.5之后的`Condition`来解决。

其实这两个概念我们经常放在一起，因为另种操作是经常同时进行，比如上面提到的生产者-消费者模式，既有对统一资源的互斥访问，也有对线程的同步控制。

### 信号量和互斥量
信号量用于线程的同步，而互斥量用于线程的互斥。其实说法也并非这么绝对，信号量同样可以用来进行线程的互斥。

互斥量是一个特殊的信号量，特殊之处在于它只有0和1的概念，一个线程要么进入临界区，要么不进入临界区，很干脆，互斥量在Java中的实现是`ReentrantLock`。

而信号量则是一个非负数，它允许一定数量的线程进入临界区。信号量在Java中的实现是`Semaphore`。

互斥量的加锁和解锁必须由同一个线程分别对应使用，而信号量则可以由一个线程释放，另一个线程得到。


下面是一个信号量的定义：
```Java
public class Semaphore{
  private boolean signal = false;

  public synchronied take(){
    this.signal = true;
    this.notifyAll();
  }

  public synchronized release() throws InterruptedException{
    while(! this.signal){ //while循环避免假唤醒
      wait();
    }
    this.signal = false;
  }
}
```


使用场景如下：
```Java
Semaphore semaphore = new Semaphore();
SendingThread sender = new SendingThread(semaphore)；
ReceivingThread receiver = new ReceivingThread(semaphore);
receiver.start();
sender.start();

public class SendingThread extends Thread{
    Semaphore semaphore = null;
    public SendingThread(Semaphore semaphore){
        this.semaphore = semaphore;
    }

    @Override
    public void run(){
        while(true){
            //do something, then signal
            this.semaphore.take();
        }
    }
}

public class RecevingThread {
    Semaphore semaphore = null;
    public ReceivingThread(Semaphore semaphore){
        this.semaphore = semaphore;
    }
    public void run(){
        while(true){
        this.semaphore.release();
        //receive signal, then do something...
        }
    }
}
```
老实说，这实际上是一个互斥量，因为它非0即1，并没有计数。而下面就是一个计数的信号量。

```Java
public class Semaphore{
  private int signal = 0;

  public synchronied void take(){
    this.signal ++;
    this.notifyAll();
  }

  public synchronized void release() throws InterruptedException{
    while(this.signals == 0)
        wait();
    this.signals--;
  }
}
```
以上的信号量是没有上界的，下面我们就创建一个有上界的信号量：
```Java
public class BoundedSemaphore {
    private int signals = 0;
    private int bound   = 0;
    public BoundedSemaphore(int upperBound){
        this.bound = upperBound;
    }
    public synchronized void take() throws InterruptedException{
        while(this.signals == bound)
            wait();
        this.signals++;
        this.notify();
    }
    public synchronized void release() throws InterruptedException{
        while(this.signals == 0)
            wait();
        this.signals--;
        this.notify();
    }
}
```
这里看到，我们在创建`BoundedSemaphore`类的时候，可以指定上界，如果信号量已经达到上界，那么执行`take`的线程就将中止。

> 实际上，上面不就是消费者-生产者模式么？

## Semaphore的实现
Java提供了内置的`Semaphore`类，其结构如下：

![Semaphore](http://ovn0i3kdg.bkt.clouddn.com/Semaphore.png)


可以看到，`Semaphore`也是AQS的实现。其中最重要的仍然是`Syn`的抽象内部类的定义及它的两个实现`NonfairSync`和`FairSync`。

下面是`Sync`的源码定义：
```Java
/**
 * Synchronization implementation for semaphore.  Uses AQS state
 * to represent permits. Subclassed into fair and nonfair
 * versions.
 */
abstract static class Sync extends AbstractQueuedSynchronizer {
    private static final long serialVersionUID = 1192457210091910933L;

    Sync(int permits) {
        setState(permits);
    }

    final int getPermits() {
        return getState();
    }

    final int nonfairTryAcquireShared(int acquires) {
        for (;;) {
            int available = getState();
            int remaining = available - acquires;
            if (remaining < 0 ||
                compareAndSetState(available, remaining))
                return remaining;
        }
    }

    protected final boolean tryReleaseShared(int releases) {
        for (;;) {
            int current = getState();
            int next = current + releases;
            if (next < current) // overflow
                throw new Error("Maximum permit count exceeded");
            if (compareAndSetState(current, next))
                return true;
        }
    }

    final void reducePermits(int reductions) {
        for (;;) {
            int current = getState();
            int next = current - reductions;
            if (next > current) // underflow
                throw new Error("Permit count underflow");
            if (compareAndSetState(current, next))
                return;
        }
    }

    final int drainPermits() {
        for (;;) {
            int current = getState();
            if (current == 0 || compareAndSetState(current, 0))
                return current;
        }
    }
}
```
在`Sync`构造函数中可以看到，它接受的的整形参数是加上就是AQS中资源的个数，而对于资源的获取，就是将资源的个数减去1，对于资源的释放，就是将资源的个数加上1。


> 其他代码暂时先不看了，总觉的AQS的基础没有打好，所以看这部分有点困难。

## 使用场景
`Semaphore`用来控制线程并发的个数，其中我们传入的参数就是允许并发的线程的个数。由于这种特性，经常用来限制获取某种资源的线程数量，比如数据库连接池中连个数的限制等。


下面是一个使用信号量来限制资源使用数量的例子：操场上有5个跑道，一个跑道一次只能有一个学生在上面跑步，一旦所有跑道在使用，那么后面的学生就需要等待，直到有一个学生不跑了。那么操场类的定义如下：
```Java
/**
 * 操场，有5个跑道
 * Created by Xingfeng on 2016-12-09.
 */
public class Playground {

    /**
     * 跑道类
     */
    static class Track {
        private int num;

        public Track(int num) {
            this.num = num;
        }

        @Override
        public String toString() {
            return "Track{" +
                    "num=" + num +
                    '}';
        }
    }

    private Track[] tracks = {
            new Track(1), new Track(2), new Track(3), new Track(4), new Track(5)};
    private boolean[] used = new boolean[5];

    private Semaphore semaphore = new Semaphore(5, true);

    /**
     * 获取一个跑道
     */
    public Track getTrack() throws InterruptedException {

        semaphore.acquire(1);
        return getNextAvailableTrack();
    }

    /**
     * 返回一个跑道
     *
     * @param track
     */
    public void releaseTrack(Track track) {
        if (makeAsUsed(track))
            semaphore.release(1);
    }

    /**
     * 遍历，找到一个没人用的跑道
     *
     * @return
     */
    private Track getNextAvailableTrack() {

        for (int i = 0; i < used.length; i++) {
            if (!used[i]) {
                used[i] = true;
                return tracks[i];
            }
        }

        return null;

    }
    /**
     * 返回一个跑道
     *
     * @param track
     */
    private boolean makeAsUsed(Track track) {

        for (int i = 0; i < used.length; i++) {
            if (tracks[i] == track) {
                if (used[i]) {
                    used[i] = false;
                    return true;
                } else {
                    return false;
                }

            }
        }

        return false;
    }
}
```

创建了5个跑道对象，并使用一个boolean类型的数组记录每个跑道是否被使用了，初始化了5个许可证的Semaphore，在获取跑道时首先调用acquire(1)获取一个许可证，在归还一个跑道是调用release(1)释放一个许可证。接下来再看启动程序，如下：
```Java
public class SemaphoreDemo {

    static class Student implements Runnable {

        private int num;
        private Playground playground;

        public Student(int num, Playground playground) {
            this.num = num;
            this.playground = playground;
        }

        @Override
        public void run() {

            try {
                //获取跑道
                Playground.Track track = playground.getTrack();
                if (track != null) {
                    System.out.println("学生" + num + "在" + track.toString() + "上跑步");
                    TimeUnit.SECONDS.sleep(2);
                    System.out.println("学生" + num + "释放" + track.toString());
                    //释放跑道
                    playground.releaseTrack(track);
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

        }
    }

    public static void main(String[] args) {

        Executor executor = Executors.newCachedThreadPool();
        Playground playground = new Playground();
        for (int i = 0; i < 100; i++) {
            executor.execute(new Student(i+1,playground));
        }

    }
}
```
Student类代表学生，首先获取跑道，一旦获取到就打印一句话，然后睡眠2s，然后再打印释放，最后归还跑道。

下面这个例子是源码中注释的给出的：
```Java
class Pool {
   private static final int MAX_AVAILABLE = 100;
   private final Semaphore available = new Semaphore(MAX_AVAILABLE, true);

   public Object getItem() throws InterruptedException {
     available.acquire();
     return getNextAvailableItem();
   }

   public void putItem(Object x) {
     if (markAsUnused(x))
       available.release();
   }

   // Not a particularly efficient data structure; just for demo

   protected Object[] items = ... whatever kinds of items being managed
   protected boolean[] used = new boolean[MAX_AVAILABLE];

   protected synchronized Object getNextAvailableItem() {
     for (int i = 0; i < MAX_AVAILABLE; ++i) {
       if (!used[i]) {
          used[i] = true;
          return items[i];
       }
     }
     return null; // not reached
   }

   protected synchronized boolean markAsUnused(Object item) {
     for (int i = 0; i < MAX_AVAILABLE; ++i) {
       if (item == items[i]) {
          if (used[i]) {
            used[i] = false;
            return true;
          } else
            return false;
       }
     }
     return false;
   }
 }
```
这里定义的是一个“池”，至于存在的是什么东西，其实就要看数组`item`是什么了。item数组是我们可获得的资源，用数组used来标识资源是否可用。同样声明一个`Semaphore`变量，其许可的数量正好是资源的数量，当申请一个资源的时候，先尝试获取许可，如果无法获得许可则线程挂起。直到许可被释放。在释放资源的时候，同时释放一个许可。


记住上面这个例子！




参考
* [Java多线程编程--（3）线程互斥、同步的理解](http://blog.csdn.net/DrifterJ/article/details/7771230)
* [Java并发编程——信号量与互斥量](https://www.jianshu.com/p/c71840db31d2)
