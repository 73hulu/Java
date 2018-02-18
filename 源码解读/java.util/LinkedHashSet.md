# LinkedHashSet

在学习`HashSet`的时候，遇到一个三个参数的构造方法，该方法中，用来存储元素的hashMap被定义为`LinkedHashMap`的实例，所以创建出来的`HashSet`就是`LinkedHashSet`。

该类的结构如下：

![LinkedHashSet](http://ovn0i3kdg.bkt.clouddn.com/LinkedHashSet.png)


可以看到，结构非常简单，只是定义了几个构造方法。

### public class LinkedHashSet<E> extends HashSet<E> implements Set<E>, Cloneable, java.io.Serializable
类声明。可以看该类继承了`HashSet`接口。


### 构造方法
重载了5个构造方法。调用的都是`HashSet`中三个参数的构造方法。默认初始化容量为16，装载因子为0.75。
```java
public LinkedHashSet() {
    super(16, .75f, true);
}
```
需要注意的是以集合为参数的构造方法：
```java
public LinkedHashSet(Collection<? extends E> c) {
     super(Math.max(2*c.size(), 11), .75f, true);
     addAll(c);
 }
```
初始化大小在2倍的集合容量与11中选择较大者。
