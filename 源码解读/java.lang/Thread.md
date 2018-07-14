# Thread

线程很重要！很重要！很重要！重要的话说三遍，然后我一直以来都闹不明白多线程是怎么回事，看了很多仍然是雾里看花，不知道这次能不能打通任督二脉。


![Thread](http://ovn0i3kdg.bkt.clouddn.com/Thread_1.png)
![Thread](http://ovn0i3kdg.bkt.clouddn.com/Thread_2.png)
![Thread](http://ovn0i3kdg.bkt.clouddn.com/Thread_3.png)

Thread类的结构如上。

## public class Thread implements Runnable
首先是类声明，可以看到`Thread`类实现了`Runnable`接口，这个接口是做什么的呢
```java
@FunctionalInterface
public interface Runnable {
    /**
     * When an object implementing interface <code>Runnable</code> is used
     * to create a thread, starting the thread causes the object's
     * <code>run</code> method to be called in that separately executing
     * thread.
     * <p>
     * The general contract of the method <code>run</code> is that it may
     * take any action whatsoever.
     *
     * @see     java.lang.Thread#run()
     */
    public abstract void run();
}
```
一个实现`Runnable`接口的子类需要重写run方法。但是不能直接调动重写过的run()来实现多线程，因为对于JVM来说，`run`方法就只是一个普通的方法。那该怎么启动呢？到后面可以看到，`Thread`类有一个构造方法：`public Thread(Runnable targer)`，此构造方法接受`Runnable`的子类实例，也就是说可以通过Thread类来启动Runnable实现的多线程。（start可以协调系统的资源）。

> `@FunctionalInterface`注解表明这是一个函数接口，这是Java 8的新特性，更多内容参见"Java 8新特性/@FunctionalInterface注解"。

## private static native void registerNatives();
诶这个就有点熟悉了，在`Object`类中见到过，本地注册方法，怎么调用这个方法呢？跟Object类中一样的处理方式，在静态构造块中调用。
```java
static {
    registerNatives();
}
```

## 构造方法
`Thread`类定义了9种构造方法，但是实质都是把形参类别再传递给`init`方法，所以只用看无参构造方法就够了。定义如下：
```java
public Thread() {
    init(null, null, "Thread-" + nextThreadNum(), 0);
}
```
方法中调用的是`init`方法，该方法定义如下：
```java
private void init(ThreadGroup g, Runnable target, String name,
                      long stackSize) {
    init(g, target, name, stackSize, null, true);
}
```
而这个方法调用的是是另外一个重载`init`的方法，定义如下：
```java
private void init(ThreadGroup g, Runnable target, String name,
                     long stackSize, AccessControlContext acc,
                     boolean inheritThreadLocals) {
     if (name == null) {
         throw new NullPointerException("name cannot be null");
     }

     this.name = name;

     Thread parent = currentThread();
     SecurityManager security = System.getSecurityManager();
     if (g == null) {
         /* Determine if it's an applet or not */

         /* If there is a security manager, ask the security manager
            what to do. */
         if (security != null) {
             g = security.getThreadGroup();
         }

         /* If the security doesn't have a strong opinion of the matter
            use the parent thread group. */
         if (g == null) {
             g = parent.getThreadGroup();
         }
     }

     /* checkAccess regardless of whether or not threadgroup is
        explicitly passed in. */
     g.checkAccess();

     /*
      * Do we have the required permissions?
      */
     if (security != null) {
         if (isCCLOverridden(getClass())) {
             security.checkPermission(SUBCLASS_IMPLEMENTATION_PERMISSION);
         }
     }

     g.addUnstarted();

     this.group = g;
     this.daemon = parent.isDaemon();
     this.priority = parent.getPriority();
     if (security == null || isCCLOverridden(parent.getClass()))
         this.contextClassLoader = parent.getContextClassLoader();
     else
         this.contextClassLoader = parent.contextClassLoader;
     this.inheritedAccessControlContext =
             acc != null ? acc : AccessController.getContext();
     this.target = target;
     setPriority(priority);
     if (inheritThreadLocals && parent.inheritableThreadLocals != null)
         this.inheritableThreadLocals =
             ThreadLocal.createInheritedMap(parent.inheritableThreadLocals);
     /* Stash the specified stack size in case the VM cares */
     this.stackSize = stackSize;

     /* Set thread ID */
     tid = nextThreadID();
 }
```

我们根据上面这个`init`整个连起来看一下。

首先是对`name`的赋值，`name`是`Thread`类的实例方法，声明如下：
```java
private volatile String name;
```
注意被`volatile`关键词修饰，说明这个变量值可能被好几个线程更改。
将形参`name`的值赋给该属性，形参`name`是字符串"Thread-" 和方法 `nextThreadNum()`返回值的拼接,与这个方法有关的定义如下：
```java
private static int threadInitNumber;
private static synchronized int nextThreadNum() {
    return threadInitNumber++;
}
```
注意到这个方法是被`synchronized`修饰，线程安全，返回的是类变量`threadInitNumber`的值，之后自增一次，注意是类变量，也就是说，每个进程中开启的线程都会被安排一个整型变量，从0开始编号，最后进行字符串拼接，赋值给name，最终这些name可能是`Thread-0`、`Thread-1`...，如果这个形参`name`是null的话，将会抛出`NullPointerException`。

接着` Thread parent = currentThread();`这句话调用了静态方法`currentThread`：
```java
public static native Thread currentThread();
```
native方法，从名字就能知道这个方法用来获取当前线程。

接下来，`SecurityManager security = System.getSecurityManager();`获取Java安全管理器，接下来是对变量g的赋值过程，变量g是什么？线程组，一个线程组管理很多线程。g的初始值为null，所以会进入if语句块中，首先尝试让Java安全组件给g赋值，如果不行，那么就将这个新创建的线程加入到当前线程的线程组中。

你说加入就加入？当然不行，后面还有一系列的检查balabala，现在看不懂，学到`ThreadGroup`的时候再说。

之后就愉快地把g这个值赋值给成员变量group了，这个成员变量定义如下：
```java
private ThreadGroup group;
```
接下来这句`this.daemon = parent.isDaemon();`，daemon是实例属性，定义为
```java
 private boolean  daemon = false;
```
这个变量标识这个线程是否为“守护线程”，什么是守护线程？

守护线程是指在程序运行的时候再后台提供的一种通用服务的线程，比如垃圾回收线程就是一个很称职的守护者，并且这种线程并不属于程序中不可或缺的部分。因此当所有的非守护线程结束时，程序也就结束了，同时会杀死进程中所有的守护线程。反过来说，只有任何非守护线程还在运行，程序就不会中止。

> 更多关于守护线程的内容参考"Java基础知识整理/多线程"

可以看到，新建的这个线程的该属性被赋值为当前线程的daemon属性值，那我们可以得出结论：从daemon线程中产生的线程也是daemon线程。

接下来执行`this.priority = parent.getPriority();`这句话，`priority`变量声明如下：
```java
private int priority;
```
注意到时int型变量，`getPriority`方法定义如下：
```java
public final int getPriority() {
    return priority;
}
```
所以新建的线程与当前线程的优先级保持一致？不，优先级这事还没完，后面还有招。

接着往下是给`contextClassLoader`和`inheritedAccessControlContext`两个变量赋值，涉及到Java安全体系，看不太懂，先不管了。

接下来给`target`变量赋值，该变量声明如下：
```java
private Runnable target;
```
自带`run`方法而来。

接下来就是设定`priority`了，`setPriority`方法定义如下：
```java
public final void setPriority(int newPriority) {
    ThreadGroup g;
    checkAccess();
    if (newPriority > MAX_PRIORITY || newPriority < MIN_PRIORITY) {
        throw new IllegalArgumentException();
    }
    if((g = getThreadGroup()) != null) {
        if (newPriority > g.getMaxPriority()) {
            newPriority = g.getMaxPriority();
        }
        setPriority0(priority = newPriority);
    }
}
```
这个方法首先进行参数合法性判断，`Thread`类定义了三种线程优先级：
```java
/**
    * The minimum priority that a thread can have.
    */
   public final static int MIN_PRIORITY = 1;

  /**
    * The default priority that is assigned to a thread.
    */
   public final static int NORM_PRIORITY = 5;

   /**
    * The maximum priority that a thread can have.
    */
   public final static int MAX_PRIORITY = 10;
```
`priority`只有三种取值，不得超出范围。接下来，新线程的优先级不得大于线程组中的最大优先级，如果超出了，将会被重新调整为线程组的最大优先级。怎么调整？调用`setPriority0`方法，这是一个native方法，不多讲了。

接下来设定`inheritableThreadLocals`这个值不知道在干什么了，/(ㄒoㄒ)/~~

然后设定`stackSize`的值，在这个无参构造方法中，将其设置为0。

接下来设定变量tid的值，调用`nextThreadID`方法，定义如下：
```java
private static synchronized long nextThreadID() {
   return ++threadSeqNumber;
}
```
其中这个`threadSeqNumber`这个变量是一个静态变量`private static long threadSeqNumber;`

有一个问题?这个变量和之前说到的`threadInitNumber`这个变量有什么区别么？现在我能分辨的区别是：`threadInitNumber`是在name中是从0开始的，而`threadSeqNumber`在被操作的第一个值是1。做了一个实验：
```java
public static void main(String[] args) {
   Thread thread = new Thread();

   System.out.print(thread.toString());
}
```
debug得到的thread的内容如下：

![thread_test](http://ovn0i3kdg.bkt.clouddn.com/Thread_test.png)

可以看到name值是“Thread-0”，而tid却是12，为什么？？？不应该是从1开始么？

总结一下，创建一个线程的过程中都做了下面这些事：
1. 命名：Thread-1
2. 加入ThreadGroup，如果在构造方法中指定线程组，那么就加入以当前父线程所在的线程组。父线程的获取方法是`currentThread()`。
3. 设定是否是守护线程，新线程与父线程的参数保持一致。即父线程如果是守护线程，那么新线程也是守护线程。
4. 设定优先级。新线程的优先级将与父线程保持一致，但是不能超过所在线程组的最高优先级。



## void blockedOn(Interruptible b) {...}


## public synchronized void start(){...}
这个方法太重要了，
```java
public synchronized void start() {
    /**
     * This method is not invoked for the main method thread or "system"
     * group threads created/set up by the VM. Any new functionality added
     * to this method in the future may have to also be added to the VM.
     *
     * A zero status value corresponds to state "NEW".
     */
    if (threadStatus != 0)
        throw new IllegalThreadStateException();

    /* Notify the group that this thread is about to be started
     * so that it can be added to the group's list of threads
     * and the group's unstarted count can be decremented. */
    group.add(this);

    boolean started = false;
    try {
        start0();
        started = true;
    } finally {
        try {
            if (!started) {
                group.threadStartFailed(this);
            }
        } catch (Throwable ignore) {
            /* do nothing. If start0 threw a Throwable then
              it will be passed up the call stack */
        }
    }
}
```
开发者调动`start`方法，然后再由JVM调用`run`方法，注意这两个方法的调用者是不同的。该方法首先判断线程状态，必须是0否则抛出异常，为什么是0？`threadStartFailed`这个变量的声明为:`private volatile int threadStatus = 0;`发现这个变量被`volatile`关键词修饰，所以这个量将来会多个线程改变。

接着`group.add(this)`将通知这个线程所属的线程组：这个线程已经准备好了哦，线程组将这个线程添加到线程数组里面，然后把数量加上1，具体的还是学到`ThreadGroup`的时候看吧。

之后调用了`start0`方法，这是一个native方法，如果这个方法抛出异常，那么线程组将会移除这个线程，并且将相应的计数器`nUnstartedThreads`增加1，但是在这个方法中对异常不作处理，把这个异常抛给调用者。

###  public void run(){...}
```java
@Override
public void run() {
    if (target != null) {
        target.run();
    }
}
```
`run`方法才是真正线程执行的逻辑。`Thread`实现了`Runnable`接口，所以重写了run方法。但是可以看到`Thread`的run方法本身调用的target的run方法，target是什么？是实现Runnable接口的类的实例。所以啊，要创建线程实现自己的run逻辑，可以有以下两种方法。

1. 自定义类继承`Thread`，重写run()方法覆盖父类（强制），实例化该类，调用start方法。

  ```java
  public class ThreadTest {
      public static void main(String[] args) {

          MyThread myThread = new MyThread();
          myThread.start();
          for (int j = 0; j < 100; j++){
              System.out.println("test1.main()----"+j);
          }
      }
  }
  class MyThread extends Thread{

      @Override
      public void run() {
          for (int i = 0; i < 100; i++){
              System.out.println("MyThread.run()..."+i);
          }   /*单条线程运行一百次*/
      }
  }
  ```
2. 自定义类继承`Runnable`接口，重写run()方法（非强制），实例化该类，以此为参数创建`Thread`对象，调用start方法。

  ```java
  public class RunableTest {
      public static void main(String[] args) {
          MyThread1 myThread = new MyThread1();
          Thread a = new Thread(myThread);
          Thread b = new Thread(myThread);
          Thread c = new Thread(myThread);
          a.start();
          b.start();
          c.start();
      }
  }
  class MyThread1 implements Runnable{
      @Override
      public void run() {
          for (int i = 0; i < 100; i++){
              System.out.println("MyThread.run()..."+i);
          }   /*单条线程运行一百次*/
      }
  }
  ```
上面两种方式都可以创建线程，而且结果相差不多，但是最好是第二种，为什么呢？因为现实问题中创建线程的目的是为了让多个线程来执行同一个任务，并且这多个线程还共享同一个资源，这种需求可以使用实现`Runable`接口的方式来实现多线程任务，但是不能通过扩展Thread类是无法实现的。比如下面这个例子：
```java
public class RunableTest {
    public static void main(String[] args) {
        MyThread m=new MyThread();
        Thread t1=new Thread(m,"Thread 1");
        Thread t2=new Thread(m,"Thread 2");
        Thread t3=new Thread(m,"Thread 3");
        t1.start();         /*共运行一百次，  三条线程哪条抢到资源就运行哪条  */
        t2.start();
        t3.start();
    }
}
class MyThread implements Runnable{
    private int i=0;

    @Override
    public void run() {
            while(i<100){
                i++;
                System.out.println(i+"  MyThread.run "+Thread.currentThread().getName());
        }
    }
}
```
两种方式的优劣总结如下：

|  | 继承 Thread   类   | 实现Runnable接口|
| :------------- | :------------- | : ---|
| 优点      | 编写简单，如果需要访问当前线程，无需使用Thread.currentThread()方法，直接使用this，即可获得当前线程。| 线程类只是实现了Runable接口，还可以继承其他的类。在这种方式下，可以多个线程共享同一个目标对象，所以非常适合多个相同线程来处理同一份资源的情况，从而可以将CPU代码和数据分开，形成清晰的模型，较好地体现了面向对象的思想。|
|缺点   |因为线程类已经继承了Thread类，所以不能再继承其他的父类。   |  编程稍微复杂，如果需要访问当前线程，必须使用Thread.currentThread()方法。 |

3. 实现Callable接口
`Callable`是JUC中新加入的接口，主要和线程池一起使用。具体的使用方法可以参见博文"多线程"。

> Thread对象可以操纵一个线程，而Runable对象代表一个可被运行的对象。

### private void exit() {...}

这个方法也是被JVM调用的，在真正退出之前给线程一个清理的机会？为什么清理？垃圾回收啊~
```java
private void exit() {
   if (group != null) {
       group.threadTerminated(this);
       group = null;
   }
   /* Aggressively null out all reference fields: see bug 4006245 */
   target = null;
   /* Speed the release of some of these resources */
   threadLocals = null;
   inheritableThreadLocals = null;
   inheritedAccessControlContext = null;
   blocker = null;
   uncaughtExceptionHandler = null;
}
```
可以看到把引用全部置为null，还通知所属的线程组结束掉该线程。

## public static native void yield();
`yield`本身是native方法，实现细节不得而知。那这个方法有什么作用呢？单词"yield"的意思是“屈服、放弃”，`Thread.yield( )`方法经常被翻译成“线程让步”，它会把自己的CPU执行时间让掉，让自己或者其他线程运行。从线程的状态转化角度看，它能够让当前线程从**运行**状态变为**就绪状态**，这时候它就和别的处于就绪状态的线程一样了，下一次能不能被调度完全取决于OS，它并没有什么优势。那么这些线程中优先级高的就有优势么？没有，只是被挑中的概率高一点而已，也有可能是优先级低的抢到了。

## sleep方法
sleep会导致**当前线程**进入睡眠状态，即阻塞状态。有两个重载的方法。下面是两个参数的sleep方法：
```java
public static void sleep(long millis, int nanos)
 throws InterruptedException {
     if (millis < 0) {
         throw new IllegalArgumentException("timeout value is negative");
     }

     if (nanos < 0 || nanos > 999999) {
         throw new IllegalArgumentException(
                             "nanosecond timeout value out of range");
     }

     if (nanos >= 500000 || (nanos != 0 && millis == 0)) {
         millis++;
     }

     sleep(millis);
 }
```
第一个参数单位是毫秒，第二个参数的单位是纳秒，为什么这么设计？这种写法在之前Object里的wait()方法中见过，为了更精确地控制等待时间。最后调用的是一个参数的sleep方法，定义如下：
```java
public static native void sleep(long millis) throws InterruptedException;
```
这个一个native方法，怎么实现不要深纠了。

sleep这个方法是静态方法，所以方法调用的形式是`Thread.sleep()`，另外注意，使用该方法需要捕获`InterruptedException`异常。还有一点非常非常非常重要。在源码中被明确说明：

> The thread does not lose ownership of any monitors.

当前线程不会失去任何的锁！！！即如果当前线程持有某个对象锁，在它sleep的这段时间内，锁还是由他保有的，不会被释放掉。

## join方法
`thread.join`方法把指定的线程加入到当前线程，可以将两个交替执行的线程合并为顺序执行的线程。什么意思呢？原来A和B线程是交替进行的，此时在B线程中调用的a.join()，那么直到线程A执行完毕之后，才会继续执行B，这样原本并发的线程就串行了。有三个重载方法，核心是下面这个：
```java
public final synchronized void join(long millis)
 throws InterruptedException {
     long base = System.currentTimeMillis();
     long now = 0;

     if (millis < 0) {
         throw new IllegalArgumentException("timeout value is negative");
     }

     if (millis == 0) {
         while (isAlive()) {
             wait(0);
         }
     } else {
         while (isAlive()) {
             long delay = millis - now;
             if (delay <= 0) {
                 break;
             }
             wait(delay);
             now = System.currentTimeMillis() - base;
         }
     }
 }
```
从代码中可以可以看到，**如果线程被生成了，但是还没有被启动，即isAlive返回false，此时调用join是没有作用的**，将直接继续往下执行。

还可以看到，join方法实际上是调用了Object的wait方法实现的。当main线程调用t.join的时候，main线程将会获得线程对象t的锁（wait意味着拿到了该对象的锁），调用该对象的wait(等待时间)，直到该对象唤醒main线程。比如退出后，这就意味着main线程调用t.join时，必须能够拿到线程t对象的锁。

一个非常好的例子可以说明join方法的作用。

首先看下面的程序：
```java
public class ThreadTest implements Runnable {
    public static  int i = 0;

    @Override
    public void run() {
        for (int k = 0; k < 5; k ++){
            i = i  + 1;
        }
    }
    public static void main(String[] args) throws Exception{
        Runnable r = new ThreadTest();
        Thread t = new Thread(r);
        t.start();
        System.out.print(i);
    }
}
```
请问会输出5么？答案是：有可能，但是大部分情况下都不会是5，当然这和机器有很大的关系。为什么？因为这里其实有两个线程，主线程main和main中产生的线程t，当主线程main执行`System.out.println(a)`这句的时候，线程t还没有真正开始运行，或许正在为它分配资源，这需要时间啊。而主线程执行到打印语句的时候a还没有改变，这时候当然输出0啊？那我怎么才能让程序输出5呢，那当然是让主线程main等待t线程执行完之后再打印。用join方法就可以做到了。
如下：
```java
public class ThreadTest implements Runnable {
    public static  int i = 0;

    @Override
    public void run() {
        for (int k = 0; k < 5; k ++){
            i = i  + 1;
        }
    }
    public static void main(String[] args) throws Exception{
        Runnable r = new ThreadTest();
        Thread t = new Thread(r);
        t.start();
        t.join();  //默认时间是0，表示会无限期地等待下去
        System.out.print(i);
    }
}
```
由于在main线程中调用了t的join方法，所以主线程必须等待t线程结束后才能继续。

join方法可以有参数，表示等待的时间，无指定的时候默认为0，表示无期限等待下去。做下面这个实验：
```java
public class ThreadTest implements Runnable {

    @Override
    public void run() {
        try {
            System.out.println("Begin sleep");
            Thread.sleep(1000);
            System.out.println("End sleep");
        }catch (InterruptedException e){
            e.printStackTrace();
        }

    }
    public static void main(String[] args) throws Exception{
        Runnable r = new ThreadTest();
        Thread t = new Thread(r);
        t.start();
        try {
            System.out.println("JoinBegin");
            t.join(1000);
            System.out.println("JoinFinish");
        }catch (InterruptedException e){
            e.printStackTrace();
        }
    }
}
```
打印的结果是：
```java
JoinBegin
Begin sleep
JoinFinish
End sleep
```
如果run中的sleep的参数改成2000，打印的结果就是下面这种：
```java
JoinBegin
Begin sleep
JoinFinish
End sleep
```
可见，等待是有期限的，到点了我就往下进行了，不管t到什么时候结束。

一个更加复杂的例子：
```java
class CustomThread1 extends Thread {

    public void run() {
        String threadName = Thread.currentThread().getName();
        System.out.println(threadName + " start.");
        try {
            for (int i = 0; i < 5; i++) {
                System.out.println(threadName + " loop at " + i);
                Thread.sleep(1000);
            }
            System.out.println(threadName + " end.");
        } catch (Exception e) {
            System.out.println("Exception from " + threadName + ".run");
        }
    }
}

class CustomThread extends Thread {
    CustomThread1 t1;
    public CustomThread(CustomThread1 t1) {
        this.t1 = t1;
    }
    public void run() {
        String threadName = Thread.currentThread().getName();
        System.out.println(threadName + " start.");
        try {
            t1.join();
            System.out.println(threadName + " end.");
        } catch (Exception e) {
            System.out.println("Exception from " + threadName + ".run");
        }
    }
}

public class JoinTest {

    public static void main(String[] args) {
        String threadName = Thread.currentThread().getName();
        System.out.println(threadName + " start.");
        CustomThread1 t1 = new CustomThread1();
        CustomThread t = new CustomThread(t1);
        try {
            t1.start();
            Thread.sleep(2000);
            t.start();
            t.join(); //在之后的那个例子中将这句话注释掉
        } catch (Exception e) {
            System.out.println("Exception from main");
        }
        System.out.println(threadName + " end!");
    }
}
```
输出的结果如下：
```java
main start.    //main方法所在的线程起动，但没有马上结束，因为调用t.join();，所以要等到t结束了，此线程才能向下执行。
[CustomThread1] Thread start.     //线程CustomThread1起动
[CustomThread1] Thread loop at 0  //线程CustomThread1执行
[CustomThread1] Thread loop at 1  //线程CustomThread1执行
[CustomThread] Thread start.      //线程CustomThread起动，但没有马上结束，因为调用t1.join();，所以要等到t1结束了，此线程才能向下执行。
[CustomThread1] Thread loop at 2  //线程CustomThread1继续执行
[CustomThread1] Thread loop at 3  //线程CustomThread1继续执行
[CustomThread1] Thread loop at 4  //线程CustomThread1继续执行
[CustomThread1] Thread end.       //线程CustomThread1结束了
[CustomThread] Thread end.        // 线程CustomThread在t1.join();阻塞处起动，向下继续执行的结果
main end!      //线程CustomThread结束，此线程在t.join();阻塞处起动，向下继续执行的结果。
```
如果将`t.join()`注释掉，那么打印结果如下：
```java
main start. // main方法所在的线程起动，但没有马上结束，这里并不是因为join方法，而是因为Thread.sleep(2000);
[CustomThread1] Thread start.      //线程CustomThread1起动
[CustomThread1] Thread loop at 0   //线程CustomThread1执行
[CustomThread1] Thread loop at 1   //线程CustomThread1执行
main end!   // Thread.sleep(2000);结束，虽然在线程CustomThread执行了t1.join();，但这并不会影响到其他线程(这里main方法所在的线程)。
[CustomThread] Thread start.       //线程CustomThread起动，但没有马上结束，因为调用t1.join();，所以要等到t1结束了，此线程才能向下执行。
[CustomThread1] Thread loop at 2   //线程CustomThread1继续执行
[CustomThread1] Thread loop at 3   //线程CustomThread1继续执行
[CustomThread1] Thread loop at 4   //线程CustomThread1继续执行
[CustomThread1] Thread end.       //线程CustomThread1结束了
[CustomThread] Thread end.        // 线程CustomThread在t1.join();阻塞处起动，向下继续执行的结果
```

此外，main线程在调用join方法的时候，必须能够拿到线程t对象的锁，如果拿不到是无法wait的，刚才的例子t.join(1000)不是为了main线程等待1秒，如果在让等待之前，其他线程获取了t对象的锁，那么它等待的时间可就不是1秒了。例如：
```java
class RunnableImpl implements Runnable {  

    public void run() {  
        try {  
            System.out.println("Begin sleep");  
            Thread.sleep(2000);  
            System.out.println("End sleep");  
        } catch (InterruptedException e) {  
            e.printStackTrace();  
        }  
    }  
}
class ThreadTest extends Thread {  

    Thread thread;  

    public ThreadTest(Thread thread) {  
        this.thread = thread;  
    }  

    @Override  
    public void run() {  
        synchronized (thread) {  
            System.out.println("getObjectLock");  
            try {  
                Thread.sleep(9000);  
            } catch (InterruptedException ex) {  
             ex.printStackTrace();  
            }  
            System.out.println("ReleaseObjectLock");  
        }  
    }  
}
public class JoinTest {  
        public static void main(String[] args) {  
            Thread t = new Thread(new RunnableImpl());  
           new ThreadTest(t).start();  
            t.start();  
            try {  
                t.join();  
                System.out.println("joinFinish");  
            } catch (InterruptedException e) {  
                e.printStackTrace();           
            }  
        }  
}
```
输出结果是：
```java
getObjectLock
Begin sleep
End sleep
ReleaseObjectLock
joinFinish
```
在main方法中 通过`new  ThreadTest(t).start()`实例化 ThreadTest 线程对象， 它通过`synchronized  (thread)` ，获取线程对象t的锁，并`Sleep（9000）`后释放，这就意味着，即使main方法`t.join(1000)`等待一秒钟，它必须等待ThreadTest 线程释放t锁后才能进入wait方法中，它实际等待时间是9000+1000ms。


## public void interrupt(){...}

定义如下：
```java
public void interrupt() {
    if (this != Thread.currentThread())
        checkAccess();

    synchronized (blockerLock) {
        Interruptible b = blocker;
        if (b != null) {
            interrupt0();           // Just to set the interrupt flag
            b.interrupt(this);
            return;
        }
    }
    interrupt0();
}
```
这里面有些变量需要解释一下。`blockerLock`是`Thread`类的一个私有常量`private final Object blockerLock = new Object();`，其实就是一个对象实例，而`blocker`是一个实例变量：`private volatile Interruptible blocker;`。而`interrupt0`是一个native方法。

源码中对该方法有这样的解释
> 中断线程。
> 如果当前线程没有中断它自己（这在任何情况下都是允许的），则该线程的 checkAccess 方法就会被调用，这可能抛出 SecurityException。
> 如果线程在调用 Object 类的 wait()、wait(long) 或 wait(long, int) 方法，或者该类的 join()、join(long)、join(long, int)、sleep(long) 或 sleep(long, int) 方法过程中受阻，则其中断状态将被清除，它还将收到一个 InterruptedException。
> 如果该线程在可中断的通道上的 I/O 操作中受阻，则该通道将被关闭，该线程的中断状态将被设置并且该线程将收到一个 ClosedByInterruptException。
> 如果该线程在一个 Selector 中受阻，则该线程的中断状态将被设置，它将立即从选择操作返回，并可能带有一个非零值，就好像调用了选择器的 wakeup 方法一样。
> 如果以前的条件都没有保存，则该线程的中断状态将被设置。

过程还是看不懂啊，那就先记住结论吧。 `interrupt()`不会中断一个正在运行的线程。这一方法实际上完成的是，在线程受到阻塞时抛出一个中断信号，这样线程就得以退出阻塞的状态。更确切的说，如果线程被`Object.wait`, `Thread.join`和`Thread.sleep`三种方法之一阻塞，那么，它将接收到一个中断异常（InterruptedException），从而提早地终结被阻塞状态；如果线程没有被阻塞，这时调用`interrupt()`将不起作用；否则，线程就将得到异常（该线程必须事先预备好处理此状况），接着逃离阻塞状态。

线程A在执行`sleep`，`wait`,`join`时,线程B调用A的 `interrupt`方法，的确这一个时候A会有 `InterruptedException`异常抛出来。但这其实是在`sleep`，`wait`，`join`这些方法内部会不断检查中断状态的值,而自己抛出的`InterruptedException`。

如果线程A正在执行一些指定的操作时如赋值，for，while，if调用方法等，都不会去检查中断状态,所以线程A不会抛出`InterruptedException`,而会一直执行着自己的操作。
当线程A终于执行到`wait()`，`sleep()`，`join()`时，才马上会抛出`InterruptedException`。若没有调用`sleep()`，`wait()`，`join()`这些方法,或是没有在线程里自己检查中断状态自己抛出`InterruptedException`的话,那`InterruptedException`是不会被抛出来的.

"中断"的意思并不是让线程停止（而stop方法则是让线程终止），而只是改变了线程的状态，至于改变了这种状态之后，线程是死亡？继续？还是怎样，完全视线程本身而定。有时候这种中断反而不是阻止，反而是让其继续执行，比如一个正发生死锁的线程，中断它才能让它继续执行。

每个线程实例都有一个boolean类型标记，表示这个线程是否被请求中断，当其`interrupt`方法被调用的时候，这个boolean状态位会置为true。调用了某个线程a的中断方法`interrupt()`，一定会让线程a停下来么？不一定，因为这个方法只是把标志位的状态改变了，并没有真正kill掉线程。那么这个状态的变化有什么用呢？当然有用，如果该线程正处于阻塞状态（比如等待阻塞、同步阻塞或者睡眠阻塞等），处于这种状态的线程是会不断检查这个标志位的，一旦检查到了这个标注为被标注为true，则抛出`InterruptedException`，同时清楚这个标志位为false。这时候线程就从阻塞状态被唤醒了，至于被唤醒之后要做什么，完全看线程本身的实现。如果线程正在运行着，它是不会检查这个标志位的，也就是说，即使标志位设为true，它也能照常运行。

上面说到wait的线程如果发现状态位为true了，将被唤醒，和notify与notifyAll的效果类似，有什么不同呢？不同之处就在于try-catch块了：如果线程是被notify或者notifyAll唤醒的，那么紧接着wait方法之后的语句会照常执行，而如果是被中断唤醒的话，那么就会直接进入catch语句，这就是普通的异常捕获过程，没什么难理解的。

举一个例子：程序开始让一个线程`sleep`一年，但是你反悔了，那么此时调用`interrupted`方法将是唤醒这个线程的唯一办法。同理，`wait`、`join`也是同样的道理，特别注意的是，`synchronized`在获锁的过程中是不能被中断的，意思是说如果产生了死锁，则不可能被中断。与`synchronized`功能相似的`reentrantLock.lock()`方法也是一样，它也不可中断的，即如果发生死锁，那么`reentrantLock.lock()`方法无法终止，如果调用时被阻塞，则它一直阻塞到它获取到锁为止。但是如果调用带超时的`tryLock`方法`reentrantLock.tryLock(long timeout, TimeUnit unit)`，那么如果线程在等待时被中断，将抛出一个`InterruptedException`异常，这是一个非常有用的特性，因为它允许程序打破死锁。你也可以调用`reentrantLock.lockInterruptibly()`方法，它就相当于一个超时设为无限的`tryLock`方法。这一点要牢记。


`interrupt`方法是JVM来中断线程的一个武器，有商量的余地，效果还是比较温和的。JVM还提供了几个杀伤力巨大的武器，由于杀伤力太大，会殃及无辜，所以都废弃了，但是还是学习一下。这些武器有：`stop`、`suspend`。

首先是`stop`方法，这个方法已经被废弃了，千万别用了，因为执行这个方法将会立即杀死一个线程，将其锁执有的锁全都释放，导致本该被锁控制同步的所有位置变得无法控制。会产生无法预计的问题，这妥妥的是一场灾难。

其次是`suspend`方法，这个方法还是可用的但是尽量别用，官方的说法是可能导致死锁。代替这个方法的是wait和notify方法。

其实学习过之后就忘了吧。脑子就这么大，记多了容易混~~~



## public static boolean interrupted(){...}
```java
public static boolean interrupted() {
    return currentThread().isInterrupted(true);
}
```
测试**当前线程**是否已经中断。线程的中断状态也由该方法清除。也就是说，如果连续两次调用该方法，那么第一次调用时，如果当前线程已经处于中断状态，那么该方法会返回true，同时清除当前线程被标记的中断状态。第二次调用时，（第二次调用之前，没有再次调用Thread.currentThread().interrupt();）就会返回false了。
其中`isInterrupted`方法定义如下：
```java
private native boolean isInterrupted(boolean ClearInterrupted);
```
该方法用来测试线程是否已经中断。线程的中断状态 不受该方法的影响。 如果该线程已经中断，则返回 true；否则返回 false。和`interrupted`方法不同的是，这个方法不具有清除状态的功能。

> 要特别注意静态方法`interrupted`方法和实例方法`isInterrupted`的区别
> 比如下面这个例子：
> ```
public static void main(String[] args) {
      /**
       * 这种方式中断线程，因为isInterrupted方法并没有中断线程的功能，
       * 所以当发生中断的时候，日志会输出异常信息，然后while循环退出，
       */
      new Thread(new Runnable() {
          @Override
          public void run() {
              while (!Thread.currentThread().isInterrupted()) {
                  try {
                      TimeUnit.SECONDS.sleep(5);
                  } catch (InterruptedException e) {
                      log.error("error", e);
                  }
              }
              System.out.println("子线被打断执行");
          }
      }).start();
}
```
而正确的做法是：
```
public static void main(String[] args) {
Thread thread = new Thread(new Runnable() {
           @Override
           public void run() {
               while (!Thread.currentThread().isInterrupted()){
                   try {
                       TimeUnit.SECONDS.sleep(5);
                   }catch (InterruptedException e){
                       log.error("error", e);
                   }
               }
               System.out.println("子线程被打断执行");
           }
       });
       thread.start();
       thread.interrupt();
   }
}
```


## public final native boolean isAlive();
用来检测线程时候还活着，只要线程状态处于start和die之间，都是活着的状态。

## public static int activeCount(){...}
获得当前线程组中的活跃的线程数。定义如下：
```java
public static int activeCount() {
    return currentThread().getThreadGroup().activeCount();
}
```

## public static int enumerate(Thread tarray[]){...}
将活动线程拷贝到一个数组
```java
public static int enumerate(Thread tarray[]) {
    return currentThread().getThreadGroup().enumerate(tarray);
}
```

## ThreadLocalMap和ThreadLocal
```Java
/* ThreadLocal values pertaining to this thread. This map is maintained
     * by the ThreadLocal class. */
ThreadLocal.ThreadLocalMap threadLocals = null;

/*
 * InheritableThreadLocal values pertaining to this thread. This map is
 * maintained by the InheritableThreadLocal class.
 */
ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;
```



参考：
* [创建线程的两种方式区别](http://blog.csdn.net/krito_blog/article/details/54808617)
* [Thread.interrupted()与Thread.isInterrupted()的区别](http://blog.csdn.net/fjse51/article/details/53928272)
* [Thread类的interrupt,interrupted,isInterrupted方法的理解](http://macleo.iteye.com/blog/626283)
* [Java多线程sleep(),join(),interrupt(),wait(),notify()](http://www.blogjava.net/fhtdy2004/archive/2009/06/08/280728.html)
* [Java多线程中join方法的理解](http://uule.iteye.com/blog/1101994)!!!
* [Thread的中断机制(interrupt)](http://www.cnblogs.com/onlywujun/p/3565082.html)
* [线程](http://www.cnblogs.com/hvicen/p/6218981.html)
* [Java线程同步小陷阱，你掉进去过吗](http://blog.csdn.net/smcwwh/article/details/7190933)
