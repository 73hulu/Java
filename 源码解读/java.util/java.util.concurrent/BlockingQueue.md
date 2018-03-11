# BlockingQueue

阻塞队列，顾名思义，它首先是一个队列，在数据结构中起到的位置大致如下：

![队列的作用](http://img.blog.csdn.net/20150929153140497)

很显然，这是一个典型的生产者-消费者场景，线程1不断生产产品将其放到队列中，线程2不断从队列取出产品进行消费，当队列满的时候，生产队列将被阻塞；当队列空的时候，消费队列将被阻塞。

当1.5之前，强大的JUC包没有问世的时候，我们需要自己去控制这些阻塞和唤醒的逻辑，比如说`wait`方法和`notifyAll`方法的配合使用（可参考 “进程通信与线程通信”一文）。1.5之后，JUC为我们提供了一个线程的阻塞队列——`BlockingQueue`，其结构如下：

![BlockingQueue](http://ovn0i3kdg.bkt.clouddn.com/BlockingQueue.png)

可以看到，`BlockingQueue`首先得是一个队列，而队列的操作无非就是出队和入队：
所以该接口中的方法可以归为两类：
## 入队方法
| 方法 | 描述 |
| :------------- | :------------- |
|boolean offer(E e); |表示如果可能的话，将anObject加到BlockingQueue里,即如果BlockingQueue可以容纳 |
|boolean offer(E e, long timeout, TimeUnit unit) throws InterruptedException;   |  可以设定等待的时间，如果在指定的时间内，还不能往队列中加入BlockingQueue，则返回失败。 |
|     void put(E e) throws InterruptedException;  |  把anObject加到BlockingQueue里，如果BlockQueue没有空间，则调用此方法的线程被阻断，直到BlockingQueue里面有空间再继续。|


　　　　
## 出队方法
| 方法 | 描述 |
| :------------- | :------------- |
| E poll(long timeout, TimeUnit unit)throws InterruptedException; | 从BlockingQueue取出一个队首的对象，如果在指定时间内，　队列一旦有数据可取，则立即返回队列中的数据。否则知道时间超时还没有数据可取，返回失败。  |
|    E take() throws InterruptedException;   | 取走BlockingQueue里排在首位的对象,若BlockingQueue为空,阻断进入等待状态直到BlockingQueue有新的数据被加入;   |
|int drainTo(Collection<? super E> c);   | 一次性从BlockingQueue获取所有可用的数据对象（还可以指定获取数据的个数），通过该方法，可以提升获取数据效率；不需要多次分批加锁或释放锁。  |


![BlockingQueue中的方法](http://ovn0i3kdg.bkt.clouddn.com/BlockingQueue%E4%B8%AD%E7%9A%84%E6%96%B9%E6%B3%95.png)

这四类方法的特点是：
1. ThrowsException：如果操作不能马上进行，则抛出异常
2. SpecialValue：如果操作不能马上进行，将会返回一个特殊的值，一般是true或者false
3. Blocks：如果操作不能马上进行，操作会被阻塞
4. TimesOut：如果操作不能马上进行，操作会被阻塞指定的时间，如果指定时间没执行，则返回一个特殊值，一般是true或者false

`BlockingQueue`的实现主要有这么几类：
1. ArrayBlockingQueue
2. DelayQueue
3. LinkedBlockingQueue
4. PriorityBlockingQueue
5. SynchronousQueue


## 生产者-消费者
下面是注解中给出的利用`BlockingQueue`实现的生产者-消费者的实现：
```java
class Producer implements Runnable {
   private final BlockingQueue queue;
   Producer(BlockingQueue q) { queue = q; }
   public void run() {
     try {
       while (true) { queue.put(produce()); }
     } catch (InterruptedException ex) { ... handle ...}
   }
   Object produce() { ... }
 }

 class Consumer implements Runnable {
   private final BlockingQueue queue;
   Consumer(BlockingQueue q) { queue = q; }
   public void run() {
     try {
       while (true) { consume(queue.take()); }
     } catch (InterruptedException ex) { ... handle ...}
   }
   void consume(Object x) { ... }
 }

 class Setup {
   void main() {
     BlockingQueue q = new SomeQueueImplementation();
     Producer p = new Producer(q);
     Consumer c1 = new Consumer(q);
     Consumer c2 = new Consumer(q);
     new Thread(p).start();
     new Thread(c1).start();
     new Thread(c2).start();
   }
 }
```
