# HashMap
`HashMap`是基于哈希表（Hash Table）的`Map`接口的实现，它和`HashTable`的区别在于两点：
1. `HashMap`是非线程安全的，而`HashTable`线程安全的。
2. `HashMap`允许以null值作为key和value值，而`HashTable`是不允许的。
除此之外，`HashMap`近似等于`HashTable`。

另外，`HashMap`不保证顺序并且不保证在使用的过程中顺序恒定。

`HashMap`能够提供线性时间的存取操作！！！当前前提是散列函数在桶之间正确分散元素。对集合视图的迭代需要的时间与`HashMap`实例的"容量"（桶的数量）加上其大小（键值映射的数量）成正比，因此，如果迭代性能很重要，就不要将初始容量设置地太高（或者负载因子太小）。

`HashMap`的性能受到两个参数的影响：初始化容量和负载因子。容量就是哈希表的桶的数量，初始容量就是哈希表创建的时的容量。负载因子是散列表在其容量增加之前允许得到度量。当哈希表中条目数量超过了负载因子和当前容量的乘积时，散列表将被重新映射（也就是说内部的数据结构将被重建），重建之后散列表的数量大约是存储桶数量的2倍。

一般来说，这个负载因子取值为0.75，这个数提供了时间和空间成本之间的良好折衷。如果过大，可以减少空间开销但是会增加查找成本，反应在大部分的HashMap类的操作中，包括get和put操作。在设置初始容量的时候，应该考虑映射中条目数量及其负载燕子，以尽量减少重新操作的次数。如果初始容量大于最大入口数量除以负载因子，则不会发生重新刷新的操作。

**HashMap是非线程安全的。**如果多个线程同时访问哈希映射，并且至少一个线程在结构上做了修改，则必须进行外部同步。这种同步通常是通过封装映射的某个对象完成的，如果不存在这样的对象，就应该使用`Collections.synchronizedMap`方法“映射”该映射，这最好在创建的时候完成，以防止意外的不同步访问地图，写法如下：
```java
Map map = Collections.synchronizedMap(new HashMap(...))
```

`HashMap`类的结构如下：

![HashMap](http://ovn0i3kdg.bkt.clouddn.com/HashMap.png)


### public class HashMap<K,V> extends AbstractMap<K,V> implements Map<K,V>, Cloneable, Serializable
类声明，`HashMap`继承`AbstractMap`，实现`Map`接口、`Cloneable`接口和`Serializable`接口。
> `AbstractMap`抽象类本身就实现了`Map`接口，为什么`HashMap`还要再继承一遍？

源码对该类的实现有这样的说明：
通常情况下，`HashMap`作为哈希表的容器，但是当容器变的很大的时候，它将被转化为`TreeNodes`的容器，类似于`java.util.TreeMap`中。大多数的方法都使用正常的容器，但是会适时转发给`TreeNodes`方法（只需要通过检查一个节点的实例）


### 构造方法
重载了四种构造方法。
#### public HashMap(int initialCapacity, float loadFactor) {...}
指定哈希表初始容量和装载因子。

```java
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);
    this.loadFactor = loadFactor;
    this.threshold = tableSizeFor(initialCapacity);
}

// 最大容量（必须是2的幂且小于2的30次方，传入容量过大将被这个值替换）
static final int MAXIMUM_CAPACITY = 1 << 30;

final float loadFactor;

int threshold;

//当在实例化HashMap实例时，如果给定了initialCapacity，由于HashMap的capacity都是2的幂，因此这个方法用于找到大于等于initialCapacity的最小的2的幂（initialCapacity如果就是2的幂，则返回的还是这个数）。
static final int tableSizeFor(int cap) {
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}

```
首先对传入的参数范围作了检查，`loadFactor`是常量，一旦赋值就不能改变。而变量`threshold`是下一次扩容的临界值，用于判断是否需要调整`HashMap`的容量（threshold = 容量*加载因子） ，调用的`tableSizeFor`方法写的很奇怪，找到下一个最小的比参数大的2的高次幂。具体的可以参考https://zhidao.baidu.com/question/291266003.html。 这里保证了该值是一个2的倍数，比如你设置的是5，而实际上空间大小为8。

#### public HashMap(int initialCapacity) {...}
指定初始化容量。
```java
public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}
static final float DEFAULT_LOAD_FACTOR = 0.75f;
```
默认的负载因子为0.75。

### public HashMap() {...}
无参构造方法，默认初始化大小为16，初始化负载因子为0.75。
```java
public HashMap() {
    this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
}
```
从方法本身看不出来初始化容量是16，看到下面的`putVal`就可以看到了。

### public HashMap(Map<? extends K, ? extends V> m){...}
用指定的映射来构造新的映射。默认负载大小为0.75。
```java
public HashMap(Map<? extends K, ? extends V> m) {
   this.loadFactor = DEFAULT_LOAD_FACTOR;
   putMapEntries(m, false);
}

final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
    int s = m.size();
    if (s > 0) {
        if (table == null) { // pre-size
            float ft = ((float)s / loadFactor) + 1.0F;
            int t = ((ft < (float)MAXIMUM_CAPACITY) ?
                     (int)ft : MAXIMUM_CAPACITY);
            if (t > threshold)
                threshold = tableSizeFor(t);
        }
        else if (s > threshold)
            resize();
        for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {
            K key = e.getKey();
            V value = e.getValue();
            putVal(hash(key), key, value, false, evict);
        }
    }
}

static final int MAXIMUM_CAPACITY = 1 << 30;

/**
 * The table, initialized on first use, and resized as
 * necessary. When allocated, length is always a power of two.
 * (We also tolerate length zero in some operations to allow
 * bootstrapping mechanics that are currently not needed.)
 */
transient Node<K,V>[] table;

static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    V value;
    Node<K,V> next; //对下一个节点的引用（看到链表的内容，结合定义的Entry数组，哈希表的链地址法!!!实现）

    Node(int hash, K key, V value, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }

    public final K getKey()        { return key; }
    public final V getValue()      { return value; }
    public final String toString() { return key + "=" + value; }

    public final int hashCode() {
        return Objects.hashCode(key) ^ Objects.hashCode(value);
    }

    public final V setValue(V newValue) {
        V oldValue = value;
        value = newValue;
        return oldValue;
    }

    public final boolean equals(Object o) {
        if (o == this)
            return true;
        if (o instanceof Map.Entry) {
            Map.Entry<?,?> e = (Map.Entry<?,?>)o;
            if (Objects.equals(key, e.getKey()) &&
                Objects.equals(value, e.getValue()))
                return true;
        }
        return false;
    }
}

public static boolean equals(Object a, Object b) {
    return (a == b) || (a != null && a.equals(b));
}

public static int hashCode(Object o) {
   return o != null ? o.hashCode() : 0;
}

```
`putMapEntries`方法用来装填映射关系，该方法被构造函数和`Map.putAll()`方法调用，第二个参数就是用来区分的，被构造函数调用的时候为false，被`putAll`方法调用时参数为true。方法首先检查旧映射的长度，存在映射（长度大于0）的时候初始化`table`变量，这是一个`Node`数组，这此时真正用来存储映射关系的数据结构。`Node<K, V>`继承了`Map.Entry<K, V>`，数据域中有`Node`类型变量`next`，实际上，它构成了一个链表。从这可以看出，`HashMap`是基于**哈希桶数组 + 链表**实现存储的，实际上，这是JDK1.7之前的是实现方式，1.8之后，以**哈希桶数组 + 链表/红黑树**的方式实现的。现在在这里还看不到红黑树的影子，往下分析就知道了。

注意到`Node`数据结构中，`hash`和`key`域是常量，一旦被赋值就不能改变。还需要注意到`hashCode`和`equals`方法的写法，其中`hashCode`方法返回的是key和value的本身哈希值的异或结果。而`equals`方法的参数`Object`类型，先用==符号比较两者是不是指向同一个地址，然后进行类型检查，必须得让key和value都相等才行。这里面用到了`Objects`辅助类的相关方法。
> JDK1.6没有用到Objects类，而是单纯的代码赋值

重新回到`putMapEntries`方法，当`table == null`的时候，表示`HashMap`实例并未分配真正的存储空间，所以接下来要做的事情就是分配存储空间了。这个过程实际上是权衡各种因素之后给`threshold`赋值，当旧映射容量大于`threshold`时，进行扩容，`resize`方法定义如下：
```java
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) {
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    else {               // zero initial threshold signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null) {
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { // preserve order
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}

// 默认的初始容量是16，必须是2的幂。
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

```
当`threshold`为0的时候，就用`DEFAULT_INITIAL_CAPACITY`作为容量大小，该值为16（这也就是上文所说在未指定容量大小的时候默认容量为16的原因）。而`threshold`赋值为`DEFAULT_INITIAL_CAPACITY * DEFAULT_LOAD_FACTOR`，否则容量扩大为原来的2倍。

扩容完成后就要将旧映射中的内容赋值到自己的`table`中，调用的是`putVal`方法，定义如下：
```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    //判断table是否为空
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;//创建一个新的table数组，用resize确定大小，并且获取该数组的长度

     //根据键值key计算hash值得到插入的数组索引i，如果table[i]==null，直接新建节点添加
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else { //如果对应的节点存在
        Node<K,V> e; K k;
        //判断table[i]的首个元素是否和key一样，如果相同直接覆盖value
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        //判断table[i] 是否为treeNode，即table[i] 是否是红黑树，如果是红黑树，则直接在树中插入键值对
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
      // 该链为链表，就用链地址法
        else {
      //遍历table[i]，判断链表长度是否大于TREEIFY_THRESHOLD(默认值为8)，大于8的话把链表转换为红黑树，在红黑树中执行插入操作，否则进行链表的插入操作；遍历过程中若发现key已经存在直接覆盖value即可；
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                //树转型
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        // 写入覆盖
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;

}

Node<K,V> newNode(int hash, K key, V value, Node<K,V> next) {
    return new Node<>(hash, key, value, next);
}

//！！Java 8 HashMap的分离链表。 在没有降低哈希冲突的度的情况下，使用红黑书代替链表。
/*
使用链表还是树，与一个哈希桶中的元素数目有关。下面两个参数中展示了Java 8的HashMap在使用树和使用链表之间切换的阈值。当冲突的元素数增加到8时，链表变为树；当减少至6时，树切换为链表。中间有2个缓冲值的原因是避免频繁的切换浪费计算机资源。
*/
static final int TREEIFY_THRESHOLD = 8;
static final int UNTREEIFY_THRESHOLD = 6;
```
这是一个非常非常重要的方法，有四个参数：
1. hash：这里是指key的哈希值，实现调用了`hash()`方法，定义如下：
```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```
可以看到key求取hash值的方法：当key为null的时候为0，否则等于该值的哈希值和其除以2^{16}的商的异或。**所以，`HashMap`可以存储一个key为null的键值对，因为null的哈希值为0，但是可以存储多个value为null的键值对**
2. key：即键的值
3. value：键值对中的“值”
4. onlyIfAbsent： 如果为true，就不改变已经存在的值
5. evict：前一个值的大小，如果无前一个值，就为null。

方法的实现过程比较复杂，关键的注释已经在源码中标出。这里有一个关键的地方需要特别注意。

首先是key=null的值一定在table[0]的位置，为什么呢？因为根据`hash`方法的实现，null的返回值是0，所以在进行第二个if语句判断的时候，`i = (n - 1) & hash`定位到索引为0，所以key=null的值一定被放置在了table[0]。

再者，根据哈希值找到了table[i]，需要判断该位置是红黑树还是链表。如果是红黑树，调用`putTreeVal`方法插入节点。否则就是链表，那么就用链地址法解决问题。在解决的过程中，如果发现冲突的数量大于约定的允许的冲突最大值（默认为8），就将链表转为红黑树，然后执行插入节点的操作，否则就按照链表的插入操作进行。注意如果已经存在key，参数`onlyIfAbsent`决定了是否要覆盖旧的value值。

> 红黑树的实现很重要，我们另外起一章来讲解。

最后size增加1（当然，如果本来就存在key值，size是不会变化的），这里的size是实际存储的映射关系真实个数，和“容量”是不同的概念。此时要判断是否需要进行扩容操作。

另外，源码中`afterNodeAccess`和`afterNodeInsertion`是两个回调方法，方法体都为空。



###  public V get(Object key){...}
取得指定key的value值。定义如下：
```java
public V get(Object key) {
   Node<K,V> e;
   return (e = getNode(hash(key), key)) == null ? null : e.value;
}

final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        if ((e = first.next) != null) {
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```
方法将返回key值对应的value值，如果不存在则返回null。`getNode`方法每次都是检查first节点，否则的话，检查剩余的节点，需要考虑剩下的节点是红黑树节点还是链表。


### public V put(K key, V value) {...}
添加一组映射关系。定义如下：
```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
```
调用的是`putVal`方法，上文已经讲解过。

### public boolean containsKey(Object key) {...}
判断是否存在key为某个值的映射关系。定义如下：
```java
public boolean containsKey(Object key) {
    return getNode(hash(key), key) != null;
}
```
调用`getNode`方法，该方法没有找到该节点的时候将返回`null`。

### public void putAll(Map<? extends K, ? extends V> m) {...}
将旧映射中的映射添加到新映射中，同理还是调用了`putMapEntries`方法，与`HashMap`调用时不一样的地方在于，这里第二个参数为true。
```java
public void putAll(Map<? extends K, ? extends V> m) {
    putMapEntries(m, true);
}
```

### public V remove(Object key){...}
移除指定key的映射关系，如果移除成功就返回被移除的映射，否则返回null。
```java
public V remove(Object key) {
    Node<K,V> e;
    return (e = removeNode(hash(key), key, null, false, true)) == null ?
        null : e.value;
}
final Node<K,V> removeNode(int hash, Object key, Object value,
                               boolean matchValue, boolean movable) {
    Node<K,V>[] tab; Node<K,V> p; int n, index;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (p = tab[index = (n - 1) & hash]) != null) {
        Node<K,V> node = null, e; K k; V v;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            node = p;
        else if ((e = p.next) != null) {
            if (p instanceof TreeNode)
                node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
            else {
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key ||
                         (key != null && key.equals(k)))) {
                        node = e;
                        break;
                    }
                    p = e;
                } while ((e = e.next) != null);
            }
        }
        if (node != null && (!matchValue || (v = node.value) == value ||
                             (value != null && value.equals(v)))) {
            if (node instanceof TreeNode)
                ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
            else if (node == p)
                tab[index] = node.next;
            else
                p.next = node.next;
            ++modCount;
            --size;
            afterNodeRemoval(node);
            return node;
        }
    }
    return null;
}
```
`removeNode`和`putVal`是刚好相反的操作，过程差不多。需要注意的是`removeNode`方法有一个`matchValue`的布尔型参数，这个参数的意义在于考虑在匹配的时候是否考虑匹配参数中的value值，为什么有这种设计呢？因为后面还有一个`remove`方法，该方法指定了特定的key和value，同样调用`removeNode`方法，这时候，我们就需要匹配value值了。而对于只有一个参数的`remove`方法，为了能复用`removeNode`方法，参数value是指定为null的，这时候我们就不用匹配这个value值。


### public boolean remove(Object key, Object value){...}
与上一个remove不同的是，这个`remove`方法删除的是一个指定的映射。定义如下：
```java
public boolean remove(Object key, Object value) {
    return removeNode(hash(key), key, value, true, true) != null;
}
```

> 实际上，对于存在于map中的一条映射key-value，调用`remove(key)`和`remove(key, value)`的效果是一样的，因为key和key-value都是唯一的，不会重复，所以当存在这种映射的时候，两个方法找到的节点是同一个，所以执行结果是一样的。

###  public void clear() {...}
清空映射。定义如下：
```java
public void clear() {
    Node<K,V>[] tab;
    modCount++;
    if ((tab = table) != null && size > 0) {
        size = 0;
        for (int i = 0; i < tab.length; ++i)
            tab[i] = null;
    }
}
```
可以是将多有的哈希桶置为null。那么丢掉的链表/红黑树节点怎么办呢？不是应该一个个释放掉以帮助gc么？？？

### public boolean containsValue(Object value) {...}
如果映射关系中存在一个或多个value等于指定值的映射，就返回true。
```java
public boolean containsValue(Object value) {
    Node<K,V>[] tab; V v;
    if ((tab = table) != null && size > 0) {
        for (int i = 0; i < tab.length; ++i) {
            for (Node<K,V> e = tab[i]; e != null; e = e.next) {
                if ((v = e.value) == value ||
                    (value != null && value.equals(v)))
                    return true;
            }
        }
    }
    return false;
}
```
可以看到这里实际上用到的是for双重循环，外层遍历了哈希桶数组，内容遍历了链表。这里有一个问题，为什么此时部分为链表或红黑树的情况？？？

### public Set<K> keySet(){...}
将返回该映射的所有key值集合。注意的是，如果对该key值做修改，这种修改将会反映到map中，反之亦然。为什么呢？因为它实现的本质是迭代器(Iterator)！！！定义如下：
```java
public Set<K> keySet() {
   Set<K> ks = keySet;
   if (ks == null) {
       ks = new KeySet();
       keySet = ks;
   }
   return ks;
}
transient Set<K> keySet;
```
从上面方法中可以看出，该方法实际上是返回了实例变量`keySet`，这个实例在该方法在第一次被调用的时候被赋值为一个`KeySet`类的实例，而`KeySet`类的定义如下：
```java
final class KeySet extends AbstractSet<K> {
    public final int size()                 { return size; }
    public final void clear()               { HashMap.this.clear(); }
    public final Iterator<K> iterator()     { return new KeyIterator(); }
    public final boolean contains(Object o) { return containsKey(o); }
    public final boolean remove(Object key) {
        return removeNode(hash(key), key, null, false, true) != null;
    }
    public final Spliterator<K> spliterator() {
        return new KeySpliterator<>(HashMap.this, 0, -1, 0, 0);
    }
    public final void forEach(Consumer<? super K> action) {
        Node<K,V>[] tab;
        if (action == null)
            throw new NullPointerException();
        if (size > 0 && (tab = table) != null) {
            int mc = modCount;
            for (int i = 0; i < tab.length; ++i) {
                for (Node<K,V> e = tab[i]; e != null; e = e.next)
                    action.accept(e.key);
            }
            if (modCount != mc)
                throw new ConcurrentModificationException();
        }
    }
}

final class KeyIterator extends HashIterator
    implements Iterator<K> {
    public final K next() { return nextNode().key; }
}
```
这个类只有默认的无参的构造方法，`KeySet`的继承关系是：`KeySet`-`AbstractSet`-`AbstractCollection`-`Object`，但是检查了一圈，发现无参构造方法都是空的，然而debug跟踪发现，在`new KeySet()`方法之后`keySet`确实有值了，到底是什么时候取得这个值的呢？我觉的很奇怪？？？

但是可以确定的是，keySet的获取是依靠迭代器实现的，所以对keySet的修改类似于对迭代器的操作，返回的是一个值的拷贝值。但是如果这个值是一个引用，那么对这个“拷贝值”的修改将会影响原映射。

这个方法还有一个疑问：该方法只在keySet为null的时候进行new，如果我进行下面这种实验：
```java
public static void main(String[] args) {
   Map<String, String> map = new HashMap<>();

   map.put("test1", "test1");
   map.put("test2", "test2");
   map.put("test3", "test3");

   Set<String> keys = map.keySet();
   for (String key : keys){
       System.out.print(key + " ");
   }

   map.put("test4", "test4");

   keys = map.keySet(); //第二次取得keySet
   for (String key : keys){ //这里到底能不能输出test4呢？
       System.out.print(key + " ");
   }
}
```
那么我第二次取得`keySet`的时候，是否可以取出"test4"呢？经过实验是可以的，在进行`Set<K> ks = keySet;`这一步骤时，就可以观察到此时`keyset`包含了该值，但是我记得在之前`putVal`的时候，并没有对keyset有过什么处理，那么`keySet`到底是什么时候产生变化的呢？？？

### public Collection<V> values() {...}
该方法返回映射关系中值的集合。定义如下：
```java
public Collection<V> values() {
    Collection<V> vs = values;
    if (vs == null) {
        vs = new Values();
        values = vs;
    }
    return vs;
}

transient Collection<V> values;

final class Values extends AbstractCollection<V> {
    public final int size()                 { return size; }
    public final void clear()               { HashMap.this.clear(); }
    public final Iterator<V> iterator()     { return new ValueIterator(); }
    public final boolean contains(Object o) { return containsValue(o); }
    public final Spliterator<V> spliterator() {
        return new ValueSpliterator<>(HashMap.this, 0, -1, 0, 0);
    }
    public final void forEach(Consumer<? super V> action) {
        Node<K,V>[] tab;
        if (action == null)
            throw new NullPointerException();
        if (size > 0 && (tab = table) != null) {
            int mc = modCount;
            for (int i = 0; i < tab.length; ++i) {
                for (Node<K,V> e = tab[i]; e != null; e = e.next)
                    action.accept(e.value);
            }
            if (modCount != mc)
                throw new ConcurrentModificationException();
        }
    }
}
```
和`keyset`一样，`values`的修改同样基于迭代器的实现，所以对`values`的修改会反映到map中。

### public Set&lt;Map.Entry&lt;K,V&gt;&gt; entrySet() {...}
这个方法将会返回映射中的所有映射关系，定义如下：
```java
public Set<Map.Entry<K,V>> entrySet() {
    Set<Map.Entry<K,V>> es;
    return (es = entrySet) == null ? (entrySet = new EntrySet()) : es;
}

transient Set<Map.Entry<K,V>> entrySet;

final class EntrySet extends AbstractSet<Map.Entry<K,V>> {
    public final int size()                 { return size; }
    public final void clear()               { HashMap.this.clear(); }
    public final Iterator<Map.Entry<K,V>> iterator() {
        return new EntryIterator();
    }
    public final boolean contains(Object o) {
        if (!(o instanceof Map.Entry))
            return false;
        Map.Entry<?,?> e = (Map.Entry<?,?>) o;
        Object key = e.getKey();
        Node<K,V> candidate = getNode(hash(key), key);
        return candidate != null && candidate.equals(e);
    }
    public final boolean remove(Object o) {
        if (o instanceof Map.Entry) {
            Map.Entry<?,?> e = (Map.Entry<?,?>) o;
            Object key = e.getKey();
            Object value = e.getValue();
            return removeNode(hash(key), key, value, true, true) != null;
        }
        return false;
    }
    public final Spliterator<Map.Entry<K,V>> spliterator() {
        return new EntrySpliterator<>(HashMap.this, 0, -1, 0, 0);
    }
    public final void forEach(Consumer<? super Map.Entry<K,V>> action) {
        Node<K,V>[] tab;
        if (action == null)
            throw new NullPointerException();
        if (size > 0 && (tab = table) != null) {
            int mc = modCount;
            for (int i = 0; i < tab.length; ++i) {
                for (Node<K,V> e = tab[i]; e != null; e = e.next)
                    action.accept(e);
            }
            if (modCount != mc)
                throw new ConcurrentModificationException();
        }
    }
}
```
同样基于迭代器的实现，对其操作的影响将反映到原映射中。`entrySet`常常用来遍历映射，比`keySet`和`values`更具有好的性能，当然这也得看具体的场合。

> 注意到，`keySet`和`entrySet`都是`Set`，因为key和key-value都是彼此唯一的，而`values`是`Collection`类型，因为允许存在重复元素。

一般来说，对map的遍历借助于`keySet`、`values`或者`entrySet`。前两者只能得到key或value，最后一个能得到两者，所以使用地更加普遍一些。下面是一个测试程序：
```java
public static void main(String[] args) {
   Map<String, String> map = new HashMap<>();

   map.put("test1", "test1");
   map.put("test2", "test2");
   map.put("test3", "test3");
   map.put("test", "test1");

   Collection<String> values = map.values();
   for (String value : values){
       System.out.print(value  + " "); //test2 test3 test1 test1
   }

   Set<String> keys = map.keySet();
   for (String key : keys){
       System.out.print(key  + " "); //test2 test3 test test1
   }


   Set<Map.Entry<String, String>> entries = map.entrySet();
   for (Map.Entry<String, String> entry : entries){
       System.out.print(entry.getKey() + " - " + entry.getValue() + " ");//test2 - test2 test3 - test3 test - test1 test1 - test1
   }

}
```
可以看到这三种遍历方式的特点，以及注意到`HashMap`不能保证元素的顺序的特点。

### public V getOrDefault(Object key, V defaultValue){...}
这个方法与`get`方法不同的地方在于：如果找不到指定key对应的value值，就返回参数指定的默认value。实现和`get`方法几乎一样，就是将null换成`defaultValue`而已。
```java
@Override
public V getOrDefault(Object key, V defaultValue) {
   Node<K,V> e;
   return (e = getNode(hash(key), key)) == null ? defaultValue : e.value;
}
```
### public V putIfAbsent(K key, V value) {...}
从名字就可以看出来，进行非覆盖型的put，还记得我们之前`putVal`有一个参数用来控制覆盖还是非覆盖么？之前的`put`和`HashMap(Map<? extends K, ? extends V>)`调用`putVal`的时候，该参数一直是false，而该方法的参数使用true即可。



> 另外还有一些方法，比如`size`、`isEmpty`、`forEach`等方法，有的太简单，有的不常用这里就先学习了，

> `ConcurrentHashMap`是另一种`AbstractMap`的实现，它过滤掉了key和value等于null的情况（抛出异常），这个`Hashtable`有点像。

参考
* [笔记001--Hashtable/HashMap与key/value为null的关系](http://blog.csdn.net/niuwei22007/article/details/52005329)
* [深入Java基础（四）--哈希表（1）HashMap应用及源码详解](http://blog.csdn.net/jack__frost/article/details/69388422)
* [java1.8 HahMap的改进](http://blog.csdn.net/seudongnan/article/details/60885223)
