# LinkedBlockingQueue

基本上看到属性就能明白这是怎么回事了：
```Java
private final int capacity;
private final AtomicInteger count;
transient LinkedBlockingQueue.Node<E> head;
private transient LinkedBlockingQueue.Node<E> last;
private final ReentrantLock takeLock;
private final Condition notEmpty;
private final ReentrantLock putLock;
private final Condition notFull;
```
## 构造方法
```Java
public LinkedBlockingQueue() {
    this(2147483647);
}

public LinkedBlockingQueue(int var1) {
    this.count = new AtomicInteger();
    this.takeLock = new ReentrantLock();
    this.notEmpty = this.takeLock.newCondition();
    this.putLock = new ReentrantLock();
    this.notFull = this.putLock.newCondition();
    if (var1 <= 0) {
        throw new IllegalArgumentException();
    } else {
        this.capacity = var1;
        this.last = this.head = new LinkedBlockingQueue.Node((Object)null);
    }
}
```
