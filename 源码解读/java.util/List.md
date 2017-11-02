# List
有序集合（也称为序列）。 该接口的用户可以精确控制列表中每个元素的插入位置。 用户可以通过整数索引（列表中的位置）访问元素，并搜索列表中的元素。与集合不同的是，序列可以**允许重复元素**，通常也**允许多个空元素**。

`List`接口结构如下：

![List](http://ovn0i3kdg.bkt.clouddn.com/List.png?imageView/2/w/400/q/90)

### public interface List<E> extends Collection<E>
接口声明，该接口继承了`Collection`接口。所以大部分的抽象方法是直接继承了`Collection`接口的方法。这里不再赘述，只列出其本身新声明的方法。


### E get(int index);
用来返回特定位置的元素。可能会出现越界异常。

###  E set(int index, E element);
将指定位置上的元素变更为指定的元素。

### void add(int index, E element);
添加一个元素到指定位置。

### E remove(int index);
删除指定位置上的元素。

### int indexOf(Object o);
返回元素第一次出现的位置（下标最小），如果不存在该元素就返回-1。

### int lastIndexOf(Object o);
返回元素最后一次出现的位置（下标最大），如果不存在该元素就返回-1。

### ListIterator<E> listIterator();
返回序列迭代器。

### ListIterator<E> listIterator(int index);
从列表中的指定位置开始，返回列表中的元素（按正确顺序）的列表迭代器。 指定的索引表示初次调用`next`返回的第一个元素。 对`previous`的初始调用将返回具有指定索引的元素减1。

### List<E> subList(int fromIndex, int toIndex);
返回此列表中指定的`fromIndex`（包括）和`toIndex`之间的独占视图。 （如果`fromIndex`和`toIndex`相等，返回的列表为空。）返回的列表由此列表支持，因此返回列表中的非结构性更改将反映在此列表中，反之亦然。 返回的列表支持此列表支持的所有可选列表操作。

这种方法消除了对显式范围操作（通常针对数组存在的排序）的需要。 任何期望列表的操作都可以通过传递一个子列表视图而不是整个列表来用作范围操作。 例如，以下语句从列表中删除了一系列元素：
```java
list.subList(from, to).clear();
```

可以为`indexOf`和`lastIndexOf`构造类似的成语，并且`Collections`类中的所有算法都可以应用于子列表。
如果支持列表（即，此列表）以除了通过返回的列表之外的任何方式进行结构修改，则此方法返回的列表的语义将变得未定义。 （结构修改是那些改变此列表的大小，或以其他方式扰乱它，使得正在进行的迭代可能产生不正确的结果）。
