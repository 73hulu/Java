# HashSet
`HashSet`是常用的`Set`接口的实现。此类实现了`Set`接口，由哈希表（实际上是`HashMap`实例）支持。 **对集合的迭代次序不作任何保证**; 特别是不能保证次序在一段时间内保持不变。 此类**允许null元素**。以下是该类的结构：

![HashSet](http://ovn0i3kdg.bkt.clouddn.com/HashSet.png)

### public class HashSet<E> extends AbstractSet<E> implements Set<E>, Cloneable, java.io.Serializable
类声明，可以看到该类继承了`AbstractSet`，实现了`Set`、`Cloneable`和`Serializable`接口。

### 构造函数
重载了5个构造函数。
#### public HashSet(){...}
无参构造函数。定义如下：
```java
public HashSet() {
    map = new HashMap<>();
}
```
其中的`map`是该类的私有成员变量，声明如下：
```java
private transient HashMap<E,Object> map;
```
注意到这是一个`HashMap`的实例，用关键词`transient`修饰，表明不是持久化。`HashSet`中的元素都是作为`map`中的key存放的。也可见，其实`HashSet`内部是由`HashMap`的实例来实现的。在这个无参构造函数中，未指定初始化大小和装载因子，根据`HashMap`的知识可知，初始化大小为16，装载因子是0.75。

#### public HashSet(Collection<? extends E> c){...}
，创建一个包含指定集合中的元素的`HashSet`实例。定义如下：
```java
public HashSet(Collection<? extends E> c) {
    map = new HashMap<>(Math.max((int) (c.size()/.75f) + 1, 16));
    addAll(c);
}
```
可以注意到的是初始化`HashMap`实例的时候，指定的装载因子是0.75，而初始化大小是指定的集合的4/3倍和16中的最大值。之后用`addAll`方法将集合中的元素加入到集合中。

####  public HashSet(int initialCapacity, float loadFactor){...}
直接指定了`HashMap`的初始化大小和装载因子的大小。定义略。

#### public HashSet(int initialCapacity){...}
只指定了初始化大小，装载因子默认为0.75。

#### HashSet(int initialCapacity, float loadFactor, boolean dummy){...}
注意该构造函数可访问修饰符是default，即包内访问，定义如下：
```java
HashSet(int initialCapacity, float loadFactor, boolean dummy) {
    map = new LinkedHashMap<>(initialCapacity, loadFactor);
}
```
注意到这个方法的第三个参数并没有什么用，只是为了能够重载一个构造函数（重载要保证方法名相同，参数表不同）而已。特别要注意的是，这个方法里的`map`的实现是`LinkedHashMap`。同样可以指定初始化大小和装载因子。

### public Iterator<E> iterator(){...}
返回迭代器，**不保证顺序**。
```java
public Iterator<E> iterator() {
    return map.keySet().iterator();
}
```
###  public int size(){...}
用来获取集合中包含的元素的个数。
```java
public int size() {
    return map.size();
}
```

### public boolean isEmpty(){...}
检测集合是否为空。
```java
public boolean isEmpty() {
    return map.isEmpty();
}
```

### public boolean contains(Object o){...}
检测集合是否含有某个元素，当且仅当至少存在一个元素e，使得`(o==null&&e==null&&o.equals(e))`时返回true。

### public boolean add(E e)
添加一个元素e。
```java
public boolean add(E e) {
    return map.put(e, PRESENT)==null;
}
```
其中`PRESENT`是该类定义的一个静态常量，声明如下：
```java
private static final Object PRESENT = new Object();
```
文档中对这个量的解释是：与支持map中的对象相关联的虚拟值。
暂时不懂是什么意思，等学习了`HashMap`之后再回过头来看。
前面说到`Set`不允许重复元素，所以当指定的对象已经在集合中存在，则不做任何改变返回false；如果原本不存在该对象，则添加元素并返回true。

### public boolean remove(Object o) {...}
从集合中移除某个元素。如果该集合中确实包含该元素则移除元素并返回true；如果不包含，则返回false。该方法被调用后，集合中不再含有该元素。定义如下：
```java
public boolean remove(Object o) {
    return map.remove(o)==PRESENT;
}
```


### public void clear(){...}
清空所有元素，指定该方法后，集合将不再含有元素。定义如下：
```java
public void clear() {
    map.clear();
}
```

### public Object clone() {...}
复制得到一个新的`HashSet`实例，该方法是**浅拷贝**！什么是“浅拷贝”？什么是“深拷贝”？

> 浅拷贝是指在拷贝对象时，对于基本数据类型的变量会重新复制一份，而对于引用类型的变量只是对引用进行拷贝。
> 深拷贝是指在拷贝对象时，同时会对引用指向的对象进行拷贝。
> 区别就在于是否对  对象中的引用变量所指向的对象进行拷贝。
> 参考[谈Java中的深拷贝和浅拷贝（转载）](http://www.cnblogs.com/dolphin0520/p/3700693.html)，“浅拷贝”和“深拷贝”的区别会在《集合框架》专题z具体讲到。

没有对引用指向的对象进行拷贝。
```java
@SuppressWarnings("unchecked")
public Object clone() {
    try {
        HashSet<E> newSet = (HashSet<E>) super.clone();
        newSet.map = (HashMap<E, Object>) map.clone();
        return newSet;
    } catch (CloneNotSupportedException e) {
        throw new InternalError(e);
    }
}
```

### public Spliterator<E> spliterator(){...}
这是JDK1.8新方法。`Spliterator`是一个并行迭代器。定义如下：
```java
public Spliterator<E> spliterator() {
    return new HashMap.KeySpliterator<E,Object>(map, 0, -1, 0, 0);
}
```
由于还没有学到`Spliterator`，所以这个方法的讲解先放一下。
