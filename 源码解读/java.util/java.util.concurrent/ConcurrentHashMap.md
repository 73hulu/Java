# ConcurrentHashMap

`ConcurrentHashMap`是实现高并发、高吞吐量的线程安全的`HashMap`，这个类服从与之相同的功能规范`Hashtable`（`Hashtable`是也是线程安全的），并且包括与每个方法相对应的方法的版本`Hashtable`。但是，即使所有操作都是线程安全的，检索操作也不需要锁定，并且也不支持以阻止所有访问的方式锁定整个表。这个类Hashtable在依赖于线程安全性的程序中是完全可互操作的，但是不依赖于它的同步细节。

![ConcurrentHashMap](http://ovn0i3kdg.bkt.clouddn.com/ConcurrentHashMap_1.png?imageView/2/w/300)
![ConcurrentHashMap](http://ovn0i3kdg.bkt.clouddn.com/ConcurrentHashMap_2.png?imageView/2/w/300)
![ConcurrentHashMap](http://ovn0i3kdg.bkt.clouddn.com/ConcurrentHashMap_3.png?imageView/2/w/300)
![ConcurrentHashMap](http://ovn0i3kdg.bkt.clouddn.com/ConcurrentHashMap_4.png?imageView/2/w/300)
![ConcurrentHashMap](http://ovn0i3kdg.bkt.clouddn.com/ConcurrentHashMap_5.png?imageView/2/w/300)

`ConcurrentHashMap`是非常非常重要，在不同版本中进行了改版。这里着重讲解JDK1.8中`ConcurrentHashMap`的实现。

> JDK1.8中`ConcurrentMap`多采用CAS算法来实现并发，这是一种乐观锁。而1.6和1.7中使用的Segment是可重入锁，是悲观锁。

## public class ConcurrentHashMap<K,V> extends AbstractMap<K,V> implements ConcurrentMap<K,V>, Serializable
类声明，继承自`AbstractMap`，实现`ConcurrentMap`，这个接口的结构如下：

![ConcurrentMap](https://ws1.sinaimg.cn/large/0069RVTdly1fuxbsge1pdj30j00co3yq.jpg)


## 重要的几个变量
* 容量
表示目前map的存储大小，在源码中分为默认和最大，默认是在没有指定容量大小的时候会赋予这个值，最大表示当容量达到这个值时，不再支持扩容。
```java
/**
 * The largest possible table capacity.  This value must be
 * exactly 1<<30 to stay within Java array allocation and indexing
 * bounds for power of two table sizes, and is further required
 * because the top two bits of 32bit hash fields are used for
 * control purposes.
 */
private static final int MAXIMUM_CAPACITY = 1 << 30;  
/**
 * The default initial table capacity.  Must be a power of 2
 * (i.e., at least 1) and at most MAXIMUM_CAPACITY.
 */
private static final int DEFAULT_CAPACITY = 16;
```
* 加载因子
`laodfactor`的含义与`HashMap`中一样，默认值也是0.75。
```java
/**
 * The load factor for this table. Overrides of this value in
 * constructors affect only the initial table capacity.  The
 * actual floating point value isn't normally used -- it is
 * simpler to use expressions such as {@code n - (n >>> 2)} for
 * the associated resizing threshold.
 */
private static final float LOAD_FACTOR = 0.75f;
```
* 单链表和红黑树转换的阈值
```java
/**
 * The bin count threshold for using a tree rather than list for a
 * bin.  Bins are converted to trees when adding an element to a
 * bin with at least this many nodes. The value must be greater
 * than 2, and should be at least 8 to mesh with assumptions in
 * tree removal about conversion back to plain bins upon
 * shrinkage.
 */
static final int TREEIFY_THRESHOLD = 8;
/**
 * The bin count threshold for untreeifying a (split) bin during a
 * resize operation. Should be less than TREEIFY_THRESHOLD, and at
 * most 6 to mesh with shrinkage detection under removal.
 */
static final int UNTREEIFY_THRESHOLD = 6;
```
*  扩容相关
主要是在扩容和参与扩容（当线程进入put的时候，发现该map正在扩容，那么它会协助扩容）的时候使用。具体使用方法参见下文中的应用。
```java
/**
 * The number of bits used for generation stamp in sizeCtl.
 * Must be at least 6 for 32bit arrays.
 */
private static int RESIZE_STAMP_BITS = 16;
/**
 * The maximum number of threads that can help resize.
 * Must fit in 32 - RESIZE_STAMP_BITS bits.
 */
private static final int MAX_RESIZERS = (1 << (32 - RESIZE_STAMP_BITS)) - 1;
/**
 * The bit shift for recording size stamp in sizeCtl.
 */
private static final int RESIZE_STAMP_SHIFT = 32 - RESIZE_STAMP_BITS;
```

* 节点状态
线程判断map当前处于什么阶段。
```java
static final int MOVED     = -1; // hash for forwarding nodes 表明该节点是forwarding Node，表示有线程处理过了。
static final int TREEBIN   = -2; // hash for roots of trees， 表示这是一个树节点
static final int RESERVED  = -3; // hash for transient reservations
static final int HASH_BITS = 0x7fffffff; // usable bits of normal node hash
```
* `sizeCtl`
`private transient volatile int sizeCtl;`非常非常重要的参数。出现在`ConcurrentHashMap`的各个阶段，不同的值也表示不同情况和不同功能：
  1. -1代表正在进行初始化或扩容操作。 -N。表示有N-1个线程正在进行扩容操作 （当线程进行值添加的时候判断到正在扩容，它就会协助扩容）
  2. 0表示当前table还没有被初始化
  3. 正数。表示初始化或者下一次进行扩容的大小。它的值始终是当前ConcurrentHashMap容量的0.75倍，这与loadfactor是对应的。实际容量>=sizeCtl，则扩容。
  注意：在某些情况下，这个值就相当于HashMap中的threshold阀值。用于控制扩容。

## 重要的内部类
### 节点Node
这就是`ConcurrentHashMap`中数据的存储单元。
```java
/**
 * Key-value entry.  This class is never exported out as a
 * user-mutable Map.Entry (i.e., one supporting setValue; see
 * MapEntry below), but can be used for read-only traversals used
 * in bulk tasks.  Subclasses of Node with a negative hash field
 * are special, and contain null keys and values (but are never
 * exported).  Otherwise, keys and vals are never null.
 */
static class Node<K,V> implements Map.Entry<K,V> {
    //链表的数据结构
    final int hash;
    final K key;
    //val和next都会在扩容时发生变化，所以加上volatile来保持可见性和禁止重排序
    volatile V val;
    volatile Node<K,V> next;

    Node(int hash, K key, V val, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.val = val;
        this.next = next;
    }

    public final K getKey()       { return key; }
    public final V getValue()     { return val; }
    public final int hashCode()   { return key.hashCode() ^ val.hashCode(); }
    public final String toString(){ return key + "=" + val; }
    // 不允许更新value
    public final V setValue(V value) {
        throw new UnsupportedOperationException();
    }

    public final boolean equals(Object o) {
        Object k, v, u; Map.Entry<?,?> e;
        return ((o instanceof Map.Entry) &&
                (k = (e = (Map.Entry<?,?>)o).getKey()) != null &&
                (v = e.getValue()) != null &&
                (k == key || k.equals(key)) &&
                (v == (u = val) || v.equals(u)));
    }

    /**
     * Virtualized support for map.get(); overridden in subclasses.
     */
    //用于map中的get（）方法，子类重写
    Node<K,V> find(int h, Object k) {
        Node<K,V> e = this;
        if (k != null) {
            do {
                K ek;
                if (e.hash == h &&
                    ((ek = e.key) == k || (ek != null && k.equals(ek))))
                    return e;
            } while ((e = e.next) != null);
        }
        return null;
    }
}
```
可以看到，这几乎就是`HashMap`中节点的数据结构一致，不同点在于`value`和`next`用`volatile`修饰，保证了可见性和有序性，从而保证了线程安全。另外需要注意的是，`setValue`方法会抛出异常，避免了直接对值进行修改，这是为了解决并发问题只能用ConcurrentHashMap封装的CAS操作。`find`方法用来用户寻找节点，把查找的工作交给了节点，其实也是因为加入了红黑树所以查找单独封装在相应节点更好（并发）。即只读不写。




### TreeNode和TreeBin
当桶的长度大于8的时候，`ConcurrentHashMap`也会进行单链表到红黑树的转化。不同的是，`HashMap`中，红黑树桶的承载的是`TreeNode`，而`ConcurrentHashMap`中红黑树桶中承载的是`TreeBin`，这里封装了红黑树删除及旋转节点操作，并且保存了树的根节点，里面才是真正的`TreeNode`。
```java
/**
 * Nodes for use in TreeBins
 */
static final class TreeNode<K,V> extends Node<K,V> {
    TreeNode<K,V> parent;  // red-black tree links
    TreeNode<K,V> left;
    TreeNode<K,V> right;
    TreeNode<K,V> prev;    // needed to unlink next upon deletion
    boolean red;

    TreeNode(int hash, K key, V val, Node<K,V> next,
             TreeNode<K,V> parent) {
        super(hash, key, val, next);
        this.parent = parent;
    }

    Node<K,V> find(int h, Object k) {
        return findTreeNode(h, k, null);
    }

    /**
     * Returns the TreeNode (or null if not found) for the given key
     * starting at given root.
     */
    final TreeNode<K,V> findTreeNode(int h, Object k, Class<?> kc) {
        if (k != null) {
            TreeNode<K,V> p = this;
            do  {
                int ph, dir; K pk; TreeNode<K,V> q;
                TreeNode<K,V> pl = p.left, pr = p.right;
                if ((ph = p.hash) > h)
                    p = pl;
                else if (ph < h)
                    p = pr;
                else if ((pk = p.key) == k || (pk != null && k.equals(pk)))
                    return p;
                else if (pl == null)
                    p = pr;
                else if (pr == null)
                    p = pl;
                else if ((kc != null ||
                          (kc = comparableClassFor(k)) != null) &&
                         (dir = compareComparables(kc, k, pk)) != 0)
                    p = (dir < 0) ? pl : pr;
                else if ((q = pr.findTreeNode(h, k, kc)) != null)
                    return q;
                else
                    p = pl;
            } while (p != null);
        }
        return null;
    }
}

//TreeBin太长，笔者截取了它的构造方法：
TreeBin(TreeNode<K,V> b) {
    super(TREEBIN, null, null, null);
    this.first = b;
    TreeNode<K,V> r = null;
    for (TreeNode<K,V> x = b, next; x != null; x = next) {
        next = (TreeNode<K,V>)x.next;
        x.left = x.right = null;
        if (r == null) {
            x.parent = null;
            x.red = false;
            r = x;
        }
        else {
            K k = x.key;
            int h = x.hash;
            Class<?> kc = null;
            for (TreeNode<K,V> p = r;;) {
                int dir, ph;
                K pk = p.key;
                if ((ph = p.hash) > h)
                    dir = -1;
                else if (ph < h)
                    dir = 1;
                else if ((kc == null &&
                          (kc = comparableClassFor(k)) == null) ||
                         (dir = compareComparables(kc, k, pk)) == 0)
                    dir = tieBreakOrder(k, pk);
                    TreeNode<K,V> xp = p;
                if ((p = (dir <= 0) ? p.left : p.right) == null) {
                    x.parent = xp;
                    if (dir <= 0)
                        xp.left = x;
                    else
                        xp.right = x;
                    r = balanceInsertion(r, x);
                    break;
                }
            }
        }
    }
    this.root = r;
    assert checkInvariants(root);
}
```
从上面的源码可以看出，在`ConcurrentHashMap`中不是直接存储`TreeNode`来实现的，而是用`TreeBin`来包装`TreeNode`来实现的。也就是说在实际的`ConcurrentHashMap`桶中，存放的是`TreeBin`对象，而不是`TreeNode`对象。之所以`TreeNode`继承自`Node`是为了附带`next`指针，而这个`next`指针可以在`TreeBin`中寻找下一个`TreeNode`，而`HashMap`中存储的是`TreeNode`，这里也是与`HashMap`之间比较大的区别。

在`ConcurrentHashMap`中，主要的存储结构定义如下：
```java
/**
 * The array of bins. Lazily initialized upon first insertion.
 * Size is always a power of two. Accessed directly by iterators.
 */
transient volatile Node<K,V>[] table;
```
如此，可以看到存储结构和`HashMap`一样：

![ConcucrrentHashMap存储结构](https://ws1.sinaimg.cn/large/0069RVTdly1fuxcrg0vqhj30e40c9glj.jpg)

### ForwordingNode
使用主要是在扩容阶段，它是链接两个table的节点类，有一个next属性用于指向下一个table，注意要理解这个table，它并不是说有2个table，而是在扩容的时候当线程读取到这个地方发现这个地方为空，这会设置为forwordingNode，或者线程处理完该节点也会设置该节点为forwordingNode，别的线程发现这个forwordingNode会继续向后执行遍历，这样一来就很好的解决了多线程安全的问题。这里有小伙伴就会问，那一个线程开始处理这个节点还没处理完，别的线程进来怎么办，而且这个节点还不是forwordingNode呐？说明你前面没看详细，在处理某个节点（桶里面第一个节点）的时候会对该节点上锁，上面文章中我已经说过了。
```java
/**
 * A node inserted at head of bins during transfer operations.
 */
static final class ForwardingNode<K,V> extends Node<K,V> {
    final Node<K,V>[] nextTable;
    ForwardingNode(Node<K,V>[] tab) {
        super(MOVED, null, null, null);
        this.nextTable = tab;
    }

    // 这里的find方法是从nextTable中查找相应的节点，而不是在当前链表中查找
    Node<K,V> find(int h, Object k) {
        // loop to avoid arbitrarily deep recursion on forwarding nodes
        outer: for (Node<K,V>[] tab = nextTable;;) {
            Node<K,V> e; int n;
            if (k == null || tab == null || (n = tab.length) == 0 ||
                (e = tabAt(tab, (n - 1) & h)) == null)
                return null;
            for (;;) {
                int eh; K ek;
                if ((eh = e.hash) == h &&
                    ((ek = e.key) == k || (ek != null && k.equals(ek))))
                    return e;
                if (eh < 0) {
                    if (e instanceof ForwardingNode) {
                        tab = ((ForwardingNode<K,V>)e).nextTable;
                        continue outer;
                    }
                    else
                        return e.find(h, k);
                }
                if ((e = e.next) == null)
                    return null;
            }
        }
    }
}
```

## final V putVal(K key, V value, boolean onlyIfAbsent) {..}
```java
/*
 * 当添加一对键值对的时候，首先会去判断保存这些键值对的数组是不是初始化了，
 * 如果没有的话就初始化数组
 *  然后通过计算hash值来确定放在数组的哪个位置
 * 如果这个位置为空则直接添加，如果不为空的话，则取出这个节点来
 * 如果取出来的节点的hash值是MOVED(-1)的话，则表示当前正在对这个数组进行扩容，复制到新的数组，则当前线程也去帮助复制
 * 最后一种情况就是，如果这个节点，不为空，也不在扩容，则通过synchronized来加锁，进行添加操作
 *    然后判断当前取出的节点位置存放的是链表还是树
 *    如果是链表的话，则遍历整个链表，直到取出来的节点的key来个要放的key进行比较，如果key相等，并且key的hash值也相等的话，
 *          则说明是同一个key，则覆盖掉value，否则的话则添加到链表的末尾
 *    如果是树的话，则调用putTreeVal方法把这个元素添加到树中去
 *  最后在添加完成之后，会判断在该节点处共有多少个节点（注意是添加前的个数），如果达到8个以上了的话，
 *  则调用treeifyBin方法来尝试将处的链表转为树，或者扩容数组
 */
 final V putVal(K key, V value, boolean onlyIfAbsent) {
    // ConcurrentHashMap不允许key或value为null
    if (key == null || value == null) throw new NullPointerException();
    int hash = spread(key.hashCode()); //两次hash，减少hash冲突，可以均匀分布
    int binCount = 0;
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;
        if (tab == null || (n = tab.length) == 0)
            tab = initTable();
        // 赋值，原子操作
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            if (casTabAt(tab, i, null,
                         new Node<K,V>(hash, key, value, null)))
                break;                   // no lock when adding to empty bin
        }
        else if ((fh = f.hash) == MOVED)  //如果在进行扩容，则先进行扩容操作
            tab = helpTransfer(tab, f);
        else {
            V oldVal = null;
            // 这里是ConcurrentHashMap保证并发性的手段，只对桶中头结点加锁
            // 结点上锁  这里的结点可以理解为hash值相同组成的链表的头结点
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                  //fh 〉0 说明这个节点是一个链表的节点 不是树的节点
                    if (fh >= 0) {
                        binCount = 1;
                        //在这里遍历链表所有的结点
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            //如果hash值和key值相同，根据onlyIfAbsent值修改则修改对应结点的value值
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                oldVal = e.val;
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }
                            Node<K,V> pred = e;
                            //如果遍历到了最后一个结点，那么就证明新的节点需要插入 就把它插入在链表尾部
                            if ((e = e.next) == null) {
                                pred.next = new Node<K,V>(hash, key,
                                                          value, null);
                                break;
                            }
                        }
                    }
                    //如果这个节点是树节点，就按照树的方式插入值
                    else if (f instanceof TreeBin) {
                        Node<K,V> p;
                        binCount = 2;
                        //红黑树结构旋转插入
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                       value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                }
            }
            //如果链表的长度大于8时就会进行红黑树的转换 注意这里判断的是添加之前的数量
            if (binCount != 0) {
                if (binCount >= TREEIFY_THRESHOLD)
                    treeifyBin(tab, i);
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }
    // 统计size，并且检查是否需要扩容
    addCount(1L, binCount);
    return null;
}
```
首先，方法一开始的if判断就表现了`ConcurrentHashMap`的特性，即`key`和`value`都不能为null。

接着，计算key的hash值，这里进行了两次哈希，方法`spread`如下
```java
static final int spread(int h) {
    return (h ^ (h >>> 16)) & HASH_BITS;
}
static final int HASH_BITS = 0x7fffffff; // usable bits of normal node hash
```
这个算法实和`HashMap`一致的。

数据存储引用`table`的初始化方法如下：
```java
/**
 * 初始化数组table，
 * 如果sizeCtl小于0，说明别的数组正在进行初始化，则让出执行权
 * 如果sizeCtl大于0的话，则初始化一个大小为sizeCtl的数组
 * 否则的话初始化一个默认大小(16)的数组
 * 然后设置sizeCtl的值为数组长度的3/4
 */
private final Node<K,V>[] initTable() {
    Node<K,V>[] tab; int sc;
    while ((tab = table) == null || tab.length == 0) {
        if ((sc = sizeCtl) < 0) // sizeCtl < 0表示其他线程已经在初始化了或者扩容了，挂起当前线程
            Thread.yield(); // lost initialization race; just spin
        else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) { // CAS操作SIZECTL为-1，表示初始化状态
            try {
                if ((tab = table) == null || tab.length == 0) {
                    int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                    @SuppressWarnings("unchecked")
                    Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                    table = tab = nt;
                    sc = n - (n >>> 2); // 记录下次扩容的大小
                }
            } finally {
                sizeCtl = sc;
            }
            break;
        }
    }
    return tab;
}
```
这里同样对多线程进行了处理，当sizeCtl为负数时，即正在被扩容和初始化时候，此时当前线程放弃初始化，等待重新获取CPU，这样就保证了table只能由一个线程初始化一次。

当table已经被初始化后，计算得到哈希桶的索引，若此时桶为空，则直接赋值，此时调用的`casTabAt`方法定义如下：
```java
static final <K,V> boolean casTabAt(Node<K,V>[] tab, int i,
                                        Node<K,V> c, Node<K,V> v) {
    return U.compareAndSwapObject(tab, ((long)i << ASHIFT) + ABASE, c, v);
}

// Unsafe mechanics
private static final sun.misc.Unsafe U;
private static final long SIZECTL;
private static final long TRANSFERINDEX;
private static final long BASECOUNT;
private static final long CELLSBUSY;
private static final long CELLVALUE;
private static final long ABASE;
private static final int ASHIFT;

static {
    try {
        U = sun.misc.Unsafe.getUnsafe();
        Class<?> k = ConcurrentHashMap.class;
        SIZECTL = U.objectFieldOffset
            (k.getDeclaredField("sizeCtl"));
        TRANSFERINDEX = U.objectFieldOffset
            (k.getDeclaredField("transferIndex"));
        BASECOUNT = U.objectFieldOffset
            (k.getDeclaredField("baseCount"));
        CELLSBUSY = U.objectFieldOffset
            (k.getDeclaredField("cellsBusy"));
        Class<?> ck = CounterCell.class;
        CELLVALUE = U.objectFieldOffset
            (ck.getDeclaredField("value"));
        Class<?> ak = Node[].class;
        ABASE = U.arrayBaseOffset(ak);
        int scale = U.arrayIndexScale(ak);
        if ((scale & (scale - 1)) != 0)
            throw new Error("data type scale not a power of two");
        ASHIFT = 31 - Integer.numberOfLeadingZeros(scale);
    } catch (Exception e) {
        throw new Error(e);
    }
}
```
注意到这里实际上是调用的`Unsafe`类的方法`compareAndSwapObject`，有四个参数：
①第一个参数为需要改变的对象
②第二个为偏移量(即之前求出来的valueOffset的值)
③第三个参数为期待的值
④第四个为更新后的值。
整个方法的作用即为若调用该方法时，value的值与expect这个值相等，那么则将value修改为update这个值，并返回一个true，如果调用该方法时，value的值与expect这个值不相等，那么不做任何操作，并范围一个false。
通常这个方法被用到循环中，即保证如果调用compareAndSet这个方法返回为false时，能再次尝试进行修改value的值，直到修改成功，并返回修改前value的值。

这样能保证在多线程时具有线程安全性，并且没有使用java中任何锁的机制，所依靠的便是`Unsafe`这个类中调用的该方法具有原子性，这个原子性的保证并不是靠java本身保证，而是靠一个更底层的与操作系统相关的特性实现。

如果索引位置上的节点，即链表的头结点是是一个forwarding节点， 说明此时`ConcurrentHashMap`正在进扩容，那么就调用`helpTransfer`方法，定义如下：
```java
/**
 * 帮助从旧的table的元素复制到新的table中
 */
final Node<K,V>[] helpTransfer(Node<K,V>[] tab, Node<K,V> f) {
    Node<K,V>[] nextTab; int sc;
    if (tab != null && (f instanceof ForwardingNode) &&
        (nextTab = ((ForwardingNode<K,V>)f).nextTable) != null) { //新的table nextTba已经存在前提下才能帮助扩容
        int rs = resizeStamp(tab.length);  // 计算一个操作校验码
        while (nextTab == nextTable && table == tab &&
               (sc = sizeCtl) < 0) {
            if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                sc == rs + MAX_RESIZERS || transferIndex <= 0)
                break;
            if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1)) { //调用扩容方法
                transfer(tab, nextTab);
                break;
            }
        }
        return nextTab;
    }
    return table;
}
/*
* 计算操作校验码
*/
static final int resizeStamp(int n) {
    return Integer.numberOfLeadingZeros(n) | (1 << (RESIZE_STAMP_BITS - 1));
}
```
`helpTransfer`方法的目的就是调用多个工作线程一起帮助进行扩容，这样的效率就会更高，而不是只有检查到要扩容的那个线程进行扩容操作，其他线程就要等待扩容操作完成才能工作。其中`transfer`是真正扩容的方法，定义如下：
```java
/**
 * 把数组中的节点复制到新的数组的相同位置，或者移动到扩张部分的相同位置
 * 在这里首先会计算一个步长，表示一个线程处理的数组长度，用来控制对CPU的使用，
 * 每个CPU最少处理16个长度的数组元素,也就是说，如果一个数组的长度只有16，那只有一个线程会对其进行扩容的复制移动操作
 * 扩容的时候会一直遍历，知道复制完所有节点，每处理一个节点的时候会在链表的头部设置一个fwd节点，这样其他线程就会跳过他，
 * 复制后在新数组中的链表不是绝对的反序的
 */
private final void transfer(Node<K,V>[] tab, Node<K,V>[] nextTab) {
    int n = tab.length, stride;
    // 每核处理的量小于16，则强制赋值16
    if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
        stride = MIN_TRANSFER_STRIDE; // subdivide range
    /*
     * 如果复制的目标nextTab为null的话，则初始化一个table两倍长的nextTab
     * 此时nextTable被设置值了(在初始情况下是为null的)
     * 因为如果有一个线程开始了表的扩张的时候，其他线程也会进来帮忙扩张，
     * 而只是第一个开始扩张的线程需要初始化下目标数组
     */
    if (nextTab == null) {            // initiating
        try {
            @SuppressWarnings("unchecked")
            //构建一个nextTable对象，其容量为原来容量的两倍
            Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n << 1];
            nextTab = nt;
        } catch (Throwable ex) {      // try to cope with OOME
            sizeCtl = Integer.MAX_VALUE;
            return;
        }
        nextTable = nextTab;
        transferIndex = n;
    }
    int nextn = nextTab.length;
    /*
     * 创建一个fwd节点，这个是用来控制并发的，当一个节点为空或已经被转移之后，就设置为fwd节点
     * 这是一个空的标志节点
     */
    ForwardingNode<K,V> fwd = new ForwardingNode<K,V>(nextTab);
    boolean advance = true; // 是否继续向前查找的标志位
    boolean finishing = false; // to ensure sweep(清扫) before committing nextTab,在完成之前重新在扫描一遍数组，看看有没完成的没 nextTab
    for (int i = 0, bound = 0;;) {
        Node<K,V> f; int fh;
        // 控制 --i ,遍历原hash表中的节点
        while (advance) {
            int nextIndex, nextBound;
            if (--i >= bound || finishing)
                advance = false;
            else if ((nextIndex = transferIndex) <= 0) {
                i = -1;
                advance = false;
            }
            // 用CAS计算得到的transferIndex
            else if (U.compareAndSwapInt
                     (this, TRANSFERINDEX, nextIndex,
                      nextBound = (nextIndex > stride ?
                                   nextIndex - stride : 0))) {
                bound = nextBound;
                i = nextIndex - 1;
                advance = false;
            }
        }
        if (i < 0 || i >= n || i + n >= nextn) {
            int sc;
             // 已经完成所有节点复制了
            if (finishing) {
                nextTable = null;
                table = nextTab; // table 指向nextTable
                sizeCtl = (n << 1) - (n >>> 1); // sizeCtl阈值为原来的1.5倍
                return;// 跳出死循环，
            }
            // CAS 更扩容阈值，在这里面sizectl值减一，说明新加入一个线程参与到扩容操作
            if (U.compareAndSwapInt(this, SIZECTL, sc = sizeCtl, sc - 1)) {
                if ((sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT)
                    return;
                finishing = advance = true;
                i = n; // recheck before commit
            }
        }
         //数组中把null的元素设置为ForwardingNode节点(hash值为MOVED[-1])
        else if ((f = tabAt(tab, i)) == null)
            advance = casTabAt(tab, i, null, fwd);
            // f.hash == -1 表示遍历到了ForwardingNode节点，意味着该节点已经处理过了
            // 这里是控制并发扩容的核心
        else if ((fh = f.hash) == MOVED)
            advance = true; // already processed
        else {
            // 节点加锁
            synchronized (f) {
              // 节点复制工作
                if (tabAt(tab, i) == f) {
                    Node<K,V> ln, hn;
                     // fh >= 0 ,表示为链表节点
                    if (fh >= 0) {
                      /*
                       * 因为n的值为数组的长度，且是power(2,x)的，所以，在&操作的结果只可能是0或者n
                       * 根据这个规则
                       *         0-->  放在新表的相同位置
                       *         n-->  放在新表的（n+原来位置）
                       */
                      // 构造两个链表  一个是原链表  另一个是原链表的反序排列
                        int runBit = fh & n;
                        Node<K,V> lastRun = f;
                        for (Node<K,V> p = f.next; p != null; p = p.next) {
                            int b = p.hash & n;
                            if (b != runBit) {
                                runBit = b;
                                lastRun = p;
                            }
                        }
                        if (runBit == 0) {
                            ln = lastRun;
                            hn = null;
                        }
                        else {
                            hn = lastRun;
                            ln = null;
                        }
                        for (Node<K,V> p = f; p != lastRun; p = p.next) {
                            int ph = p.hash; K pk = p.key; V pv = p.val;
                            if ((ph & n) == 0)
                                ln = new Node<K,V>(ph, pk, pv, ln);
                            else
                                hn = new Node<K,V>(ph, pk, pv, hn);
                        }
                        // 在nextTable i 位置处插上链表
                        setTabAt(nextTab, i, ln);
                        // 在nextTable i + n 位置处插上链表
                        setTabAt(nextTab, i + n, hn);
                        // 在table i 位置处插上ForwardingNode 表示该节点已经处理过了
                        setTabAt(tab, i, fwd);
                        // advance = true 可以执行--i动作，遍历节点
                        advance = true;
                    }
                     // 如果是TreeBin，则按照红黑树进行处理，处理逻辑与上面一致
                    else if (f instanceof TreeBin) {
                        TreeBin<K,V> t = (TreeBin<K,V>)f;
                        TreeNode<K,V> lo = null, loTail = null;
                        TreeNode<K,V> hi = null, hiTail = null;
                        int lc = 0, hc = 0;
                        for (Node<K,V> e = t.first; e != null; e = e.next) {
                            int h = e.hash;
                            TreeNode<K,V> p = new TreeNode<K,V>
                                (h, e.key, e.val, null, null);
                            if ((h & n) == 0) {
                                if ((p.prev = loTail) == null)
                                    lo = p;
                                else
                                    loTail.next = p;
                                loTail = p;
                                ++lc;
                            }
                            else {
                                if ((p.prev = hiTail) == null)
                                    hi = p;
                                else
                                    hiTail.next = p;
                                hiTail = p;
                                ++hc;
                            }
                        }
                        // 扩容后树节点个数若<=6，将树转链表
                        ln = (lc <= UNTREEIFY_THRESHOLD) ? untreeify(lo) :
                            (hc != 0) ? new TreeBin<K,V>(lo) : t;
                        hn = (hc <= UNTREEIFY_THRESHOLD) ? untreeify(hi) :
                            (lc != 0) ? new TreeBin<K,V>(hi) : t;
                        setTabAt(nextTab, i, ln);
                        setTabAt(nextTab, i + n, hn);
                        setTabAt(tab, i, fwd);
                        advance = true;
                    }
                }
            }
        }
    }
}
```
> `put`、`transfer`和`get`方法非常重要，基本上是`ConcurrentHashMap`的核心。

如果该链表并没有正在进行扩容，那么就就行正常的扩容，此时进行了加锁处理，即只对头节点加锁，然后根据节点是链表节点还是树节点进行增长和扩容。

最后，等到以上步骤都完成之后，再判断是否已经达到了阈值，是需要进行扩容还是转化为树。

在put方法的详解中，我们可以看到，在同一个节点的个数超过8个的时候，会调用`treeifyBin`方法来看看是扩容还是转化为一棵树。
同时在每次添加完元素的`addCount`方法中，也会判断当前数组中的元素是否达到了`sizeCtl`的量，如果达到了的话，则会进入transfer方法去扩容
```java
/**
* 当数组长度小于64的时候，扩张数组长度一倍，否则的话把链表转为树
*/
private final void treeifyBin(Node<K,V>[] tab, int index) {
   Node<K,V> b; int n, sc;
   if (tab != null) {
       if ((n = tab.length) < MIN_TREEIFY_CAPACITY)
           tryPresize(n << 1); // 数组扩容
       else if ((b = tabAt(tab, index)) != null && b.hash >= 0) {
           synchronized (b) { // 使用synchronized同步器，将该节点出的链表转为树
               if (tabAt(tab, index) == b) {
                   TreeNode<K,V> hd = null, tl = null; // hd：树的头(head)
                   for (Node<K,V> e = b; e != null; e = e.next) {
                       TreeNode<K,V> p =
                           new TreeNode<K,V>(e.hash, e.key, e.val,
                                             null, null);
                       if ((p.prev = tl) == null) // 把Node组成的链表，转化为TreeNode的链表，头结点任然放在相同的位置
                           hd = p; //设置head
                       else
                           tl.next = p;
                       tl = p;
                   }
                   setTabAt(tab, index, new TreeBin<K,V>(hd)); // 把TreeNode的链表放入容器TreeBin中
               }
           }
       }
   }
}
```
`tryPresize`方法定义如下：
```java
/**
 * 扩容表为指可以容纳指定个数的大小（总是2的N次方）
 * 假设原来的数组长度为16，则在调用tryPresize的时候，size参数的值为16<<1(32)，此时sizeCtl的值为12
 * 计算出来c的值为64,则要扩容到sizeCtl≥为止
 *  第一次扩容之后 数组长：32 sizeCtl：24
 *  第二次扩容之后 数组长：64 sizeCtl：48
 *  第二次扩容之后 数组长：128 sizeCtl：94 --> 这个时候才会退出扩容
 */
private final void tryPresize(int size) {
        /*
         * MAXIMUM_CAPACITY = 1 << 30
         * 如果给定的大小大于等于数组容量的一半，则直接使用最大容量，
         * 否则使用tableSizeFor算出来
         * 后面table一直要扩容到这个值小于等于sizeCtrl(数组长度的3/4)才退出扩容
         */
    int c = (size >= (MAXIMUM_CAPACITY >>> 1)) ? MAXIMUM_CAPACITY :
        tableSizeFor(size + (size >>> 1) + 1);
    int sc;
    while ((sc = sizeCtl) >= 0) {
        Node<K,V>[] tab = table; int n;
//            printTable(tab);    调试用的
        /*
         * 如果数组table还没有被初始化，则初始化一个大小为sizeCtrl和刚刚算出来的c中较大的一个大小的数组
         * 初始化的时候，设置sizeCtrl为-1，初始化完成之后把sizeCtrl设置为数组长度的3/4
         * 为什么要在扩张的地方来初始化数组呢？这是因为如果第一次put的时候不是put单个元素，
         * 而是调用putAll方法直接put一个map的话，在putALl方法中没有调用initTable方法去初始化table，
         * 而是直接调用了tryPresize方法，所以这里需要做一个是不是需要初始化table的判断
         */
        if (tab == null || (n = tab.length) == 0) {
            n = (sc > c) ? sc : c;
            if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {    //初始化tab的时候，把sizeCtl设为-1
                try {
                    if (table == tab) {
                        @SuppressWarnings("unchecked")
                        Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                        table = nt;
                        sc = n - (n >>> 2);
                    }
                } finally {
                    sizeCtl = sc;
                }
            }
        }
        /*
         * 一直扩容到的c小于等于sizeCtl或者数组长度大于最大长度的时候，则退出
         * 所以在一次扩容之后，不是原来长度的两倍，而是2的n次方倍
         */
        else if (c <= sc || n >= MAXIMUM_CAPACITY) {
                break;    //退出扩张
        }
        else if (tab == table) {
            int rs = resizeStamp(n);
            /*
             * 如果正在扩容Table的话，则帮助扩容
             * 否则的话，开始新的扩容
             * 在transfer操作，将第一个参数的table中的元素，移动到第二个元素的table中去，
             * 虽然此时第二个参数设置的是null，但是，在transfer方法中，当第二个参数为null的时候，
             * 会创建一个两倍大小的table
             */
            if (sc < 0) {
                Node<K,V>[] nt;
                if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                    sc == rs + MAX_RESIZERS || (nt = nextTable) == null ||
                    transferIndex <= 0)
                    break;
                /*
                 * transfer的线程数加一,该线程将进行transfer的帮忙
                 * 在transfer的时候，sc表示在transfer工作的线程数
                 */
                if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))
                    transfer(tab, nt);
            }
            /*
             * 没有在初始化或扩容，则开始扩容
             */
            else if (U.compareAndSwapInt(this, SIZECTL, sc,
                                         (rs << RESIZE_STAMP_SHIFT) + 2)) {
                    transfer(tab, null);
            }
        }
    }
}
```

## public V get(Object key) {...}
`get`是指定一个key来确定value的时候，必须满足两个条件①key相同②hash值相同，对于节点可能在链表或树上的情况，需要分别去查找。
```java
/**
 * Returns the value to which the specified key is mapped,
 * or {@code null} if this map contains no mapping for the key.
 *
 * <p>More formally, if this map contains a mapping from a key
 * {@code k} to a value {@code v} such that {@code key.equals(k)},
 * then this method returns {@code v}; otherwise it returns
 * {@code null}.  (There can be at most one such mapping.)
 *
 * @throws NullPointerException if the specified key is null
 */
public V get(Object key) {
    Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
    // 计算hash值
    int h = spread(key.hashCode());
    // 根据hash值确定节点位置
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (e = tabAt(tab, (n - 1) & h)) != null) {
          //如果搜索到的节点key与传入的key相同且不为null,直接返回这个节点
        if ((eh = e.hash) == h) {
            if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                return e.val;
        }
        //如果eh<0 说明这个节点在树上 直接寻找
        else if (eh < 0)
            return (p = e.find(h, key)) != null ? p.val : null;
        //否则遍历链表 找到对应的值并返回
        while ((e = e.next) != null) {
            if (e.hash == h &&
                ((ek = e.key) == key || (ek != null && key.equals(ek))))
                return e.val;
        }
    }

    return null;
}
```
可以看到，在`get`过程中，不需要加锁的操作，与`HashMap`不同的是，在红黑树的查找过程中，使用了辅助查找方法`find`

## size相关方法
在`ConcurrentHashMap`中，计算元素个数size是一件不容易的事情，因为在计算size的时候，可能有线程正在进行扩容。所以它不能计算得到一个精确值。
首先看下`size`方法的定义：
```java
public int size() {
    long n = sumCount();
    return ((n < 0L) ? 0 :
            (n > (long)Integer.MAX_VALUE) ? Integer.MAX_VALUE :
            (int)n);
}
```
最大返回 int 最大值，但是这个Map的长度是有可能超过int最大值的，所以JDK 8增了`mappingCount`方法。代码如下：
```java
public long mappingCount() {
    long n = sumCount();
    return (n < 0L) ? 0L : n; // ignore transient negative values
}
```
相比较 size 方法，mappingCount 方法的返回值是 long 类型。所以不必限制最大值必须是 Integer.MAX_VALUE。而 JDK 推荐使用这个方法。但这个返回值依然不一定绝对准确。
这两个方法都调用了`sumCount`方法，这才是核心：
```java
final long sumCount() {
    CounterCell[] as = counterCells; CounterCell a;
    long sum = baseCount;
    // 当 counterCells 不是 null，就遍历元素，并和 baseCount 累加
    if (as != null) {
        for (int i = 0; i < as.length; ++i) {
            if ((a = as[i]) != null)
                sum += a.value;
        }
    }
    return sum;
}

/**
 * Base counter value, used mainly when there is no contention,
 * but also as a fallback during table initialization
 * races. Updated via CAS.
 * 当没有争用时，使用这个变量计数
 */
private transient volatile long baseCount;

/**
 * Table of counter cells. When non-null, size is a power of 2.
 */
private transient volatile CounterCell[] counterCells;
```
这里比较重要的是两个变量`baseCount`和`counterCells`。前者是voliate变量，在`addCount`方法被使用，而这个方法在`putVal`方法之后被使用，定义如下

![addCount](https://ws4.sinaimg.cn/large/006tNbRwly1fuyhujjjn9j314c0z8gpe.jpg)
![fullAddCount](https://ws3.sinaimg.cn/large/006tNbRwly1fuyhux7zatj311y0p2mzq.jpg)

可以看到，`baseCount`和`counterCells`两重保障来最大程度保证size的精确。

## 其他
### public V putIfAbsent(K key, V value) {...}
该方法在`putVal`的时候约定不覆盖原值，由于`ConcurrentHashMap`的线程安全性，所以这个方法可以用来作为实现“往map中只put一次元素”作用。例如：

```java
/**
 * 写一段程序：多线程往map中put元素，要求每个元素只能被put一次
 * 思路： 利用concurrentHashMap中的putIfAbsent方法
 */
public class OnceOnlyMap {
    public static void main(String[] args) {
        String key = "key";
        String value1 = "value1";
        String value2 = "value2";
        ConcurrentHashMap<String, String> threadSafeMap = new ConcurrentHashMap<String, String>();

        new Thread(() -> {
            String ifAbsent = threadSafeMap.putIfAbsent(key, value1);
            if (Objects.equals(ifAbsent, null)) System.out.println("Successfully put the element into Map!");
            else System.out.println("The element is existing! The previous value is: " + ifAbsent);
        }).start();

        new Thread(() -> {
            String ifAbsent = threadSafeMap.putIfAbsent(key, value2);
            if (Objects.equals(ifAbsent, null)) System.out.println("Successfully put the element into Map!");
            else System.out.println("The element is existing! The previous value is: " + ifAbsent);
        }).start();
    }
}
```

> `HashMap`中`put()`和`resize()`都是尾插节点，而`ConcurrentHashMap`中`put()`是尾插节点，`transfer()`是头插节点。


参考
* [ConcurrentHashMap能完全替代HashTable吗？](https://my.oschina.net/hosee/blog/675423)
* [ConcurrentHashMap总结](http://www.importnew.com/22007.html)
* [JDK1.8逐字逐句带你理解ConcurrentHashMap(2)](https://blog.csdn.net/u012403290/article/details/68485855)
* [ConcurrentHashMap源码分析(1.8)](https://www.cnblogs.com/zerotomax/p/8687425.html)
* [并发编程 —— ConcurrentHashMap size 方法原理分析](https://www.cnblogs.com/stateis0/p/9062054.html)
