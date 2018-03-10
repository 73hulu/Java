# ArrayBlockingQueue


`ArrayBlockingQueue`是一个有边界的阻塞队列，它的内部实现是一个数组。有边界的意思是它的容量是有限的，我们必须在其初始化的时候指定它的容量大小，容量大小一旦指定就不可改变。它先进先出的方式存储数据，最新插入的对象是尾部，最新移出的对象是头部

![ArrayBlockingQueue](http://ovn0i3kdg.bkt.clouddn.com/ArrayBlockingQueue.png?imageView/2/w/500)

## 构造函数
```java

/** Condition for waiting puts */
private final Condition notFull;
/** Condition for waiting takes */
private final Condition notEmpty;
/** Main lock guarding all access */
final ReentrantLock lock;

/** The queued items */
final Object[] items;

public ArrayBlockingQueue(int capacity) {
    this(capacity, false);
}
public ArrayBlockingQueue(int capacity, boolean fair) {
    if (capacity <= 0)
        throw new IllegalArgumentException();
    this.items = new Object[capacity];
    lock = new ReentrantLock(fair);
    notEmpty = lock.newCondition();
    notFull =  lock.newCondition();
}

public ArrayBlockingQueue(int capacity, boolean fair,
                              Collection<? extends E> c) {
    this(capacity, fair);

    final ReentrantLock lock = this.lock;
    lock.lock(); // Lock only for visibility, not mutual exclusion
    try {
        int i = 0;
        try {
            for (E e : c) {
                checkNotNull(e);
                items[i++] = e;
            }
        } catch (ArrayIndexOutOfBoundsException ex) {
            throw new IllegalArgumentException();
        }
        count = i;
        putIndex = (i == capacity) ? 0 : i;
    } finally {
        lock.unlock();
    }
}
```

## offer方法
```java
/** items index for next put, offer, or add */
int putIndex; //指向下一次入队元素的位置
/** Number of elements in the queue */
int count;
public boolean offer(E e) {
    checkNotNull(e);
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        if (count == items.length)
            return false;
        else {
            enqueue(e);
            return true;
        }
    } finally {
        lock.unlock();
    }
}

public boolean offer(E e, long timeout, TimeUnit unit)
    throws InterruptedException {

    checkNotNull(e);
    long nanos = unit.toNanos(timeout);
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        while (count == items.length) {
            if (nanos <= 0)
                return false;
            nanos = notFull.awaitNanos(nanos);
        }
        enqueue(e);
        return true;
    } finally {
        lock.unlock();
    }
}
private void enqueue(E x) {
    // assert lock.getHoldCount() == 1;
    // assert items[putIndex] == null;
    final Object[] items = this.items;
    items[putIndex] = x;
    if (++putIndex == items.length)
        putIndex = 0; // 从头开始
    count++;
    notEmpty.signal();
}
```

## put方法
```java
public void put(E e) throws InterruptedException {
   checkNotNull(e);
   final ReentrantLock lock = this.lock;
   lock.lockInterruptibly();
   try {
       while (count == items.length)
           notFull.await();
       enqueue(e);
   } finally {
       lock.unlock();
   }
}
```

## poll方法
```java
/**
* Shared state for currently active iterators, or null if there
* are known not to be any.  Allows queue operations to update
* iterator state.
*/
transient Itrs itrs = null;

/** items index for next take, poll, peek or remove */
int takeIndex;

public E poll() {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        return (count == 0) ? null : dequeue();
    } finally {
        lock.unlock();
    }
}
private E dequeue() {
    // assert lock.getHoldCount() == 1;
    // assert items[takeIndex] != null;
    final Object[] items = this.items;
    @SuppressWarnings("unchecked")
    E x = (E) items[takeIndex];
    items[takeIndex] = null;
    if (++takeIndex == items.length)
        takeIndex = 0;
    count--;
    if (itrs != null)
        itrs.elementDequeued();
    notFull.signal();
    return x;
}

public E poll(long timeout, TimeUnit unit) throws InterruptedException {
    long nanos = unit.toNanos(timeout);
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        while (count == 0) {
            if (nanos <= 0)
                return null;
            nanos = notEmpty.awaitNanos(nanos);
        }
        return dequeue();
    } finally {
        lock.unlock();
    }
}
```
## take方法
```java
public E take() throws InterruptedException {
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        while (count == 0)
            notEmpty.await();
        return dequeue();
    } finally {
        lock.unlock();
    }
}
```

## peek
```java
public E peek() {
  final ReentrantLock lock = this.lock;
  lock.lock();
  try {
      return itemAt(takeIndex); // null when queue is empty
  } finally {
      lock.unlock();
  }
}
final E itemAt(int i) {
    return (E) items[i];
}
```

## remove
```java
public boolean remove(Object o) {
    if (o == null) return false;
    final Object[] items = this.items;
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        if (count > 0) {
            final int putIndex = this.putIndex;
            int i = takeIndex;
            do {
                if (o.equals(items[i])) {
                    removeAt(i);
                    return true;
                }
                if (++i == items.length)
                    i = 0;
            } while (i != putIndex);
        }
        return false;
    } finally {
        lock.unlock();
    }
}
```
## drainTo
```java
public int drainTo(Collection<? super E> c) {
    return drainTo(c, Integer.MAX_VALUE);
}

public int drainTo(Collection<? super E> c, int maxElements) {
    checkNotNull(c);
    if (c == this)
        throw new IllegalArgumentException();
    if (maxElements <= 0)
        return 0;
    final Object[] items = this.items;
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        int n = Math.min(maxElements, count);
        int take = takeIndex;
        int i = 0;
        try {
            while (i < n) {
                @SuppressWarnings("unchecked")
                E x = (E) items[take];
                c.add(x);
                items[take] = null;
                if (++take == items.length)
                    take = 0;
                i++;
            }
            return n;
        } finally {
            // Restore invariants even if c.add() threw
            if (i > 0) {
                count -= i;
                takeIndex = take;
                if (itrs != null) {
                    if (count == 0)
                        itrs.queueIsEmpty();
                    else if (i > take)
                        itrs.takeIndexWrapped();
                }
                for (; i > 0 && lock.hasWaiters(notFull); i--)
                    notFull.signal();
            }
        }
    } finally {
        lock.unlock();
    }
}
```
