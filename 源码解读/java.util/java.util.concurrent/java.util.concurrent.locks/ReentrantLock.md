# ReentrantLock

> 这篇文章讲的最明白： http://blog.csdn.net/pengdandezhi/article/details/67082171

`Reentrant`这个单词的意思是“可重入”，所以`ReentrantLock`的意思就是"可重入锁"。它允许把锁的实现作为Java类，而不是作为语言的特性来实现。它除了在扩展性上与同步方法和声明的隐式监控锁不同外，在基本行为和语义上都相同。还支持取锁的公平与不公平的选择。

> 这里提到了两个陌生的概念。
> 1. 什么是“可重入”？ 可重入的锁即表示该锁能够支持一个线程对资源的重复加锁。实际上`synchorinized`也支持隐式重入，而对于`ReentrantLock`而言，对于已经获取到锁的线程，再次调用`lock()`方法时可以获取锁而不被阻塞。
> 2. 什么是公平获取锁？什么是不公平获取锁。**如果在绝对时间上，先对于锁进行获取请求一定先被满足，那么这个锁就是公平的，反之就是不公平**。`ReentrantLock`可以通过构造函数来控制到底是公平锁还是不公平锁。实际上非公平锁的效率远远大于公平锁。

类结构如下：

![ReentrantLock](http://ovn0i3kdg.bkt.clouddn.com/ReenrantLock.png)

![ReenrantLock](http://img.blog.csdn.net/20170327231424901?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcGVuZ2RhbmRlemhp/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

一个拥有可重入锁的线程最后成功锁定，但尚未解锁它。当这个锁并没有被其他线程拥有时，一个线程调用锁，将成功的返回请求的这个锁。如果当前线程已经拥有该锁了，该方法会立刻返回。这可以通过`isHeldByCurrentThread()`和`getHoldCount()`两个方法检查。


## public class ReentrantLock extends Object implements Lock, Serializable
类声明，可以看到类继承`Object`，实现了`Lock`接口和`Serializable`接口。


## abstract static class Sync extends AbstractQueuedSynchronizer{..}
AQS是整个JUC的核心，对于各种锁的实现来说，他们并不直接继承AQS，而是定义内部类来继承AQS。在学习AQS的时候说到，同步锁的实现实际上只需要重写AQS中的`tryAcquire-tryRelease`或者`tryAcquireShared-tryReleaseShare`，具体是那个要看采取独占模式还是共享模式。

`Sync`就是`ReentrantLock`中对于AQS的实现，这是一个静态抽象内部类，非常重要。其定义如下：
```Java

abstract static class Sync extends AbstractQueuedSynchronizer {
   private static final long serialVersionUID = -5179523762034025860L;

   /**
    * Performs {@link Lock#lock}. The main reason for subclassing
    * is to allow fast path for nonfair version.
    */
   abstract void lock();

   /**
    * Performs non-fair tryLock.  tryAcquire is implemented in
    * subclasses, but both need nonfair try for trylock method.
    */
   final boolean nonfairTryAcquire(int acquires) {
       final Thread current = Thread.currentThread();
       int c = getState();
       if (c == 0) {
           if (compareAndSetState(0, acquires)) {
               setExclusiveOwnerThread(current);
               return true;
           }
       }
       else if (current == getExclusiveOwnerThread()) {
           int nextc = c + acquires;
           if (nextc < 0) // overflow
               throw new Error("Maximum lock count exceeded");
           setState(nextc);
           return true;
       }
       return false;
   }

   protected final boolean tryRelease(int releases) {
       int c = getState() - releases;
       if (Thread.currentThread() != getExclusiveOwnerThread())
           throw new IllegalMonitorStateException();
       boolean free = false;
       if (c == 0) {
           free = true;
           setExclusiveOwnerThread(null);
       }
       setState(c);
       return free;
   }

   protected final boolean isHeldExclusively() {
       // While we must in general read state before owner,
       // we don't need to do so to check if current thread is owner
       return getExclusiveOwnerThread() == Thread.currentThread();
   }

   final ConditionObject newCondition() {
       return new ConditionObject();
   }

   // Methods relayed from outer class

   final Thread getOwner() {
       return getState() == 0 ? null : getExclusiveOwnerThread();
   }

   final int getHoldCount() {
       return isHeldExclusively() ? getState() : 0;
   }

   final boolean isLocked() {
       return getState() != 0;
   }

   /**
    * Reconstitutes the instance from a stream (that is, deserializes it).
    */
   private void readObject(java.io.ObjectInputStream s)
       throws java.io.IOException, ClassNotFoundException {
       s.defaultReadObject();
       setState(0); // reset to unlocked state
   }
}
```
诶？看到这里有一个疑问，这是一个抽象类，不能实例化，那怎么用？并且，没有看到`tryAcquire`方法？没错，`Sync`是`ReentrantLock`中对于AQS的抽象实现，真正的实现是`Sync`的两个实现类`NonfairSync`和`FairSync`，他们分别对应了不公平的锁的获取方式和公平的锁的获取方式。

这里提到了公平锁和非公平锁，什么是公平锁？什么是非公平锁。

公平锁是指在加锁之前，是否有排队等待的线程，优先排队等待的线程，先来先得。

而非公平锁是指加锁时不考虑排队等待问题，直接尝试获取锁，获取不到自动到队尾等待。

下面是知乎上对于公平锁和非公平锁的一个回答：
> 基于AQS的锁(比如ReentrantLock)原理大体是这样:
> 有一个state变量，初始值为0，假设当前线程为A,每当A获取一次锁，status++. 释放一次，status--.锁会记录当前持有的线程。
> 当A线程拥有锁的时候，status>0. B线程尝试获取锁的时候会对这个status有一个CAS(0,1)的操作，尝试几次失败后就挂起线程，进入一个等待队列。
> 如果A线程恰好释放，--status==0, A线程会去唤醒等待队列中第一个线程，即刚刚进入等待队列的B线程，B线程被唤醒之后回去检查这个status的值，尝试CAS(0,1),而如果这时恰好C线程也尝试去争抢这把锁。
> 此时非公平锁实现：C直接尝试对这个status CAS(0,1)操作，并成功改变了status的值，B线程获取锁失败，再次挂起，这就是非公平锁，B在C之前尝试获取锁，而最终是C抢到了锁。
> 公平锁：C发现有线程在等待队列，直接将自己进入等待队列并挂起,B获取锁
> 链接：https://www.zhihu.com/question/36964449/answer/69843172

## static final class NonfairSync extends Sync{}
对于`AQS`独占策略、非公平性的锁的获取，定义如下：
```java
static final class NonfairSync extends Sync {
     private static final long serialVersionUID = 7316153563782823691L;

     /**
      * Performs lock.  Try immediate barge, backing up to normal
      * acquire on failure.
      */
     final void lock() {
         if (compareAndSetState(0, 1))
             setExclusiveOwnerThread(Thread.currentThread());
         else
             acquire(1);
     }

     protected final boolean tryAcquire(int acquires) {
         return nonfairTryAcquire(acquires);
     }
 }
```
这个实现类中能看到`tryAcquire`的实现，它覆盖了超父类AQS的`tryAcquire`方法。

## static final class FairSync extends Sync{}
对于`Sync`的实现，与`NonfairSync`相反，这是"公平锁"的实现，定义如下：
```java
static final class FairSync extends Sync {
    private static final long serialVersionUID = -3000897897090466540L;

    final void lock() {
        acquire(1);
    }
    /**
     * Fair version of tryAcquire.  Don't grant access unless
     * recursive call or no waiters or is first.
     */
    protected final boolean tryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        int c = getState();
        if (c == 0) {
            if (!hasQueuedPredecessors() &&
                compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        else if (current == getExclusiveOwnerThread()) {
            int nextc = c + acquires;
            if (nextc < 0)
                throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }
        return false;
    }
}
```
可以看到的是，这个采取“公平”的策略，重写了`tryAcquire`方法，

## 构造函数
有两个重载的构造函数，无参数的构造方法定义如下：
```Java
public ReentrantLock() {
   sync = new NonfairSync();
}

private final Sync sync;
```
可以看到，无参数的构造方法创建的`ReenrantLock`对象默认采取的是独占的非公平锁。

另外接受一个boolean类型参数的`ReenrantLock`构造方法，定义如下：
```Java
public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}
```
这里可进行自定义的创建规则，当参数是true的时候创建的是公平锁。

### public void lock{...}
重写了`Lock`的`lock()`方法，定义如下：
```Java
public void lock() {
    sync.lock();
}
```

## public void lockInterruptibly() throws InterruptedException{...}

重写了`Lock`接口中的方法，定义如下：
```Java
public void lockInterruptibly() throws InterruptedException {
    sync.acquireInterruptibly(1);
}
```

## tryLock方法
重载了两个方法，同样都是依靠`sync`对象实现的，无参的方法定义如下：
```Java
public boolean tryLock() {
    return sync.nonfairTryAcquire(1);
}
```
有参的方法定义如下：
```Java
public boolean tryLock(long timeout, TimeUnit unit)
       throws InterruptedException {
   return sync.tryAcquireNanos(1, unit.toNanos(timeout));
}
```

## public void unlock(){...}
```Java
public void unlock() {
    sync.release(1);
}
```

## public Condition newCondition(){...}
```Java
public Condition newCondition() {
    return sync.newCondition();
}
```

## ReentrantLock的使用
`synchronized`隐式获得和释放锁，而`ReentrantLock`将锁作为了一个Java对象，需要手动获得和手动释放。另外，`synchronized`都是公平锁，而我们可以在创建`ReentrantLock`的时候选择锁的公平性。
```java
Lock lock1 = new ReentrantLock(); //默认为不公平锁
Lock lock2 = new ReentrantLock(true); //声明为公平锁，但是效率比不同锁要低
```


`ReentrantLock`可以适用于以下适用场景：
1. 防止重入执行（忽略重复触发）
```java
  Lock lock = new ReentrantLock();
  if(lock.tryLock()){ //如果锁已经被lock了，那么立即返回false,达到忽略操作的效果
    try{
      //操作
    }finally{
      lock.unlock();//这一步一定不要忘记了！！！
    }
  }
```
这种方法使用与防止重入执行，比如一个定时任务,第一次定时任务未完成,重复发起了第二次,直接返回flase。再例如用在界面交互时点击执行较长时间请求操作时，防止多次点击导致后台重复执行。

2. 同步执行，类似于synchronized
```java
Lock lock = new ReentrantLock();
lock.lock(); //如果已经被其他资源锁定，那么线程会在这里等待，达到暂停的效果
try{
  //操作
}finally{
    lock.unlock();//一定不要忘记！！！
}
```
3. 尝试等待执行
```java
ReentrantLock lock = new ReentrantLock(true); //公平锁  
try {  
    if (lock.tryLock(5, TimeUnit.SECONDS)) {      
        //如果已经被lock，尝试等待5s，看是否可以获得锁，如果5s后仍然无法获得锁则返回false继续执行  
        try {  
            //操作  
        } finally {  
            lock.unlock();  
        }  
    }  
}catch (InterruptedException e) {  
    e.printStackTrace(); //当前线程被中断时(interrupt)，会抛InterruptedException                   
}
```
这种方法如果发现该操作正在执行,等待一段时间，如果规定时间未得到锁，放弃。防止资源处理不当，线程队列溢出,出现死锁。

4. 可中断锁的同步执行
```java
public ReentrantLockTest{
    private static Lock reenT = new ReentrantLock(); //不公平锁
    public static void lockInterruptTest(){
        try {
            reenT.lockInterruptibly();
            //操作
            System.out.println("aaaa"+Thread.currentThread().getName());
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            reenT.unlock();
        }
    }
    public static void main(String[] args) {
    //同时启动1000个线程，去进行i++计算，看看实际结果
        for (int i = 0; i < 1000; i++) {
            new Thread(new Runnable() {
                @Override
                public void run() {
                    ReentrantLockTest.tryLockTest();
                }
            }).start();
        }
    }
}
```
对于同步锁，一定要注意的是，要在finally块中手动释放锁！！！如果没有释放，等于是在程序中埋下了一颗定时炸弹。



参考
* [ReentrantLock实现原理深入探究](https://www.cnblogs.com/xrq730/p/4979021.html)
* [ReentrantLock的使用](http://chenjumin.iteye.com/blog/2182168)
* [Java中的公平锁和非公平锁实现详解](http://blog.csdn.net/qyp199312/article/details/70598480)
