# ReentrantLock

`Reentrant`这个单词的意思是“可重入”，所以`ReentrantLock`的意思就是"可重入锁"。它允许把锁定的实现作为Java类，而不是作为语言的特性来实现。它除了在扩展性上与同步方法和声明的隐式监控锁不同外，在基本行为和语义上都相同。还支持取锁手的公平与不公平的选择。

> 这里提到了两个陌生的概念。
> 1. 什么是“可重入”？ 可冲入的锁即表示该锁能够支持一个线程对资源的重复加锁。实际上`synchorinized`也支持隐式重进入，而对于`ReentrantLock`而言，对于已经获取到锁的现场，再次调用`lock()`方法时可以获取锁而不被阻塞。
> 2. 什么是公平获取锁？什么是不公平获取锁。**如果在绝对时间上，先对于锁进行获取请求一定先被满足，那么这个锁就是公平的，反之就是不公平**。`ReentrantLock`可以通过构造函数来控制到底是公平锁还是不公平锁。实际上非公平锁的消息源源大于公平锁。

类结构如下：

![ReentrantLock](http://ovn0i3kdg.bkt.clouddn.com/ReenrantLock.png)



一个拥有可重入锁的线程最后成功锁定，但尚未解锁它。当这个锁并没有被其他线程拥有时，一个线程调用锁，将成功的返回请求的这个锁。如果当前线程已经拥有该锁了，该方法会立刻返回。这可以通过`isHeldByCurrentThread()`和`getHoldCount()`两个方法检查。


### public class ReentrantLock extends Object implements Lock, Serializable
类声明，可以看到类继承`Object`，实现了`Lock`接口和`Serializable`接口，所以就会有一个`serialVersionUID`属性（这个就不提了）和重写的接口的方法。


### abstract static class Sync extends AbstractQueuedSynchronizer{}
这是一个静态抽象内部类，非常重要。其定义如下：
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

### static final class NonfairSync extends Sync{}
这是静态抽象内部类的一种实现，其实例就是上面提到的“非公平锁”，定义如下：
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

### static final class FairSync extends Sync{}
这是静态抽象内部类的另一种实现，与`NonfairSync`相反，它其实就是上面提到的"公平锁"，定义如下：
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

### 构造函数
有两个重载的构造函数，无参数的构造方法定义如下：
```Java
public ReentrantLock() {
   sync = new NonfairSync();
}
```
其中同步器`sync`是`ReentrantLock`的一个私有属性，定义如下：
```Java
private final Sync sync;
```
可以看到这个属性被final修饰，说明不能这个属性不能被中途改变。属性所属类`Sync`是一个静态抽象的内部类，定义如下：
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
可以看到这个同步类继承了`AbstractQueuedSynchronizer`父类，暂时还没有学到这个类是干什么用的，反正现在可以知道，它默认的赋值是一个`NonfairSync`类的实例，`NonfairSync`从名字就可以看出来，它就是我们之前说到的非公平锁，这个类的定义如下：
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
另外还有一个有参数的`ReenrantLock`构造方法，定义如下：
```Java
public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}
```
这个构造方法就给你选择了，到底选择公不公平就依靠参数判断，如果选择公平，就给`sync`赋值为一个`FairSync`的实例，这个类的定义如下：
```Java
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

`FairSync`和`NonfairSync`是非常重要的两个类，基本上`ReenrantLock`的基本实现都是它实现的。

#### public void lock{...}
重写了`Lock`的`lock()`方法，定义如下：
```Java
public void lock() {
    sync.lock();
}
```

### public void lockInterruptibly() throws InterruptedException{...}

重写了`Lock`接口中的方法，定义如下：
```Java
public void lockInterruptibly() throws InterruptedException {
    sync.acquireInterruptibly(1);
}
```

### tryLock方法
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

### public void unlock(){...}
```Java
public void unlock() {
    sync.release(1);
}
```

### public Condition newCondition(){...}
```Java
public Condition newCondition() {
    return sync.newCondition();
}
```
