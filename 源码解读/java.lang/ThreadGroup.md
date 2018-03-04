# ThreadGroup
在学习到`Thread`类的时候，发现线程实际上分组，即每个线程被创建的时候，会有一个属性`threadgroup`来表示这个线程是属于哪个组的，如果没有指定，就跟父线程同组。那么这个“线程组”到底是什么作用的呢？今天来学习来这个类，首先看结构：

![ThreadGroup](http://ovn0i3kdg.bkt.clouddn.com/Thread_group.png)

> 有一些基本的get set方法的实现其实很简单，就不说了。


## public class ThreadGroup implements Thread.UncaughtExceptionHandler
这个类声明，可以看到类实现了`Thread`类的一个内部接口，定义如下：
```java
@FunctionalInterface
public interface UncaughtExceptionHandler {
    /**
     * Method invoked when the given thread terminates due to the
     * given uncaught exception.
     * <p>Any exception thrown by this method will be ignored by the
     * Java Virtual Machine.
     * @param t the thread
     * @param e the exception
     */
    void uncaughtException(Thread t, Throwable e);
}
```
这是一个函数式接口，从注释部分可以看到通过这个方法抛出的所有异常都会被虚拟机忽略。为什么忽略？不知道。

源码中对该类做出了解释：线程组代表了线程的集合。一个线程组也可以包含另一个线程组，也就是说**线程组和线程组组成了树，而线程和线程组是元素和集合的关系**。除了初始线程组，每个线程组都有父母。线程组可以访问组内线程的信息，但是组内线程，只允许访问有关自己线程的信息，不允许访问其所在线程组的、父线程组和其他线程组信息。

## 构造方法
一共定义了四种构造方法。
### private ThreadGroup(){...}
无参构造方法，定义如下：
```java
private ThreadGroup() {     // called from C code
   this.name = "system";
   this.maxPriority = Thread.MAX_PRIORITY;
   this.parent = null;
}
```
这个方法创建了一个新的空的线程组，可以看到，线程组的名字叫做“system”，它的最高优先级是线程的最高优先级，即10。而父线程组是null。三个变量的声明如下：
```java
private final ThreadGroup parent;
String name;
int maxPriority;
```
这个方法是C语言调用的，创建的就是前文提到过的"初始线程组"。


### public ThreadGroup(String name){...}
指定了线程组的名称name，定义如下：
```java
public ThreadGroup(String name) {
    this(Thread.currentThread().getThreadGroup(), name);
}
```
可以看到，它取得了当前线程所属的线程组作为其父线程。调用的是另一个重载的方法(就是下一个介绍的)，可能会抛出`SecurityException`异常。


### public ThreadGroup(ThreadGroup parent, String name){...}
这就是上面提到的重载的构造函数，定义如下：
```java
public ThreadGroup(ThreadGroup parent, String name) {
    this(checkParentAccess(parent), parent, name);
}
```
这里调用的是下一个重载的构造方法了，第一个参数是`checkParentAccess`方法的返回值了，这个方法定义如下：
```java
private static Void checkParentAccess(ThreadGroup parent) {
    parent.checkAccess();
    return null;
}
```
其中的`checkAccess`定义如下：
```java
public final void checkAccess() {
    SecurityManager security = System.getSecurityManager();
    if (security != null) {
        security.checkAccess(this);
    }
}
```
这个方法决定了当前运行的线程是否有修改线程组的权利。该方法可能会抛出`SecurityException`异常。再回到`checkParentAccess`这个方法，注意它的返回值是`Void`,不是`void`。 这个方法是用来可达性的。

### private ThreadGroup(Void unused, ThreadGroup parent, String name){...}
这个构造方法就是最终的大BOSS了。定义如下：
```java
private ThreadGroup(Void unused, ThreadGroup parent, String name) {
     this.name = name;
     this.maxPriority = parent.maxPriority;
     this.daemon = parent.daemon;
     this.vmAllowSuspension = parent.vmAllowSuspension;
     this.parent = parent;
     parent.add(this);
 }
```
当指定了父线程组的时候，其`maxPriority`、`daemon`、`vmAllowSuspension`就和父线程组保持一致了，`vmAllowSuspension`看名字就知道含义是虚拟机允许挂起。最后调用了`add`方法，定义如下：
```java
private final void add(ThreadGroup g){
    synchronized (this) {
        if (destroyed) {
            throw new IllegalThreadStateException();
        }
        if (groups == null) {
            groups = new ThreadGroup[4];
        } else if (ngroups == groups.length) {
            groups = Arrays.copyOf(groups, ngroups * 2);
        }
        groups[ngroups] = g;

        // This is done last so it doesn't matter in case the
        // thread is killed
        ngroups++;
    }
}
```
其中`destroyed`就是用来表示线程组的状态的，是不是被销毁了？（诶，线程组被销毁了难道不是真正被销毁了？）如果这个线程组已经是被销毁的状态，那就再不能往里装东西了，抛出异常。如果还没有初始化，即`ThreadGroup groups[]`这个数组还没初始化呢，那就初始化被，可以看到，它首次分配了4个空间，然后再往里面装东西，有一个`ngroups`量来标识其中的个数。被分配的空间都装满了怎么办？扩容。扩多少？两倍！其实`groups`数组和`ngroups`合起来就是一个线性表。

## public final void setMaxPriority(int pri){...}

```java
public final void setMaxPriority(int pri) {
    int ngroupsSnapshot;
    ThreadGroup[] groupsSnapshot;
    synchronized (this) {
        checkAccess();
        if (pri < Thread.MIN_PRIORITY || pri > Thread.MAX_PRIORITY) {
            return;
        }
        maxPriority = (parent != null) ? Math.min(pri, parent.maxPriority) : pri;
        ngroupsSnapshot = ngroups;
        if (groups != null) {
            groupsSnapshot = Arrays.copyOf(groups, ngroupsSnapshot);
        } else {
            groupsSnapshot = null;
        }
    }
    for (int i = 0 ; i < ngroupsSnapshot ; i++) {
        groupsSnapshot[i].setMaxPriority(pri);
    }
}
```
这个用来设置线程组的优先级。首先判断了一些可达性（其实我不知道这个是干什么用的），如果参数不合法，即超出了范围（小于1或大于10），那么设置是不成功的，优先级不变，否则将会取参数和父线程的最大优先级的较小值赋值给当前线程组的`maxPriority`属性，如果当前线程是初始线程，那就没什么比较了，直接设置就行了。注意到，这个方法是递归性的。也就是说，如果一个线程组被重新设定了优先级，其子线程组都会跟着变。那问题来了，会影响其中线程的优先级么？答案是不会，哪些已经有了优先级的线程的优先级不会跟着改变，但是会影响在这个时间点之后被创建出来的、属于这个线程组的线程，还记得`Thread`类新建线程的时候，新线程的优先级不会大于其所在线程组的`maxPriority`属性值。所以自创一道题目：线程组中的线程的优先级有可能大于线程组的最大优先级么？现在看来是有可能的。

> 这里有一个问题，为什么要有复制一份`groupsSnapshot`,直接对`groups`操作不是挺好的么？注意，Arrays.copyOf()是浅复制，`groupsSnapshot`和`groups`中相应元素实际上是一样的。

## public final boolean parentOf(ThreadGroup g){...}
从方法名字就可以看出来，这个方法用来判断这个线程组是不是参数的父线程组。这里说"父线程组"不太准确，应该说“祖先线程组”。
```java
public final boolean parentOf(ThreadGroup g) {
   for (; g != null ; g = g.parent) {
       if (g == this) {
           return true;
       }
   }
   return false;
}
```
从方法定义就可以看出来，形参一直找寻着其父线程组，直到初始线程。


## public int activeCount(){...}
返回此线程中活动线程数的估计组及其子组。 递归迭代所有子线程组。需要注意的是，这里返回的是所有活动的**线程**的数量，要和后面`activeGroupCount`方法区分开来。
```java
public int activeCount() {
    int result;
    // Snapshot sub-group data so we don't hold this lock
    // while our children are computing.
    int ngroupsSnapshot;
    ThreadGroup[] groupsSnapshot;
    synchronized (this) {
        if (destroyed) {
            return 0;
        }
        result = nthreads;
        ngroupsSnapshot = ngroups;
        if (groups != null) {
            groupsSnapshot = Arrays.copyOf(groups, ngroupsSnapshot);
        } else {
            groupsSnapshot = null;
        }
    }
    for (int i = 0 ; i < ngroupsSnapshot ; i++) {
        result += groupsSnapshot[i].activeCount();
    }
    return result;
}
```
源码对该方法有一个说明：返回的值只是一个估计，因为数量线程可以动态更改，而此方法遍历内部数据结构，并可能受到某些存在的影响系统线程。 此方法主要用于调试和监测目的。

## enumerate方法
这又是一个比较重要的方法，一共有6个重载方法。其中一般的作用是复制当前线程的线程组及其子组中的每一个活动**线程**到指定的数组。另一半的是复制当前线程的线程组及其子组中的每一个活动**线程组**到指定的数组。 由于过程都差不多，这里就看下前一半。

### public int enumerate(Thread list[]){...}
参数list就是制定的数组。最后被复制进去的线程组或活动线程的总个数。
```java
public int enumerate(Thread list[]) {
    checkAccess();
    return enumerate(list, 0, true);
}
```

### public int enumerate(Thread list[], boolean recurse) {...}
```java
public int enumerate(Thread list[], boolean recurse) {
    checkAccess();
    return enumerate(list, 0, recurse);
}
```

### private int enumerate(Thread list[], int n, boolean recurse){...}
这个此时最终的方法。三个参数，第二个参数都被默认为0，第三个参数recurse如果是true，那么将会递归调用这个子线程组下的所有子线程组，如果超出了list的大小，那么多出的部分将被忽略。如果避免这个问题呢？可以首先调用一些`activeCount`方法确定一下大概的属性，然后再用这个大小声明list数组就可以了。

```java
private int enumerate(Thread list[], int n, boolean recurse) {
    int ngroupsSnapshot = 0;
    ThreadGroup[] groupsSnapshot = null;
    synchronized (this) {
        if (destroyed) {
            return 0;
        }
        int nt = nthreads;
        if (nt > list.length - n) {
            nt = list.length - n;
        }
        for (int i = 0; i < nt; i++) {
            if (threads[i].isAlive()) {
                list[n++] = threads[i];
            }
        }
        if (recurse) {
            ngroupsSnapshot = ngroups;
            if (groups != null) {
                groupsSnapshot = Arrays.copyOf(groups, ngroupsSnapshot);
            } else {
                groupsSnapshot = null;
            }
        }
    }
    if (recurse) {
        for (int i = 0 ; i < ngroupsSnapshot ; i++) {
            n = groupsSnapshot[i].enumerate(list, n, true);
        }
    }
    return n;
}
```

## public int activeGroupCount(){...}
这个方法返回的是这个线程组下所有活动线程组的个数。注意是**线程组**的个数。与上面`activeCount`区分开来。
```java
public int activeGroupCount() {
    int ngroupsSnapshot;
    ThreadGroup[] groupsSnapshot;
    synchronized (this) {
        if (destroyed) {
            return 0;
        }
        ngroupsSnapshot = ngroups;
        if (groups != null) {
            groupsSnapshot = Arrays.copyOf(groups, ngroupsSnapshot);
        } else {
            groupsSnapshot = null;
        }
    }
    int n = ngroupsSnapshot;
    for (int i = 0 ; i < ngroupsSnapshot ; i++) {
        n += groupsSnapshot[i].activeGroupCount();
    }
    return n;
}
```

## public final void interrupt(){...}
中断这个线程组中所有的线程。同样采取了递归的方式。

```java
public final void interrupt() {
    int ngroupsSnapshot;
    ThreadGroup[] groupsSnapshot;
    synchronized (this) {
        checkAccess();
        for (int i = 0 ; i < nthreads ; i++) {
            threads[i].interrupt();
        }
        ngroupsSnapshot = ngroups;
        if (groups != null) {
            groupsSnapshot = Arrays.copyOf(groups, ngroupsSnapshot);
        } else {
            groupsSnapshot = null;
        }
    }
    for (int i = 0 ; i < ngroupsSnapshot ; i++) {
        groupsSnapshot[i].interrupt();
    }
}
```

## private boolean stopOrSuspend(boolean suspend){...}
从方法名字可以看出，这个方法来停止或停止当前线程组中的除了当前线程之外的所有线程，那到底是挂起还是停止呢，参数`suspend`如果是true，就表示全部挂起，否则就全部停止。方法返回true当且仅当当前线程属于这个线程组或其子线程组。
```java
@SuppressWarnings("deprecation")
private boolean stopOrSuspend(boolean suspend) {
   boolean suicide = false;
   Thread us = Thread.currentThread();
   int ngroupsSnapshot;
   ThreadGroup[] groupsSnapshot = null;
   synchronized (this) {
       checkAccess();
       for (int i = 0 ; i < nthreads ; i++) {
           if (threads[i]==us)
               suicide = true;
           else if (suspend)
               threads[i].suspend();
           else
               threads[i].stop();
       }

       ngroupsSnapshot = ngroups;
       if (groups != null) {
           groupsSnapshot = Arrays.copyOf(groups, ngroupsSnapshot);
       }
   }
   for (int i = 0 ; i < ngroupsSnapshot ; i++)
       suicide = groupsSnapshot[i].stopOrSuspend(suspend) || suicide;

   return suicide;
}
```

## public final void destroy(){...}
用来销毁线程组(除初始线程组)其子线程组，前提条件是该线程组中`thread`必须得空，意思就是如果该线程组中还要运行着的线程，那么这个线程组就不能被销毁。销毁过程就是将各个属性置为null或0。最后将其从父线程组中移除。
```java
public final void destroy() {
   int ngroupsSnapshot;
   ThreadGroup[] groupsSnapshot;
   synchronized (this) {
       checkAccess();
       if (destroyed || (nthreads > 0)) {
           throw new IllegalThreadStateException();
       }
       ngroupsSnapshot = ngroups;
       if (groups != null) {
           groupsSnapshot = Arrays.copyOf(groups, ngroupsSnapshot);
       } else {
           groupsSnapshot = null;
       }
       if (parent != null) {
           destroyed = true;
           ngroups = 0;
           groups = null;
           nthreads = 0;
           threads = null;
       }
   }
   for (int i = 0 ; i < ngroupsSnapshot ; i += 1) {
       groupsSnapshot[i].destroy();
   }
   if (parent != null) {
       parent.remove(this);
   }
}
```
其中`remove`方法定义如下：
```java
private void remove(ThreadGroup g) {
    synchronized (this) {
        if (destroyed) {
            return;
        }
        for (int i = 0 ; i < ngroups ; i++) {
            if (groups[i] == g) {
                ngroups -= 1;
                System.arraycopy(groups, i + 1, groups, i, ngroups - i);
                // Zap dangling reference to the dead group so that
                // the garbage collector will collect it.
                groups[ngroups] = null;
                break;
            }
        }
        if (nthreads == 0) {
            notifyAll();
        }
        if (daemon && (nthreads == 0) &&
            (nUnstartedThreads == 0) && (ngroups == 0))
        {
            destroy();
        }
    }
}
```

## list方法
打印出线程组中线程的相关信息，这个方法只在debugging的时候有用。有两个重载方法。无参数方法定义如下：
```java
public void list() {
    list(System.out, 0);
}
```
默认的输出是`System.out`，缩进为0。被调用的方法定义如下：
```java
void list(PrintStream out, int indent) {
    int ngroupsSnapshot;
    ThreadGroup[] groupsSnapshot;
    synchronized (this) {
        for (int j = 0 ; j < indent ; j++) {
            out.print(" ");
        }
        out.println(this);
        indent += 4;
        for (int i = 0 ; i < nthreads ; i++) {
            for (int j = 0 ; j < indent ; j++) {
                out.print(" ");
            }
            out.println(threads[i]);
        }
        ngroupsSnapshot = ngroups;
        if (groups != null) {
            groupsSnapshot = Arrays.copyOf(groups, ngroupsSnapshot);
        } else {
            groupsSnapshot = null;
        }
    }
    for (int i = 0 ; i < ngroupsSnapshot ; i++) {
        groupsSnapshot[i].list(out, indent);
    }
}
```

## public void uncaughtException(Thread t, Throwable e){...}
设置当前线程组的异常处理器（只对没有异常处理器的线程有效）。
```java
public void uncaughtException(Thread t, Throwable e) {
     if (parent != null) {
         parent.uncaughtException(t, e);
     } else {
         Thread.UncaughtExceptionHandler ueh =
             Thread.getDefaultUncaughtExceptionHandler();
         if (ueh != null) {
             ueh.uncaughtException(t, e);
         } else if (!(e instanceof ThreadDeath)) {
             System.err.print("Exception in thread \""
                              + t.getName() + "\" ");
             e.printStackTrace(System.err);
         }
     }
 }
```

## public String toString() {...}
打印出线程组信息。
```java
public String toString() {
    return getClass().getName() + "[name=" + getName() + ",maxpri=" + maxPriority + "]";
}
```

> 如何创建一个线程并且加入到一个指定的线程组中？b
