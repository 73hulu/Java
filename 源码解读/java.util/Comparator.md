# Comparator
Java8 中提供的函数式接口比较器，其中的抽象方法为`int compare(T t, T t)`和`boolean equals(Object object)`。
结构如下：
![Comparator](https://ws1.sinaimg.cn/large/006tKfTcgy1fs4yei75i2j30w20ka0xu.jpg)

## int compare(T o1, T o2);
比较两个参数，并返回正数、0和负数，分别代表大于，等于和小于。在Java8之间，经典的比较器是这么写的。

```Java
Comparator<Developer> byName = new Comparator<Developer>(){
  @Override
  int compare(Developer o1, Developer o2){
    return o1.getName().compareTo(o2.getName());
  }
}
```
1.8之后，上面的过程就可以精简很多：
```java
Comparator<Developer> comparator = (Developer o1, Developer o2) -> o1.getName().compareTo(o2.getName());
```

##  boolean equals(Object obj);
用来判断对象是否相等。

## 正序逆序自然顺序
提供了几个静态方法来返回正序、逆序和自然序，其本质还是调用`Collectors`中的正序、逆序和自然顺序。

```Java
// 非静态方法，逆序
default Comparator<T> reversed() {
    return Collections.reverseOrder(this);
}

// 静态方法， 逆序
public static <T extends Comparable<? super T>> Comparator<T> reverseOrder() {
    return Collections.reverseOrder();
}

// 静态方法，自然序
public static <T extends Comparable<? super T>> Comparator<T> naturalOrder() {
    return (Comparator<T>) Comparators.NaturalOrderComparator.INSTANCE;
}
```

## null 值处理
提供了null值在排序中的特殊处理方法
```Java
// null 值最后会在排序的最前面
public static <T> Comparator<T> nullsFirst(Comparator<? super T> comparator) {
    return new Comparators.NullComparator<>(true, comparator);
}

// null 值最后会在排序的最后面
public static <T> Comparator<T> nullsLast(Comparator<? super T> comparator) {
    return new Comparators.NullComparator<>(false, comparator);
}
```

## 比较器复合
`thenComparing`和`comparingXXX`方法提供了链式操作的契机。
