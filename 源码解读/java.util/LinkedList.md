# LinkedList
`LinkedList`是Java中线性表的链表实现。类结构如下：

![LinkedList](http://ovn0i3kdg.bkt.clouddn.com/LinkedList,png)
![LinkedList](http://ovn0i3kdg.bkt.clouddn.com/LinkedList_1.png)


### public class LinkedList<E> extends AbstractSequentialList<E> implements List<E>, Deque<E>, Cloneable, java.io.Serializable
类声明，`LinkedList`继承自抽象父类`AbstractSequentialList`，实现`List`、`Deque`、`Cloneable`和`Serializable`接口。其中`Deque`是双向队列，所以可以在这个类中看到该类对双向队列的实现。

另外一点需要说明的是，`LinkedList`没有继承`RandomAccess`，虽然`LinkedList`也通过内置迭代器类实现了迭代器，但是最好不要用迭代器遍历。

### 构造函数
重载了两个构造函数。
#### public LinkedList(){...}
默认的公共构造函数，方法体为空。

### public LinkedList(Collection<? extends E> c) {...}
以集合c为参数，构造一个包含集合c内元素的链表。
```Java
public LinkedList(Collection<? extends E> c) {
    this();
    addAll(c);
}
```
其中调用的addAll方法定义如下:
```Java
public boolean addAll(Collection<? extends E> c) {
    return addAll(size, c);
}
```
`addAll(size, c)`方法将会把集合c中的元素追加到链表中，其中变量`size`表示的是当前链表中的元素个数，定义为`transient int size = 0;`，该方法的定义为：
```Java
public boolean addAll(int index, Collection<? extends E> c) {
    checkPositionIndex(index);

    Object[] a = c.toArray();
    int numNew = a.length;
    if (numNew == 0)
        return false;

    Node<E> pred, succ;
    if (index == size) {
        succ = null;
        pred = last;
    } else {
        succ = node(index);
        pred = succ.prev;
    }

    for (Object o : a) {
        @SuppressWarnings("unchecked") E e = (E) o;
        Node<E> newNode = new Node<>(pred, e, null);
        if (pred == null)
            first = newNode;
        else
            pred.next = newNode;
        pred = newNode;
    }

    if (succ == null) {
        last = pred;
    } else {
        pred.next = succ;
        succ.prev = pred;
    }

    size += numNew;
    modCount++;
    return true;
}

private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}

Node<E> node(int index) {
    // assert isElementIndex(index);

    if (index < (size >> 1)) {
        Node<E> x = first;
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else {
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
}

transient Node<E> first;
transient Node<E> last;
```
注意到链表节点`Node`的数据结构中有前驱节点指针和后继节点指针，所以`LinkedList`是实际上是一个双向链表。在`addAll`方法中，首先将集合c转化为数组，之后的插入过程就是遍历数组，一个个创建节点建立链接的过程。其中定义的`pred`和`succ`节点指示当前应该插入的节点的前驱节点和后继节点。node是查找第index位置上的元素的方法，在该方法中，根据index和二分之一size的大小关系，决定搜索策略，如果要查找的节点在链表的前半部分，那就从头开始用next指针域遍历查找，否则从last开始用prev指针域遍历查找。其中`first`是链表的第一个节点，`last`是链表的最后一个节点。

在插入节点的时候对空链表（`prev`是否为null）做单独处理。

addAll方法调用成功后会返回布尔量表示操作是否成功。

### public E getFirst(){...} 和 public E getLast() {...}
获取链表的首个节点的数据和取得链表的最后一个数据节点。定义如下：
```Java
public E getFirst() {
    final Node<E> f = first;
    if (f == null)
        throw new NoSuchElementException();
    return f.item;
}

public E getLast() {
    final Node<E> l = last;
    if (l == null)
        throw new NoSuchElementException();
    return l.item;
}
```
很好理解，不用多解释。

### public E removeFirst() {...}和public E removeLast(){...}
移除首个节点和移除最后一个节点。定义如下：
```java
public E removeFirst() {
    final Node<E> f = first;
    if (f == null)
        throw new NoSuchElementException();
    return unlinkFirst(f);
}

private E unlinkFirst(Node<E> f) {
    // assert f == first && f != null;
    final E element = f.item;
    final Node<E> next = f.next;
    f.item = null;
    f.next = null; // help GC
    first = next;
    if (next == null)
        last = null;
    else
        next.prev = null;
    size--;
    modCount++;
    return element;
}

public E removeLast() {
   final Node<E> l = last;
   if (l == null)
       throw new NoSuchElementException();
   return unlinkLast(l);
}

private E unlinkLast(Node<E> l) {
   // assert l == last && l != null;
   final E element = l.item;
   final Node<E> prev = l.prev;
   l.item = null;
   l.prev = null; // help GC
   last = prev;
   if (prev == null)
       first = null;
   else
       prev.next = null;
   size--;
   modCount++;
   return element;
}
```
注意到删除元素的时候，不仅要将被删除的元素置为null，还要更新后继/前驱节点的相应指针，注意对首尾节点的处理。方法最后返回的是被删除的节点的值。

### public void addFirst(E e){...}和public void addLast(E e) {...}
向链表头添加元素和向链表为添加元素。
```Java
public void addFirst(E e) {
    linkFirst(e);
}
private void linkFirst(E e) {
    final Node<E> f = first;
    final Node<E> newNode = new Node<>(null, e, f);
    first = newNode;
    if (f == null)
        last = newNode;
    else
        f.prev = newNode;
    size++;
    modCount++;
}

public void addLast(E e) {
    linkLast(e);
}

void linkLast(E e) {
    final Node<E> l = last;
    final Node<E> newNode = new Node<>(l, e, null);
    last = newNode;
    if (l == null)
        first = newNode;
    else
        l.next = newNode;
    size++;
    modCount++;
}
```
### public int size(){...}
取得链表的长度，直接返回成员变量size即可。

### public boolean contains(Object o){...}
查看链表是否包含某一个指定的元素，借助的是`indexOf`方法，该方法如果未查找到元素，则返回-1.否则返回该元素所在的位置索引。
```Java
public boolean contains(Object o) {
    return indexOf(o) != -1;
}
```

### public int indexOf(Object o){...}
从前往后，查看指定元素在链表中的位置，如果链表中无该元素，则返回-1，注意到参数类型是Object，而不是泛型。定义如下：
```Java
public int indexOf(Object o) {
    int index = 0;
    if (o == null) {
        for (Node<E> x = first; x != null; x = x.next) {
            if (x.item == null)
                return index;
            index++;
        }
    } else {
        for (Node<E> x = first; x != null; x = x.next) {
            if (o.equals(x.item))
                return index;
            index++;
        }
    }
    return -1;
}
```
注意到对指定的元素为null时的特殊处理。这点在`ArrayList`的学习中出现过，这样处理的目的是为了防止出现空指针异常。

### public int lastIndexOf(Object o){...}
从后往前，查看指定元素在链表中的位置，如果无该元素，则返回-1，同样元素类型是Object而不是泛型。定义如下：
```Java
public int lastIndexOf(Object o) {
    int index = size;
    if (o == null) {
        for (Node<E> x = last; x != null; x = x.prev) {
            index--;
            if (x.item == null)
                return index;
        }
    } else {
        for (Node<E> x = last; x != null; x = x.prev) {
            index--;
            if (o.equals(x.item))
                return index;
        }
    }
    return -1;
}
```
由于`LinkedList`是双向链表的实现，所以从后往前查找十分方便，使用`prev`指针遍历就行。
### public boolean add(E e){...}
向链表中追加一个元素，相当于`addLast(e)`，永远返回true。定义如下：
```Java
public boolean add(E e) {
    linkLast(e);
    return true;
}
```
### public boolean remove(Object o){...}
移除第一个指定的元素o，注意参数类型是Object而不是泛型。定义如下：
```Java
public boolean remove(Object o) {
    if (o == null) {
        for (Node<E> x = first; x != null; x = x.next) {
            if (x.item == null) {
                unlink(x);
                return true;
            }
        }
    } else {
        for (Node<E> x = first; x != null; x = x.next) {
            if (o.equals(x.item)) {
                unlink(x);
                return true;
            }
        }
    }
    return false;
}
```

### public void clear(){...}
将链表中的所有元素移除，注意不单单是让fist = last = null，这么简单，需要释放每一个节点中的数据域和指针域。最后长度置为0.定义如下：
```Java
public void clear() {
    // Clearing all of the links between nodes is "unnecessary", but:
    // - helps a generational GC if the discarded nodes inhabit
    //   more than one generation
    // - is sure to free memory even if there is a reachable Iterator
    for (Node<E> x = first; x != null; ) {
        Node<E> next = x.next;
        x.item = null;
        x.next = null;
        x.prev = null;
        x = next;
    }
    first = last = null;
    size = 0;
    modCount++;
}
```

### get和set方法
get方法用来取得第index索引位置的节点数据，set方法用来设置第index索引位置的节点数据。
```Java
public E get(int index) {
    checkElementIndex(index);
    return node(index).item;
}
public E set(int index, E element) {
    checkElementIndex(index);
    Node<E> x = node(index);
    E oldVal = x.item;
    x.item = element;
    return oldVal;
}
```
需要注意的是`set`方法有返回值，返回的是替换之前的元素的数据值。

### public void add(int index, E element){...}
在index位置上添加一个元素节点。
```Java
public void add(int index, E element) {
    checkPositionIndex(index);

    if (index == size)
        linkLast(element);
    else
        linkBefore(element, node(index));
}
```
可以看到这里为了效率对index做了判断，如果要插入到最后一个节点之后，直接调用的是`linkLast`方法来追加节点。否则是将元素节点添加到原来index节点之前，调用的`linkBefore`方法定义如下：
```Java
void linkBefore(E e, Node<E> succ) {
    // assert succ != null;
    final Node<E> pred = succ.prev;
    final Node<E> newNode = new Node<>(pred, e, succ);
    succ.prev = newNode;
    if (pred == null)
        first = newNode;
    else
        pred.next = newNode;
    size++;
    modCount++;
}
```

### public E remove(int index){...}
移除指定位置上的元素。
```Java
public E remove(int index) {
    checkElementIndex(index);
    return unlink(node(index));
}
```

### 单向队列（Queue）
Java中`LinkedList`实现了单向队列的功能，单向队列规定队首只能出队，队尾只能入队。下面这些方法就是针对队列设计的功能。
#### public E peek(){...} 和 public E element(){...}
两个方法都是获取到队首的元素，但是不出队。两者有细微的差别：`peek`方法在遇到空链表的时候会返回null，而`element`方法在遇到空链表的时候会抛出异常。
```Java
public E peek() {
    final Node<E> f = first;
    return (f == null) ? null : f.item;
}

public E element() {
    return getFirst();
}

public E getFirst() {
    final Node<E> f = first;
    if (f == null)
        throw new NoSuchElementException();
    return f.item;
}
```


#### public E poll(){...} 和 public E remove(){...}
真正的出队操作。两者的差别在于前者在遇到空链表的时候返回null，后者抛出异常。
```Java
public E poll() {
    final Node<E> f = first;
    return (f == null) ? null : unlinkFirst(f);
}

public E remove() {
    return removeFirst();
}

public E removeFirst() {
    final Node<E> f = first;
    if (f == null)
        throw new NoSuchElementException();
    return unlinkFirst(f);
}
```

#### public boolean offer(E e){...}
入队操作。即追加元素到链表末尾。定义如下：
```Java
public boolean offer(E e) {
   return add(e);
}
```

### 双向队列（Deque）
`LinkedList`同样实现了双向队列。即队首和队尾都可以实现出队和入队。

#### public boolean offerFirst(E e)
队首入队。
```java
public boolean offerFirst(E e) {
    addFirst(e);
    return true;
}
```
#### public boolean offerLast(E e){...}
队尾入队。
```java
public boolean offerLast(E e) {
    addLast(e);
    return true;
}
```
#### public E peekFirst() {...}
获取队首元素，但是不出队。
```java
public E peekFirst() {
  final Node<E> f = first;
  return (f == null) ? null : f.item;
}
```
#### public E peekLast() {...}
获取队尾元素，但是不出队。
```java
public E peekLast() {
    final Node<E> l = last;
    return (l == null) ? null : l.item;
}
```

#### public E pollFirst() {...}
队首元素出队。
```java
public E pollFirst() {
   final Node<E> f = first;
   return (f == null) ? null : unlinkFirst(f);
}
```
#### public E pollLast() {...}
队尾元素出队。
```java
public E pollLast() {
   final Node<E> l = last;
   return (l == null) ? null : unlinkLast(l);
}
```


### 栈
`LinkedList`同样实现了栈的操作，主要是出栈`pop`操作和入栈`push`操作。

#### public void push(E e) {...}
入栈操作。
```java
public void push(E e) {
    addFirst(e);
}
```
####   public E pop(){...}
出栈操作。
```java
public E pop() {
    return removeFirst();
}
```

### public Object clone(){...}
浅复制方法。
```java
public Object clone() {
    LinkedList<E> clone = superClone();

    // Put clone into "virgin" state
    clone.first = clone.last = null;
    clone.size = 0;
    clone.modCount = 0;

    // Initialize clone with our elements
    for (Node<E> x = first; x != null; x = x.next)
        clone.add(x.item);

    return clone;
}

@SuppressWarnings("unchecked")
private LinkedList<E> superClone() {
   try {
       return (LinkedList<E>) super.clone();
   } catch (CloneNotSupportedException e) {
       throw new InternalError(e);
   }
}
```

### public Object[] toArray(){...}
将链表转化称为数组。很简单，就是构造一个size长度的对象数组，遍历赋值即可。
```java
public Object[] toArray() {
    Object[] result = new Object[size];
    int i = 0;
    for (Node<E> x = first; x != null; x = x.next)
        result[i++] = x.item;
    return result;
}
```

## 总结
`LinkedList`是Java中线性表的链表存储方式的实现。同时该类实现了单项队列、双向队列和栈的功能。同时该类是非线程安全的。如果程序对线程安全性有要求。可以使用`Collections.synchronizedList()`来创建线程安全的线性表。

另外对于`LinkedList`的遍历，有以下几种方法：
1. 通过迭代器遍历。即通过Iterator去遍历
```java
for (Iterator itr = list.iterator(); itr.hasNext();){
            System.out.println(itr.next());
        }
```
2. 通过随机访问遍历(不可取，太慢了)
```java
int size = list.size();
for (int i=0; i<size; i++) {
    list.get(i);        
}
```
3. 通过for循环
```java
for (Integer integ:list)
    ;
```
4. 通过pollFirst()来遍历LinkedList
```java
while(list.pollFirst() != null)
    ;
```
5. 通过pollLast()来遍历LinkedList
```java
while(list.pollLast() != null)
    ;
```
6. 通过removeFirst()来遍历LinkedList
```java
try {
    while(list.removeFirst() != null)
        ;
} catch (NoSuchElementException e) {
}
```
7. 通过removeLast()来遍历LinkedList
```java
try {
    while(list.removeLast() != null)
        ;
} catch (NoSuchElementException e) {
}
```

以上这些方法的测试时间是：
```
iteratorLinkedListThruIterator：8 ms
iteratorLinkedListThruForeach：3724 ms
iteratorThroughFor2：5 ms
iteratorThroughPollFirst：8 ms
iteratorThroughPollLast：6 ms
iteratorThroughRemoveFirst：2 ms
iteratorThroughRemoveLast：2 ms
```
由此可见，遍历`LinkedList`时，使用`removeFist()`或`removeLast()`效率最高。但用它们遍历时，会删除原始数据；若单纯只读取，而不删除，应该使用第3种遍历方式。**无论如何，千万不要通过随机访问去遍历LinkedList！**


参考
* [Java 集合系列05之 LinkedList详细介绍(源码解析)和使用示例](https://www.cnblogs.com/skywang12345/p/3308807.html#a1)
