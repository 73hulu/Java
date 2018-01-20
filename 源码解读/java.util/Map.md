# Map<K,V>

该类提供了key到value的映射。**不能包含重复的key，每一个key只能被映射到一个value值**。该类代替了`Dictionary`，这是一个抽象类而非接口。

Map包含了3种视图：一个key的集合（通过`keySet()`方法），一个value的集合（通过`value()`方法）和一个key-value（通过`entrySet()`方法）的映射关系。Map的顺序是指用迭代器遍历的时候返回元素的顺序。许多Map接口的实现类，比如`TreeMap`类，定义了特殊的顺序，而有一些实现类，比如`HashMap`，就没有对顺序做特殊约定。

在用可变对象作为键值的时候要特别小心。

Note: great care must be exercised if mutable objects are used as map keys. The behavior of a map is not specified if the value of an object is changed in a manner that affects equals comparisons while the object is a key in the map. A special case of this prohibition is that it is not permissible for a map to contain itself as a key. While it is permissible for a map to contain itself as a value, extreme caution is advised: the equals and hashCode methods are no longer well defined on such a map.

所有通用的map的实现类一般都提供两种构造：一个无参构造方法来创建一个空映射集合；一个带有Map类型的单个参数的构造函数，该构造函数创建一个具有与其参数相同的键值映射的新映射。而实际上，后者的构造函数允许用户复制任何map，产生所需的等价map。没有办法强制执行这个建议（因为接口不能包含构造函数），但是JDK中的所有通用map的实现都符合。

很多map的实现类对它们可能包含的key和value都做了限制，比如，一些map的实现类禁止null作为key或者value的值；一些map实现类对key与value的类型有限制，尝试插入不合格的key或value将引发免检异常，通常为`NullPointerExeption`和`ClassCastException`，尝试查询不合格的key或value可能会引发异常，也可能会返回false。

注意，集合框架上的许多方法都是定义在`equals`方法的基础上的。

Map接口结构如下：

![Map](http://ovn0i3kdg.bkt.clouddn.com/Map.png)


### int size();
返回键值对的个数，如果个数超过`Integer.MAX_VALUE`，就只会返回`Integer.MAX_VALUE`。

### boolean isEmpty();
查看键值对是否为空。


### boolean containsKey(Object key);
当且仅当存在一个k使得`(key==null ? k==null : key.equals(k))`时候返回true。需要注意的是，这种k最多只有一个，因为map不允许存在重复的key。

### boolean containsValue(Object value);
当且仅当存在v使得`(value==null ? v==null : value.equals(v))`时候返回true。需要注意的是，这种v值可能有好几个。

### V get(Object key);
取得key对应的值，当map中含有一对键值k-v使得`(key==null ? k==null : key.equals(k))}`，此时返回v，否则返回null。（如果map允许null值，那么另外返回null这种设计是没有必要的）。

### V put(K key, V value);
往map中存放入一对键值。如果map中已经包含key作为键的键值对，那么将会覆盖，狗则会建增。

### V remove(Object key);
删除map中键值为key的键值对，如果存在该键值对，则删除并返回被删除的value值，如果不存在就返回null。

###  void putAll(Map<? extends K, ? extends V> m);
将另一个map中的集合都拷贝到当前map中，其效果相当于遍历一个map，对每个map元素都调用put方法。

### void clear();
移除map中的所有元素。

### Set&l;tK> keySet();
返回此映射中包含的key的Set视图。**对map的任何修改都会反映到该集合中，反之亦然**。如果在对集合进行迭代的过程中修改了映射（除了通过迭代器自己的删除操作），迭代的结果是未定义的。该Set集合支持元素删除，通过`Iterator.remove`，`Set.remove`、`Set.removeAll`、`Set.retainAll`和`Set.clear`操作从映射中删除相应的映射，它不支持`add`和`addAll`操作。

### Collection<V> values();
返回此映射中包含的value的集合视图。同样，**对map的任何修改都会反映到该集合中，反之亦然。**

### Set<Map.Entry<K, V>> entrySet();
返回此映射中包含的映射关系。**对map的任何修改都会反映到该集合中，反之亦然。**。注意到，其中Set的元素类型是`Map.Entry<K, V>`类型，这是Map定义的一个内部接口，指的是一个映射关系。定义如下：
```java
interface Entry<K,V> {
  //返回此映射关系对应的key值
   K getKey();

  //返回此映射关系对应的value值
   V getValue();

  //给此映射对应的value值重新复制
   V setValue(V value);

  //判断相等，当且仅当参数也为映射，且key与value完全相等
  boolean equals(Object o);

  //返回此映射关系的hashCode
  int hashCode();

  //还有1.8中加入的几个比较方法
  public static <K extends Comparable<? super K>, V> Comparator<Map.Entry<K,V>> comparingByKey() {
      return (Comparator<Map.Entry<K, V>> & Serializable)
          (c1, c2) -> c1.getKey().compareTo(c2.getKey());
  }
  public static <K, V extends Comparable<? super V>> Comparator<Map.Entry<K,V>> comparingByValue() {
      return (Comparator<Map.Entry<K, V>> & Serializable)
          (c1, c2) -> c1.getValue().compareTo(c2.getValue());
  }
  public static <K, V> Comparator<Map.Entry<K, V>> comparingByKey(Comparator<? super K> cmp) {
      Objects.requireNonNull(cmp);
      return (Comparator<Map.Entry<K, V>> & Serializable)
          (c1, c2) -> cmp.compare(c1.getKey(), c2.getKey());
  }
  public static <K, V> Comparator<Map.Entry<K, V>> comparingByValue(Comparator<? super V> cmp) {
      Objects.requireNonNull(cmp);
      return (Comparator<Map.Entry<K, V>> & Serializable)
          (c1, c2) -> cmp.compare(c1.getValue(), c2.getValue());
  }
}
```


### boolean equals(Object o);
整个映射集合的 `equals`方法，当且仅当两者完全相同时才返回true。

### int hashCode();
返回映射集合的哈希值，整个哈希值是`entrySet`哈希值的总和。

### default V getOrDefault(Object key, V defaultValue){...}
1.8新方法。当map中包含key的键值对的时候返回其value值，如果不包含，那么返回自定义的默认的`defaultValue`值。定义如下：
```java
default V getOrDefault(Object key, V defaultValue) {
    V v;
    return (((v = get(key)) != null) || containsKey(key))
        ? v
        : defaultValue;
}
```

后面还有几个JDK1.8新加入的方法，不常用在此不做讲解。

继承Map接口的类或者接口有：`HashMap`、`TreeMap`、`HashTable`、`SortedMap`、`Set`。
