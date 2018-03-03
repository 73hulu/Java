# LockSupport

AQS提供了线程等待队列的操作，其中线程阻塞和唤醒操作是通过`LockSupport`的静态方法来进行的。实际上，`LockSupport`是用来创建锁和其他同步类的基本线程阻塞原语。

类结构如下

![LockSupport](http://ovn0i3kdg.bkt.clouddn.com/LockSupport.png?imageView/2/w/500)

注意到，该类对外提供的都是静态方法，其中`park()`和`unpark()`分别用来阻塞线程和解除阻塞。而且`park()`和`unpark()`不会遇到`Thread.suspend` 和 `Thread.resume`所可能引发的死锁”问题。
因为`park()` 和 `unpark()`有许可的存在；调用`park()`的线程和另一个试图将其`unpark()`的线程之间的竞争将保持活性。

而事实上，这些方法都是抵用了`Unsafe`中的接口来实现阻塞和非阻塞的。比较难以理解，所以这里只需要知道这个API的说明即可。


| 方法 | 含义 |
| :------------- | :------------- |
| public static void park()   |  为了线程调度，禁用当前线程，除非许可可用。 |
| public static void park(Object blocker)       |  为了线程调度，在许可可用之前禁用当前线程。  |
|  public static void unpark(Thread thread)  | 如果给定线程的许可尚不可用，则使其可用。  |
|public static void parkNanos(long nanos)   | 为了线程调度禁用当前线程，最多等待指定的等待时间，除非许可可用。  |
|public static void parkNanos(Object blocker, long nanos)   |  为了线程调度，在许可可用前禁用当前线程，并最多等待指定的等待时间。  |
|  public static Object getBlocker(Thread t) |  返回提供给最近一次尚未解除阻塞的 park 方法调用的 blocker 对象，如果该调用不受阻塞，则返回 null。 |
| public static void parkUntil(long deadline)| 为了线程调度，在指定的时限前禁用当前线程，除非许可可用。  |
|  public static void parkUntil(Object blocker, long deadline) |为了线程调度，在指定的时限前禁用当前线程，除非许可可用。   |

下面是利用`synchronized`来进行同步的一段程序：
```java
public class WaitTest1 {

    public static void main(String[] args) {

        ThreadA ta = new ThreadA("ta");

        synchronized(ta) { // 通过synchronized(ta)获取“对象ta的同步锁”
            try {
                System.out.println(Thread.currentThread().getName()+" start ta");
                ta.start();

                System.out.println(Thread.currentThread().getName()+" block");
                // 主线程等待
                ta.wait();

                System.out.println(Thread.currentThread().getName()+" continue");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    static class ThreadA extends Thread{

        public ThreadA(String name) {
            super(name);
        }

        public void run() {
            synchronized (this) { // 通过synchronized(this)获取“当前对象的同步锁”
                System.out.println(Thread.currentThread().getName()+" wakup others");
                notify();    // 唤醒“当前对象上的等待线程”
            }
        }
    }
}
```
我们可以利用阻塞原语`LockSupport`来实现：
```java
public class LockSupportTest {

    private static Thread mainTread;

    public static void main(String[] args) {
        Thread ta = new LockSupportTest().new ThreadA("test Thread");

        mainTread = Thread.currentThread(); //获得主线程

        System.out.println(Thread.currentThread().getName() + " start thread ta");

        ta.start();

        System.out.println(Thread.currentThread().getName() + " blocked");

        LockSupport.park(mainTread);//当前主线程阻塞

        System.out.println(Thread.currentThread().getName() + " continue");

    }
    private class ThreadA extends Thread{
        public ThreadA(String name){
            super(name);
        }
        @Override
        public void run() {
            System.out.println(Thread.currentThread().getName() + " wake up other threads");
            LockSupport.unpark(mainTread);
        }
    }
}
```
代码运行结果为：
```
main start thread ta
main blocked
ta wake up other threads
main continue
```

经常会问到`park`和`wait`方法有什么不同，两者都是让线程进入等待阻塞，不同的是，`wait`方法是通过锁来进行同步的，即让线程阻塞之前，必须要获得锁。而`park`是阻塞原语，是不需要获得锁的。


> 什么是原语？内核或微核提供核外调用的过程或函数称为原语(primitive)，就理解为一组机器指令。



参考
* [Java多线程系列--“JUC锁”07之 LockSupport](https://www.cnblogs.com/skywang12345/p/3505784.html#a1)
