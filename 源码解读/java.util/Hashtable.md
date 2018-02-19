# Hashtable

`Hashtable`和`HashMap`可谓是焦不离孟，孟不离焦，但凡提起其中一个，一定会用来做对比，两者之间有哪些不同呢？被问的多了，答案也就记住了：默认初始化的大小不同，对null值的允许程度不同，线程安全性不同。所谓的“三大不同”，为什么会有这样的不同呢？还是从源码里找答案。

`Hashtable`的结构如下：

![Hashtable](http://ovn0i3kdg.bkt.clouddn.com/HashTable.png)

由于`HashMap`和`Hashtable`很大程度是相似，所以下面只挑选几处重要的不同的地方进行学习。

### public class Hashtable<K,V> extends Dictionary<K,V> implements Map<K,V>, Cloneable, java.io.Serializable {...}
类声明，`HashMap`继承自`AbstractMap`，而`Hashtable`继承自`Dictionary`，除此之外相同。

### 构造方法
同样重载了四种构造方法。
#### public Hashtable(int initialCapacity, float loadFactor) {...}
指定初始化容量和负载因子。定义如下：
```java
public Hashtable(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal Load: "+loadFactor);

    if (initialCapacity==0)
        initialCapacity = 1;
    this.loadFactor = loadFactor;
    table = new Entry<?,?>[initialCapacity];
    threshold = (int)Math.min(initialCapacity * loadFactor, MAX_ARRAY_SIZE + 1);
}

private float loadFactor;

private int threshold;

private transient Entry<?,?>[] table;

private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

```
显然`Hashtable`的设计没有`HashMap`来的细腻。其存储结构是"哈希桶数组 + 链表"的形式，没有红黑树这种骚操作。在对扩容阈值的估计上，简单采取了`initialCapacity * loadFactor`和`MAX_ARRAY_SIZE + 1`的最小值，没有2的倍数的约定，同样对于容量也没有2的倍数的约定。容量的最大值为`Integer.MAX_VALUE - 8`。

#### public Hashtable(int initialCapacity) {...}
只指定初始容量。定义如下：
```java
public Hashtable(int initialCapacity) {
    this(initialCapacity, 0.75f);
}
```
从这类可以看到，默认的负载因子为0.75，这一点和`HashMap`一致。

#### public Hashtable()
无参的默认构造方法，默认初始化容量为11，初始化负载因子为0.75。从这一定可以看出，`Hashtable`对容量大小并没有必须为2的倍数这种限定。
```java
public Hashtable() {
    this(11, 0.75f);
}
```

####   public Hashtable(Map<? extends K, ? extends V> t) {...}
用一个映射关系来创建另外一个映射关系。定义如下：
```java
public Hashtable(Map<? extends K, ? extends V> t) {
   this(Math.max(2*t.size(), 11), 0.75f);
   putAll(t);
}

public synchronized void putAll(Map<? extends K, ? extends V> t) {
    for (Map.Entry<? extends K, ? extends V> e : t.entrySet())
        put(e.getKey(), e.getValue());
}

public synchronized V put(K key, V value) {
    // Make sure the value is not null
    if (value == null) {
        throw new NullPointerException();
    }

    // Makes sure the key is not already in the hashtable.
    Entry<?,?> tab[] = table;
    int hash = key.hashCode();
    int index = (hash & 0x7FFFFFFF) % tab.length;
    @SuppressWarnings("unchecked")
    Entry<K,V> entry = (Entry<K,V>)tab[index];
    for(; entry != null ; entry = entry.next) {
        if ((entry.hash == hash) && entry.key.equals(key)) {
            V old = entry.value;
            entry.value = value;
            return old;
        }
    }

    addEntry(hash, key, value, index);
    return null;
}

private void addEntry(int hash, K key, V value, int index) {
    modCount++;

    Entry<?,?> tab[] = table;
    if (count >= threshold) {
        // Rehash the table if the threshold is exceeded
        rehash();

        tab = table;
        hash = key.hashCode();
        index = (hash & 0x7FFFFFFF) % tab.length;
    }

    // Creates the new entry.
    @SuppressWarnings("unchecked")
    Entry<K,V> e = (Entry<K,V>) tab[index];
    tab[index] = new Entry<>(hash, key, value, e);
    count++;
}

protected void rehash() {
  int oldCapacity = table.length;
  Entry<?,?>[] oldMap = table;

  // overflow-conscious code
  int newCapacity = (oldCapacity << 1) + 1;
  if (newCapacity - MAX_ARRAY_SIZE > 0) {
      if (oldCapacity == MAX_ARRAY_SIZE)
          // Keep running with MAX_ARRAY_SIZE buckets
          return;
      newCapacity = MAX_ARRAY_SIZE;
  }
  Entry<?,?>[] newMap = new Entry<?,?>[newCapacity];

  modCount++;
  threshold = (int)Math.min(newCapacity * loadFactor, MAX_ARRAY_SIZE + 1);
  table = newMap;

  for (int i = oldCapacity ; i-- > 0 ;) {
      for (Entry<K,V> old = (Entry<K,V>)oldMap[i] ; old != null ; ) {
          Entry<K,V> e = old;
          old = old.next;

          int index = (e.hash & 0x7FFFFFFF) % newCapacity;
          e.next = (Entry<K,V>)newMap[index];
          newMap[index] = e;
      }
  }
}

```
实现手段非常简单粗暴了，其大小简单粗暴取为该映射的大小的2倍与11的最大值。注意到，之后调用的`putAll`、`put`、`addEntry`方法都是同步方法，`Hashtable`中其他方法也都是同步方法，这也就是为什么`hashtable`为什么是线程安全的原因。注意`put`方法，当value为null的时候会抛出异常，这就确保了`Hashtable`不能存储值为null的映射。在`addEntry`方法中，如果key为null，那么`hash = key.hashCode();`这句话直接会抛出空指针异常，这就确保了`Hashtable`不能存储key为null的映射。后面的方法都比较简单。注意到，`count`表示实际存储的映射关系的个数，等同于`HashMap`中的`size`，当实际个数超过阈值的时候我们需要进行重新散列，方法就是扩容后再散列，新容量等于2倍的旧容量再加上1（`HashMap`的扩容规则是2倍）。后面的过程也比较简单，这里就不详细说了。



### keys方法和elements方法
这两个方法都覆盖其父类`Dictionary`中的方法，定义如下:
```java
public synchronized Enumeration<K> keys() {
    return this.<K>getEnumeration(KEYS);
}
public synchronized Enumeration<V> elements() {
    return this.<V>getEnumeration(VALUES);
}
```
注意到，这个方法的可见性是protected，即包内可见。所以我们在使用的时候都不怎么用到。

### contains方法
`Hashtable`有三种contains方法，实际上只有两种用处，就是判断指定的key或者value是否存在。定义如下：
```java
public synchronized boolean contains(Object value) {
   if (value == null) {
       throw new NullPointerException();
   }

   Entry<?,?> tab[] = table;
   for (int i = tab.length ; i-- > 0 ;) {
       for (Entry<?,?> e = tab[i] ; e != null ; e = e.next) {
           if (e.value.equals(value)) {
               return true;
           }
       }
   }
   return false;
}
public boolean containsValue(Object value) {
    return contains(value);
}

public synchronized boolean containsKey(Object key) {
    Entry<?,?> tab[] = table;
    int hash = key.hashCode();
    int index = (hash & 0x7FFFFFFF) % tab.length;
    for (Entry<?,?> e = tab[index] ; e != null ; e = e.next) {
        if ((e.hash == hash) && e.key.equals(key)) {
            return true;
        }
    }
    return false;
}
```

> `Hashtable`同样有一些与`HashMap`类似的方法，比如remove、keyset、values、entrySet等，在此不做赘述。


总结一下`HashMap`和`Hashtable`的差异之处：
1. `HashMap`继承自`AbstractMap`，`Hashtable`继承自`Dictionary`，这也是造成两者不同的主要原因。
2. `HashMap`默认初始容量为16，负载因子为0.75，约定容量和阈值必须为2的倍数，即一定为合数，扩容策略为2倍；`Hashtable`默认初始容量为11，负载因子为0.75，没有2的倍数的硬性约定，扩容策略为2倍加上1。（Hashtable的容量设计里面来自于“素数导致冲突的概率小于合数”，参考http://blog.csdn.net/liuqiyao_01/article/details/14475159）
3. `HashMap`允许一个key为null的映射和多个value为null的映射；`Hashtable`不允许key为null或value为null的映射，否则会抛出空指针异常。
4. `HashMap`是非线程安全的，如果要建立线程安全的HashMap，则需要借助`Collections`；`Hashtable`是线程安全的。
5. `HashMap`发生冲突的使用链地址法，`Hashtable`使用再散列。
6. `HashMap`借助哈希桶 + 链表/红黑树的设计（JDK1.8）,`Hashtable`借助哈希桶 + 链表的设计方法。
