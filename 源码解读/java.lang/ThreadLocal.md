# ThreadLocal

>这篇文章的解释非常通俗易懂 [线程的私家小院儿：ThreadLocal](https://mp.weixin.qq.com/s?__biz=MzA3MDExNzcyNA==&mid=2650392118&idx=1&sn=a2144a19bdeba48001f4f76f423e25d9&scene=0#wechat_redirect)

> 可以这么想，每一个`Thread`实例中都保存了一个map，这个map叫做`ThreadLocalMap`，和我们一般用的`HashMap`不太一样，但是总之就是一个键值对，我们可以往这个map中加入一点东西，即map.put(k, value)，我们给这个key定义了一个类，叫做`ThreadLocal`，到这个地步，我们是可以操作的，比如想要从一个线程中取得一个名叫"context"的`ThreadLocal`类型变量对象的值，那我们就可以通过下面的方式取得：
> ```Java
> Thread thread = Thread.currentThread();
> ThreadLocalMap<ThreadLocal, object> map = thred.getThreadLocalMap();
> Object o = map.get(context);
> //进行类型转换
> ```
> 以上是取值过程，设值过程的前两步也是这样，是不是很繁琐？所以，为了简化取值的步骤，我们将上面的过程打包进了`ThreadLocal`中，取值和设值过程分别定义为`get`和`set`方法。

`ThreadLocal`类结果如下：

![ThreadLocal](http://ovn0i3kdg.bkt.clouddn.com/ThreadLocal.png)

`ThreadLocal`类用来提供线程内部的局部变量。这种变量在多线程环境下访问(通过get或set方法访问)时能保证各个线程里的变量相对独立于其他线程内的变量。`ThreadLocal`实例通常来说都是`private static`类型的，用于关联线程和线程的上下文。

可以总结为一句话：`ThreadLocal`的作用是提供线程内的局部变量，这种变量在线程的生命周期内起作用，减少同一个线程内多个函数或者组件之间一些公共变量的传递的复杂度。

## 构造函数
里面什么都没有。
```java
/**
 * Creates a thread local variable.
 * @see #withInitial(java.util.function.Supplier)
 */
public ThreadLocal() {
}
```

## get
```java
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}

ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}

private Entry getEntry(ThreadLocal<?> key) {
    int i = key.threadLocalHashCode & (table.length - 1);
    Entry e = table[i];
    if (e != null && e.get() == key)
        return e;
    else
        return getEntryAfterMiss(key, i, e);
}

private T setInitialValue() {
    T value = initialValue();
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
    return value;
}
```

## set方法
```Java
/**
 * Sets the current thread's copy of this thread-local variable
 * to the specified value.  Most subclasses will have no need to
 * override this method, relying solely on the {@link #initialValue}
 * method to set the values of thread-locals.
 *
 * @param value the value to be stored in the current thread's copy of
 *        this thread-local.
 */
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}
void createMap(Thread t, T firstValue) {
   t.threadLocals = new ThreadLocalMap(this, firstValue);
}
```

## remove方法
```Java
public void remove() {
   ThreadLocalMap m = getMap(Thread.currentThread());
   if (m != null)
       m.remove(this);
}

private void remove(ThreadLocal<?> key) {
    Entry[] tab = table;
    int len = tab.length;
    int i = key.threadLocalHashCode & (len-1);
    for (Entry e = tab[i];
         e != null;
         e = tab[i = nextIndex(i, len)]) {
        if (e.get() == key) {
            e.clear();
            expungeStaleEntry(i);
            return;
        }
    }
}
```

## ThreadLocalMap
这是`ThreadLocal`的内部类，虽然是map，但是和我们一般用的 `ThreadLocalMap`不一样。

首先看到它其中的属性：
```Java
/**
   * The initial capacity -- MUST be a power of two.
   */
  private static final int INITIAL_CAPACITY = 16;

  /**
   * The table, resized as necessary.
   * table.length MUST always be a power of two.
   */
  private Entry[] table;

  /**
   * The number of entries in the table.
   */
  private int size = 0;

  /**
   * The next size value at which to resize.
   */
  private int threshold; // Default to 0
```
其中有些概念还是非常相似的，比如初始容量是16，然后扩容阈值，元素个数和哈希数组。
这里我们看到数组元素是`Entry`类型，该类型的定义如下：
```Java
static class Entry extends WeakReference<ThreadLocal<?>> {
    /** The value associated with this ThreadLocal. */
    Object value;

    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}
```
它继承的是`WeakReference`类，该类定义如下：
```Java

public class WeakReference<T> extends Reference<T> {

    /**
     * Creates a new weak reference that refers to the given object.  The new
     * reference is not registered with any queue.
     *
     * @param referent object the new weak reference will refer to
     */
    public WeakReference(T referent) {
        super(referent);
    }

    /**
     * Creates a new weak reference that refers to the given object and is
     * registered with the given queue.
     *
     * @param referent object the new weak reference will refer to
     * @param q the queue with which the reference is to be registered,
     *          or <tt>null</tt> if registration is not required
     */
    public WeakReference(T referent, ReferenceQueue<? super T> q) {
        super(referent, q);
    }
}
```
从名字上看，这是一个“弱引用”，实现自`Reference`。Java中有四种引用，具体可以参见相关博文。

总之，这是一个map，有点不一样而已。我们暂且不深究弱引用和强引用有什么区别，先来看看这个map的一些操作：

### 构造方法
```Java
//构造函数
ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
    table = new Entry[INITIAL_CAPACITY];
    int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
    table[i] = new Entry(firstKey, firstValue);
    size = 1;
    setThreshold(INITIAL_CAPACITY);
}

private ThreadLocalMap(ThreadLocalMap parentMap) {
    Entry[] parentTable = parentMap.table;
    int len = parentTable.length;
    setThreshold(len);
    table = new Entry[len];

    for (int j = 0; j < len; j++) {
        Entry e = parentTable[j];
        if (e != null) {
            @SuppressWarnings("unchecked")
            ThreadLocal<Object> key = (ThreadLocal<Object>) e.get();
            if (key != null) {
                Object value = key.childValue(e.value);
                Entry c = new Entry(key, value);
                int h = key.threadLocalHashCode & (len - 1);
                while (table[h] != null)
                    h = nextIndex(h, len);
                table[h] = c;
                size++;
            }
        }
    }
}
```
### Set方法
采取线性探测法解决哈希冲突。
```java
private void set(ThreadLocal<?> key, Object value) {

      // We don't use a fast path as with get() because it is at
      // least as common to use set() to create new entries as
      // it is to replace existing ones, in which case, a fast
      // path would fail more often than not.

      Entry[] tab = table;
      int len = tab.length;
      int i = key.threadLocalHashCode & (len-1);

      for (Entry e = tab[i];
           e != null;
           e = tab[i = nextIndex(i, len)]) {
          ThreadLocal<?> k = e.get();

          if (k == key) {
              e.value = value;
              return;
          }

          if (k == null) {
              replaceStaleEntry(key, value, i);
              return;
          }
      }

      tab[i] = new Entry(key, value);
      int sz = ++size;
      if (!cleanSomeSlots(i, sz) && sz >= threshold)
          rehash();
  }
```
### get方法
```java
private Entry getEntry(ThreadLocal<?> key) {
     int i = key.threadLocalHashCode & (table.length - 1);
     Entry e = table[i];
     if (e != null && e.get() == key)
         return e;
     else
         return getEntryAfterMiss(key, i, e);
 }
//当发生哈希冲突的时候，将采取线性探测法进行查找
 private Entry getEntryAfterMiss(ThreadLocal<?> key, int i, Entry e) {
    Entry[] tab = table;
    int len = tab.length;

    while (e != null) {
        ThreadLocal<?> k = e.get();
        if (k == key)
            return e;
        if (k == null)
            expungeStaleEntry(i);
        else
            i = nextIndex(i, len);
        e = tab[i];
    }
    return null;
}
```

## 内存泄漏
> 参考 [深入分析 ThreadLocal 内存泄漏问](http://www.importnew.com/22039.html)

让我们重新回顾一下`ThreadLocal`原理

![ThreadLocal原理](http://ovn0i3kdg.bkt.clouddn.com/ThreadLocal%E5%8E%9F%E7%90%86.png)

`ThreadLocalMap`使用`ThreadLocal`的弱引用作为`key`，如果一个`ThreadLocal`没有外部强引用来引用它，那么系统 GC 的时候，这个`ThreadLocal`势必会被回收，这样一来，`ThreadLocalMap`中就会出现key为null的`Entry`，就没有办法访问这些key为null的Entry的value，如果当前线程再迟迟不结束的话，这些key为null的Entry的value就会一直存在一条强引用链：`Thread Ref -> Thread -> ThreaLocalMap -> Entry -> value`永远无法回收，造成内存泄漏。

其实，`ThreadLocalMap`的设计中已经考虑到这种情况，也加上了一些防护措施：在`ThreadLocal`的`get()`,`set()`,`remove()`的时候都会清除线程`ThreadLocalMap`里所有key为null的value。

但是这些被动的预防措施并不能保证不会内存泄漏：

* 使用static的ThreadLocal，延长了ThreadLocal的生命周期，可能导致的内存泄漏（参考[ThreadLocal 内存泄露的实例分析](http://blog.xiaohansong.com/2016/08/09/ThreadLocal-leak-analyze/)）。
* 分配使用了ThreadLocal又不再调用get(),set(),remove()方法，那么就会导致内存泄漏。


**每次使用完ThreadLocal，都调用它的remove()方法，清除数据。**
