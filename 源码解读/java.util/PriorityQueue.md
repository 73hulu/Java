# PriorityQueue

<!-- toc -->
<!-- tocstop -->

`PriorityQueue`是优先队列，实际是就是堆。堆是一种特殊的树，具备根节点大于左右左子树上的节点或根节点小于左右子树上的节点，前者称为大根堆，后者称为小跟堆。

`PriorityQueue`使用数组实现堆，因为堆是完全二叉树，所以使用数组可以随机访问节点的父节点和子节点。注意，`PriorityQueue`**不允许存在null元素**。

![PriorityQueue](http://ovn0i3kdg.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-02-14%20%E4%B8%8B%E5%8D%888.48.52.png?imageView/2/w/400)
![PriorityQueue](http://ovn0i3kdg.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-02-14%20%E4%B8%8B%E5%8D%888.54.37.png?imageView/2/w/400)


## public class PriorityQueue<E> extends AbstractQueue<E> implements java.io.Serializable
类声明，`PriorityQueue`继承自`AbstractQueue`，而这个抽象类实现了接口`Quequ`。

## 构造方法
`PriorityQueue`提供了7种重载的构造方法。下面四种构造方法用来指定队列的容量和比较器：
```java
public PriorityQueue() {
    this(DEFAULT_INITIAL_CAPACITY, null);
}

public PriorityQueue(int initialCapacity) {
    this(initialCapacity, null);
}

public PriorityQueue(Comparator<? super E> comparator) {
    this(DEFAULT_INITIAL_CAPACITY, comparator);
}


public PriorityQueue(int initialCapacity,
                     Comparator<? super E> comparator) {
    // Note: This restriction of at least one is not actually needed,
    // but continues for 1.5 compatibility
    if (initialCapacity < 1)
        throw new IllegalArgumentException();
    this.queue = new Object[initialCapacity];
    this.comparator = comparator;
}
private static final int DEFAULT_INITIAL_CAPACITY = 11;
/**
 * Priority queue represented as a balanced binary heap: the two
 * children of queue[n] are queue[2*n+1] and queue[2*(n+1)].  The
 * priority queue is ordered by comparator, or by the elements'
 * natural ordering, if comparator is null: For each node n in the
 * heap and each descendant d of n, n <= d.  The element with the
 * lowest value is in queue[0], assuming the queue is nonempty.
 */
transient Object[] queue;
```
可以看到，默认的队列容量为11，默认的比较器为null，此时队列将按照自然顺序进行元素的比较。另外，优先队列的存储结构实际上是一个`Object`类型的数组，从0单元开始存储。对于完全二叉树来来说，queue[i]的左右孩子节点分别是queue[2i+1]和queue[2(i+1)]。

另外三种构造方法接收一个集合或者优先队列作为参数，定义如下：
```java
public PriorityQueue(Collection<? extends E> c) {
    if (c instanceof SortedSet<?>) {
        SortedSet<? extends E> ss = (SortedSet<? extends E>) c;
        this.comparator = (Comparator<? super E>) ss.comparator();
        initElementsFromCollection(ss);
    }
    else if (c instanceof PriorityQueue<?>) {
        PriorityQueue<? extends E> pq = (PriorityQueue<? extends E>) c;
        this.comparator = (Comparator<? super E>) pq.comparator();
        initFromPriorityQueue(pq);
    }
    else {
        this.comparator = null;
        initFromCollection(c);
    }
}

private void initFromCollection(Collection<? extends E> c) {
    initElementsFromCollection(c);
    heapify();
}

public PriorityQueue(PriorityQueue<? extends E> c) {
    this.comparator = (Comparator<? super E>) c.comparator();
    initFromPriorityQueue(c);
}

private void initFromPriorityQueue(PriorityQueue<? extends E> c) {
    if (c.getClass() == PriorityQueue.class) {
        this.queue = c.toArray();
        this.size = c.size();
    } else {
        initFromCollection(c);
    }
}

public PriorityQueue(SortedSet<? extends E> c) {
    this.comparator = (Comparator<? super E>) c.comparator();
    initElementsFromCollection(c);
}

private void initElementsFromCollection(Collection<? extends E> c) {
    Object[] a = c.toArray();
    // If c.toArray incorrectly doesn't return Object[], copy it.
    if (a.getClass() != Object[].class)
        a = Arrays.copyOf(a, a.length, Object[].class);
    int len = a.length;
    if (len == 1 || this.comparator != null)
        for (int i = 0; i < len; i++)
            if (a[i] == null)
                throw new NullPointerException();
    this.queue = a;
    this.size = a.length;
}

```
可以看到，当参数为`SortedSet`和`PriorityQueue`的实例的时候，由于元素已经是有序的，只需要将其中的存储结构变成数组即可，不需要调整，比如对于`PriorityQueue`的实例，直接取得该实例的存储结构。对于`SortedSet`实例，复制了一份该实例中的数组，但是注意，这个过程中需要检查元素是否存在null值，`PriorityQueue`是不允许null值出现的，将抛出空指针异常。

而对于其他集合，由于无序性，所以需要堆的构造，这里重要的操作在`heapify`方法中定义：
```java
private void heapify() {
    for (int i = (size >>> 1) - 1; i >= 0; i--)
        siftDown(i, (E) queue[i]);
}

private void siftDown(int k, E x) {
    if (comparator != null)
        siftDownUsingComparator(k, x);
    else
        siftDownComparable(k, x);
}

@SuppressWarnings("unchecked")
private void siftDownUsingComparator(int k, E x) {
   int half = size >>> 1;
   while (k < half) {
       int child = (k << 1) + 1;
       Object c = queue[child];
       int right = child + 1;
       if (right < size &&
           comparator.compare((E) c, (E) queue[right]) > 0)
           c = queue[child = right];
       if (comparator.compare(x, (E) c) <= 0)
           break;
       queue[k] = c;
       k = child;
   }
   queue[k] = x;
}

@SuppressWarnings("unchecked")
private void siftDownComparable(int k, E x) {
    Comparable<? super E> key = (Comparable<? super E>)x;
    int half = size >>> 1;        // loop while a non-leaf
    while (k < half) {
        //这里假设左孩子存在，左孩子存在但是右孩子不一定存在
        int child = (k << 1) + 1; // assume left child is least
        Object c = queue[child];
        int right = child + 1;
        if (right < size &&
            ((Comparable<? super E>) c).compareTo((E) queue[right]) > 0)
            c = queue[child = right]; //这里比较左右孩子的大小，最终c是左右孩子中比较小的那一个节点
        if (key.compareTo((E) c) <= 0) //key的值比当前这个节点的值小，那么这个位置就是目标值的最终位置，此时退出循环，将目标值赋值给该位置即可
            break;
        queue[k] = c; //比目标值小的元素上移
        k = child;
    }
    queue[k] = key;
}
```
这里可以看到在得到一个无须的数组之后，需要从最后一个节点的父节点开始进行调整，这种调整是将以此节点为根节点的树调整为堆。`siftDownUsingComparator`和`siftDownComparable`主要是考虑到是否定义了比较器，如果定义了比较器，则根据比较器排序，如果没有定义比较器，则按照自然顺序进行排序。这个方法也是`poll`即出队方法的核心，我们下面再讨论。

`PriorityQueue`的主要操作是新建、插入和删除，构造函数承担了新建的任务，下面我们就只用讨论插入和删除操作。

## 插入
插入操作是在一个堆中插入一个新的节点，经过调整之后仍然是一个堆。`PriorityQueue`中提供了`add`和`offer`方法，前者直接调用后者，后者定义如下：
```java
public boolean offer(E e) {
    if (e == null)
        throw new NullPointerException();
    modCount++;
    int i = size;
    if (i >= queue.length)
        grow(i + 1);
    size = i + 1;
    if (i == 0)
        queue[0] = e;
    else
        siftUp(i, e);
    return true;
}

private void grow(int minCapacity) {
    int oldCapacity = queue.length;
    // Double size if small; else grow by 50%
    int newCapacity = oldCapacity + ((oldCapacity < 64) ?
                                     (oldCapacity + 2) :
                                     (oldCapacity >> 1));
    // overflow-conscious code
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    queue = Arrays.copyOf(queue, newCapacity);
}

private void siftUp(int k, E x) {
    if (comparator != null)
        siftUpUsingComparator(k, x);
    else
        siftUpComparable(k, x);
}
```
这里我们看到，往其中插入元素的时候首先进行了扩容，扩容的政策是如果当前数组比较小（小于64），则增加2倍，如果比较大（大于64），则增加1倍+2。然后判断队列是否为空，为空则直接插入作为根节点，不为空则使用插入算法，而插入算法`siftUp`仍旧对是否指定比较器做了区分，我们着重看一下指定了比较器的插入算法：
```java
@SuppressWarnings("unchecked")
private void siftUpUsingComparator(int k, E x) {
    while (k > 0) {
        int parent = (k - 1) >>> 1;
        Object e = queue[parent];
        if (comparator.compare(x, (E) e) >= 0)
            break;
        queue[k] = e;
        k = parent;
    }
    queue[k] = x;
}
```
由以上算法可以看出，新插入的节点一开始是被放到了数组的最后一个位置，然后比较其祖先节点序列，将大于它的往下移动，慢慢找到自己的位置，最后赋值到该位置，完成了插入操作。下图是该过程的示例：

![offer操作过程示意](https://ws4.sinaimg.cn/large/0069RVTdly1fuwdpm5bu4j30kf09bjrm.jpg)

## 删除
堆的删除的位置是定的，一定发生在根节点。`PriorityQueue`提供`poll`和`remove`方法，这两个方法是不一样的，首先来看`poll`方法：
```java
@SuppressWarnings("unchecked")
public E poll() {
    if (size == 0)
        return null;
    int s = --size;
    modCount++;
    E result = (E) queue[0];
    E x = (E) queue[s];
    queue[s] = null;
    if (s != 0)
        siftDown(0, x);
    return result;
}
```
`poll`删除的是根节点，它将根节点和最后一个节点交换，其根本还是从根节点开始进行"下沉"的操作，调用了`siftDown`方法，这个方法在前面已经讲过。

![poll操作1](https://ws1.sinaimg.cn/large/0069RVTdly1fuwdq6akehj30br08l0sw.jpg)

![poll操作2](https://ws3.sinaimg.cn/large/0069RVTdly1fuwdql0wksj30es08ht8v.jpg)

![poll操作3](https://ws2.sinaimg.cn/large/0069RVTdly1fuwdqvmxx5j30kb07zjrr.jpg)



`remove`和`poll`不一样的是，它接受一个`Object`类型的参数，不仅可以删除头节点而且还可以用 remove(Object o) 来删除堆中的与给定对象相同的最先出现的对象。其定义如下：
```java
public boolean remove(Object o) {
    int i = indexOf(o);
    if (i == -1)
        return false;
    else {
        removeAt(i);
        return true;
    }
}

private int indexOf(Object o) {
    if (o != null) {
        for (int i = 0; i < size; i++)
            if (o.equals(queue[i]))
                return i;
    }
    return -1;
}

@SuppressWarnings("unchecked")
private E removeAt(int i) {
   // assert i >= 0 && i < size;
   modCount++;
   int s = --size;
   if (s == i) // removed last element
       queue[i] = null;
   else {
       E moved = (E) queue[s];
       queue[s] = null;
       siftDown(i, moved);
       if (queue[i] == moved) {
           siftUp(i, moved);
           if (queue[i] != moved)
               return moved;
       }
   }
   return null;
}
```
而这个过程就比较麻烦，因为删除了这个节点后，不仅需要"上浮"还需要“下沉”操作。下面是一个remove(4)的过程：

![remove(4)](https://ws1.sinaimg.cn/large/0069RVTdly1fuwdrd12axj30ls0gmdgm.jpg)


`PriorityQueue`默认的是小根堆，如果实现大根堆呢？可以传入自定义的比较器，例如：
```java

private static final int DEFAULT_INITIAL_CAPACITY = 11;

PriorityQueue<Integer> maxHeap=new PriorityQueue<Integer>(DEFAULT_INITIAL_CAPACITY, new Comparator<Integer>() {
    @Override
    public int compare(Integer o1, Integer o2) {                
        return o2-o1;
    }
});
```
这样得到的`maxHeap`就是一个大根堆。


参考
* [Java堆结构PriorityQueue完全解析](http://blog.csdn.net/u013309870/article/details/71189189)
