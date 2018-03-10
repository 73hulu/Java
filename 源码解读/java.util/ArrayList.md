# ArrayList

经过前面众多的铺垫，终于到了一个真正的实现类。`ArrayList`被经常用到，了解真正的底层实现才能真正了解集合框架。该类的结构图如下：

![ArrayList](http://ovn0i3kdg.bkt.clouddn.com/ArrayList_1.png?imageView/2/w/400/q/90)
![ArrayList](http://ovn0i3kdg.bkt.clouddn.com/ArrayList_2.png?imageView/2/w/400/q/90)


ArrayList是List接口的可调整大小的实现。可以存储所有的值，包括null。这个类大致等于`Vector`，除了它是不同步的。

## public class ArrayList<E> extends AbstractList<E> implements List<E>, RandomAccess, Cloneable, java.io.Serializable
类声明。直接继承`AbstractList`抽象类，实现List接口、RandomAccess接口、Cloneable接口、Serializable接口。
其中`RandomAccess`接口中没有任何方法，即这是一个标记接口。用来表明其支持快速（通常是固定时间）随机访问。此接口的主要目的是允许一般的算法更改其行为，从而在其应用到随机或连续访问列表时提供更好的性能。

JDK中推荐的是对List集合尽量要实现`RandomAccess`接口。如果集合类是`RandomAccess`的实现，则尽量用`for(int i = 0; i < size; ++)`来代替`Iterator`迭代器来遍历，后者的效果差一点，反过来，如果List是`Sequence List`，则最好用迭代器来进行迭代。

JDK中说的很清楚，在对List特别是Huge size的List的遍历算法中，要尽量来判断是属于RandomAccess(如ArrayList)还是Sequence List (如LinkedList），因为适合RandomAccess List的遍历算法，用在Sequence List上就差别很大，常用的作法就是：
要作一个判断：
```java
if (list instance of RandomAccess) {
    for(int m = 0; m < list.size(); m++){}
}else{
    Iterator iter = list.iterator();
    while(iter.hasNext()){}
}
```
这一点我们在遇到遍历时候再进行学习。

另外注意到，`ArrayList`使用了泛型。


## 构造方法
`ArrayList`重载了3个构造方法。
### public ArrayList(int initialCapacity){...}
指定缓存区初始大小。
```java
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    }
}
```
其中`elementData`是ArrayList中真正用来存储数据的对象数组。定义如下：
```java
transient Object[] elementData
```
前面说到ArrayList使用了泛型，所以这里元素的缓存就使用了顶级父类`Object`来声明，那么为什么不使用泛型数组`E[]` 来声明呢？因为实际上并不存在泛型数组这个东西，而且这里使用泛型将会在后面引起很多的麻烦。具体可以参考“泛型”的讲解内容。

**该缓存数组用`transient`修饰。**

在这个构造方法中可以看到，如果指定的参数合法那么初始的缓存区大小就可被指定，否则就被赋值未`EMPTY_ELEMENTDATA`，其定义如下：
```java
private static final Object[] EMPTY_ELEMENTDATA = {};
```
注意到，这是一个静态常量空数组，用于空实例的共享空数组。即数组缓存被初始化为空数组。

###  public ArrayList() {}
无参构造方法。定义如下：
```java
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}
```
无指定的情况下，对象数组`elementData`被赋值为`DEFAULTCAPACITY_EMPTY_ELEMENTDATA`，这个变量定义如下：
```java
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
```
诶，这个`DEFAULTCAPACITY_EMPTY_ELEMENTDATA`空对象数组和之前提到的`EMPTY_ELEMENTDATA`定义完全一样，用法看起来似乎差不多，有什么用处呢？根据指示，大概意思是构造一个空的对象数组，用来与`EMPTY_ELEMENTDATA`这个数组进行对比，来确定当第一次向`ArrayList`中添加数据时，应该如何进行扩容。暂时不太好理解，往后看遇到了再说。

另外这个方法的注释说“默认构造了一个容量为10的缓存区”，怎么会是10呢，明明`DEFAULTCAPACITY_EMPTY_ELEMENTDATA`是一个空数组。往后看就知道了。

### public ArrayList(Collection<? extends E> c){...}
指定一个集合，构造一个包含集合中元素的`ArrayList`实例。定义如下：
```java
public ArrayList(Collection<? extends E> c) {
    elementData = c.toArray();
    if ((size = elementData.length) != 0) {
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    } else {
        // replace with empty array.
        this.elementData = EMPTY_ELEMENTDATA;
    }
}
```
这里首先用到了`Collection.toArray()`方法进行数组转换，将自己的缓存指向了这个转化得到数组。这里有一个确认对象类型的检验，如果集合中的对象不是Object类型，利用`Arrays.copyOf`方法进行数组的拷贝。这里有一个问题，为什么“集合中的数组不是Object类型”？注释给出的解释是:`c.toArray`方法有可能导致的问题，这个bugID是6260652。当转化得到的数组为空时，ArrayList并没有使用这个数组，而是仍然能使用自己构造的空数组。

## public int size(){...}
用来获取`ArrayList`中真正元素的个数。定义如下：
```java
public int size() {
    return size;
}
```
`size`就是一个私有变量，默认为0，之后往里添加元素的时候就增加，删除元素的时候就减小。注意和缓存空间大小的区别！！！

## public boolean isEmpty(){...}
查看是否存在元素。怎么判断，很简单，还是看size的大小，为0就是无元素，否则就是有元素。定义略。




## add方法
往`ArrayList`中添加元素，这是最常使用的方法。重载了两种add方法。
### public boolean add(E e){...}
只指定了元素，那么就是**追加**元素。定义如下：
```java
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}
```
首先需要保证能够添加得进去，什么意思？就是保证容量足够，有位置添加。`ensureCapacityInternal`的定义如下：
```java
private void ensureCapacityInternal(int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
    }

    ensureExplicitCapacity(minCapacity);
}
```
这个方法的参数是最低需要保证的容量，在这个add方法中，最低容量就是在当前size基础上加1。从上面的方法可以看到，首先将缓冲区和`DEFAULTCAPACITY_EMPTY_ELEMENTDATA`进行比较，如果相等说明什么？说明这个实例创建的时候是用默认构造方法创建的而且还没有添加过元素。这不行啊，我得有空间才行，行，那我创建一个，多大空间合适啊？这里取常量`DEFAULT_CAPACITY`和指定参数`minCapacity`的较大者，其中常量`DEFAULT_CAPACITY`等于10。这里较大者一定是10(因为if判断成立的时候，说明原来是空数组，size等于0，那么指定的minCapacity是1，10和1比较当然取10，这也就是为什么前面说到，默认构造方法相当于创建了一个初始容量为10的数组缓存区)。之后调用的`ensureExplicitCapacity`，定义如下：
```java
private void ensureExplicitCapacity(int minCapacity){
  modCount++;
  // overflow-conscious code
  if (minCapacity - elementData.length > 0)
      grow(minCapacity);
}
```
`modCount`是该类的一个保护变量，定义如下：
```java
protected transient int modCount = 0;
```
我们暂且先不管它的具体含义。看后面的代码，很简单，判断容量和现在的容量相比那么大，如果现在不够大，那就增加容量。`grow`方法定义如下：
```java
private void grow(int minCapacity) {
   // overflow-conscious code
   int oldCapacity = elementData.length;
   int newCapacity = oldCapacity + (oldCapacity >> 1);
   if (newCapacity - minCapacity < 0)
       newCapacity = minCapacity;
   if (newCapacity - MAX_ARRAY_SIZE > 0)
       newCapacity = hugeCapacity(minCapacity);
   // minCapacity is usually close to size, so this is a win:
   elementData = Arrays.copyOf(elementData, newCapacity);
}

private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE :
        MAX_ARRAY_SIZE;
}
```
这段代码进行了扩容，新容量和旧容量的关系是`新容量= 1.5旧容量`。但是这并不是最终的容量，需要检验。有两种可能：
1. 得到的容量比需要的容量要少，为什么会这样？溢出了！
2. 得到的容量超过了规定的最大容量，进入`hugeCapacity`中，如果需求的容量小于0，则抛出内存溢出异常，如如果需要的容量比规定的最大容量大，那么最大容量只能是Integer.MAX_VALUE了。

最后elementData通过`Arrays`的赋值拷贝方法进行扩容。

扩容后再回到add方法中，将size位置赋值为指定值，size增加1，返回true。【注意这个方法永远都返回true，所以我们不能通过返回值来判断是否添加成功】。

**从ArrayList的扩容过程可以看出，ArrayList并不是无限大的，它指定了一个最大容量是Integer.MAX_VALUE - 8，而实际的最大只能是Integer.MAX_VALUE这个值了。**

### public void add(int index, E element){...}
指定位置和指定元素。定义如下：
```java
public void add(int index, E element) {
    rangeCheckForAdd(index);

    ensureCapacityInternal(size + 1);  // Increments modCount!!
    System.arraycopy(elementData, index, elementData, index + 1,
                     size - index);
    elementData[index] = element;
    size++;
}

private void rangeCheckForAdd(int index) {
    if (index > size || index < 0)
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}
```
首先检查了位置参数index是否合法。接着同样进行是否扩容的判断。接着用调用`System.arraycopy`方法将index上及之后的元素后移移位，这是一个本地方法，看不到源码。最后给index上的元素赋值，size增加1。注意这个方法的返回类型是`void`而不是`boolean`。


## remove方法
从线性表中删除元素。重载了两个方法。
### public E remove(int index) {...}
指定元素删除的位置，返回的是被删除的元素。定义如下：
```java
public E remove(int index) {
    rangeCheck(index);

    modCount++;
    E oldValue = elementData(index);

    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    elementData[--size] = null; // clear to let GC do its work

    return oldValue;
}

private void rangeCheck(int index) {
    if (index >= size)
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}

@SuppressWarnings("unchecked")
E elementData(int index) {
    return (E) elementData[index];
}
```
首先检查参数index是否越界，这类的越界检查和add方法中的越界检查不一样，`rangeCheck`没有对`index < 0 `的情况作出处理。所以如果参数是-1，那么在`elementData`方法中会抛出`ArrayIndexOutOfBoundsException`异常，注意到这个方法上确实用`@SuppressWarnings`注解做了标记，可能会抛出免检异常。诶，既然做了越界检查，为什么不索性连负数的越界一起检测了？？？不太懂这里的设计思路。

之后`numMoved`记录的是需要移动元素的位数，如果等于0，说明要删除的的是最后一位，就没有必要进行移动了。最后将移动后的数组末尾赋值为null。注释里说这是释放内存，但是我们不能依赖这种方法，因为我们永远你不知道GC如何回收它们。

> 读到这里，我有一个启发，就是数组的移动操作可以用`System.arraycopy()`方法代替我们自己写的遍历赋值的移动方法，虽然前者的底层实现原理就是遍历赋值，但是由于前者是native方法，效率更高。这点在`String`类的设计中可以看到实际的应用。

### public boolean remove(Object o){...}
移除`ArrayList`中**第一个**指定的对象o。如果删除成功就返回true，否则返回false。定义如下：
```java
public boolean remove(Object o) {
   if (o == null) {
       for (int index = 0; index < size; index++)
           if (elementData[index] == null) {
               fastRemove(index);
               return true;
           }
   } else {
       for (int index = 0; index < size; index++)
           if (o.equals(elementData[index])) {
               fastRemove(index);
               return true;
           }
   }
   return false;
}

private void fastRemove(int index) {
   modCount++;
   int numMoved = size - index - 1;
   if (numMoved > 0)
       System.arraycopy(elementData, index+1, elementData, index,
                        numMoved);
   elementData[--size] = null; // clear to let GC do its work
}
```
这里可以看到，首先要判断指定的对象是不是null，为什么要区别？因为`o.equals(elementData[index]`可能会引起空指针异常。除此之外操作过程都相同，主要调用的是`fastRemove`方法，这个方法实际上就是`remove(int index)`的后半段代码，那句很奇怪了，为什么`remove(int index)`不直接调用这个方法呢？


## public void clear() {...}
清除元素。该方法被调用后，`ArrayList`缓冲区中不再有元素，定义如下：
```java
public void clear() {
    modCount++;

    // clear to let GC do its work
    for (int i = 0; i < size; i++)
        elementData[i] = null;

    size = 0;
}
```
将所有元素都置为null，再将size置为0。

## addAll方法
`add`方法是向线性表中添加单个元素，而`addAll`方法是想向线性表添加一个集合中的所有元素，同样重载了两个方法。
### public boolean addAll(Collection<? extends E> c){...}
向线性表追加元素。
```java
public boolean addAll(Collection<? extends E> c) {
    Object[] a = c.toArray();
    int numNew = a.length;
    ensureCapacityInternal(size + numNew);  // Increments modCount
    System.arraycopy(a, 0, elementData, size, numNew);
    size += numNew;
    return numNew != 0;
}
```
追加的很简单，将集合转为数组、扩容、移动赋值、更新size，最后返回是否添加成功的boolean变量，实际上一系列过程都是正确执行的，所以该方法是不是返回true取决于指定的这个集合是不是空集合，是空集合的话那还添加个啥，返回false，否则返回true(因为这种情况一定添加成功)。

### public boolean addAll(int index, Collection<? extends E> c){...}
指定元素集合和添加的位置。定义如下：
```java
public boolean addAll(int index, Collection<? extends E> c) {
    rangeCheckForAdd(index);

    Object[] a = c.toArray();
    int numNew = a.length;
    ensureCapacityInternal(size + numNew);  // Increments modCount

    int numMoved = size - index;
    if (numMoved > 0)
        System.arraycopy(elementData, index, elementData, index + numNew,
                         numMoved);

    System.arraycopy(a, 0, elementData, index, numNew);
    size += numNew;
    return numNew != 0;
}
```
套路可以延续`add(int index, E o)`的过程：越界检查、集合转为数组、扩容、移动赋值，更新size，不同的地方在于，这里需要两次数组的移动赋值：一次是将原缓冲区中index后的元素都往后移动numNew个位置，一次是将新元素添加进缓冲区。

另外还需要注意的是，这个方法返回的类型是boolean，而添加单个元素`add(int index, E o)`方法的返回值是void。

## public boolean removeAll(Collection<?> c) {...}
这是JDK1.7的新方法。`removeAll`就没有重载方法了，就一个移除指定集合中的所有元素。定义如下：
```java
public boolean removeAll(Collection<?> c) {
    Objects.requireNonNull(c);
    return batchRemove(c, false);
}
public static <T> T requireNonNull(T obj) {
    if (obj == null)
        throw new NullPointerException();
    return obj;
}

private boolean batchRemove(Collection<?> c, boolean complement) {
    final Object[] elementData = this.elementData;
    int r = 0, w = 0;
    boolean modified = false;
    try {
        for (; r < size; r++)
            if (c.contains(elementData[r]) == complement)
                elementData[w++] = elementData[r];
    } finally {
        // Preserve behavioral compatibility with AbstractCollection,
        // even if c.contains() throws.
        if (r != size) {
            System.arraycopy(elementData, r,
                             elementData, w,
                             size - r);
            w += size - r;
        }
        if (w != size) {
            // clear to let GC do its work
            for (int i = w; i < size; i++)
                elementData[i] = null;
            modCount += size - w;
            size = w;
            modified = true;
        }
    }
    return modified;
}
```
首先调用的是工具类`Objects`【注意不是`Object`】的静态方法`requireNonNull`，检验c非空。如果为空则抛出空指针异常。接下来就是`batchRemove`方法，这个方法指定的第二个参数是false。意思就是留下与`c `中不一样元素，这样就达到了删除c中所有元素的目的。

在`batchRemove`方法中，首先对数组进行了一次赋值，然后将数组中的元素一次与collection中的元素进行比较。try块的结果是一个数组，数组前w位元素或与colletcion相同（当complement = true的时）或不同（complement = false）。

在finally中，第二个if代码块的作用是将数组中w之后的元素全部变成null，让gc回收。而第一个if块中，看条件`r != size`似乎永远不会满足，因为try块中r一直递增，什么时候会出现`r != size`呢？当`c.contains(elementData[r])`抛出异常的时候，一旦类型不匹配（之前我们说过，toArray可能会导致类型不一致），就会抛出异常。这种情况下，就把剩下没有比较的原数组部分，即位置r之后的部分复制都w位置之后。再回到第二个if块，进行到这一步，可以明确，原缓冲区一定是remove了一些元素了，所以将布尔变量modified置为true，表示修改成功。

> 这个的方法很巧妙，要眼熟它。

## public boolean retainAll(Collection<?> c){...}
`retain`的意思是“保留”，所以`retainAll`方法用于从列表中移除未包含在collection中的所有元素。如果list集合对象由于调用该方法而发生改变，则返回true。

> 原来我很纳闷，“只保留集合中的所有元素”，那最后不就是得到一个与collection相同的集合呢，为什么不知道让elementData = c，转念一想，太傻了我，根本不能赋值啊，首先父类不能赋值给子类，再者就算是转化为list类型，顺序也不一致了，所以这个想法太蠢了。

方法定义如下：
```java
public boolean retainAll(Collection<?> c) {
    Objects.requireNonNull(c);
    return batchRemove(c, true);
}
```
仍旧调用的`batchRemove`方法，但是第二个参数是true，表示留下的而是存在集合c中的那些元素。

## public void sort(Comparator<? super E> c){...}
使用规则c对线性表进行排序，调用的是`Arrays.sort`方法。定义如下：
```java
@Override
@SuppressWarnings("unchecked")
public void sort(Comparator<? super E> c) {
    final int expectedModCount = modCount;
    Arrays.sort((E[]) elementData, 0, size, c);
    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();
    }
    modCount++;
}
```

## public int indexOf(Object o){...}
从头开始，查找指定元素的第一个索引，如果没有该元素就返回-1。
```java
public int indexOf(Object o) {
    if (o == null) {
        for (int i = 0; i < size; i++)
            if (elementData[i]==null)
                return i;
    } else {
        for (int i = 0; i < size; i++)
            if (o.equals(elementData[i]))
                return i;
    }
    return -1;
}
```

## public int lastIndexOf(Object o){...}
从尾开始，查找指定元素的第一个索引，如果没有该元素就返回-1。
```java
public int lastIndexOf(Object o) {
    if (o == null) {
        for (int i = size-1; i >= 0; i--)
            if (elementData[i]==null)
                return i;
    } else {
        for (int i = size-1; i >= 0; i--)
            if (o.equals(elementData[i]))
                return i;
    }
    return -1;
}
```
## public Object clone(){...}
重写了Object类的拷贝方法，注意`ArrayList`的拷贝方法是**浅拷贝**。
```java
public Object clone() {
    try {
        ArrayList<?> v = (ArrayList<?>) super.clone();
        v.elementData = Arrays.copyOf(elementData, size);
        v.modCount = 0;
        return v;
    } catch (CloneNotSupportedException e) {
        // this shouldn't happen, since we are Cloneable
        throw new InternalError(e);
    }
}
```
> 浅拷贝是说对于对象的拷贝，实际上只拷贝了对象的地址，源对象和经过拷贝得到的对象指向的是同一个对象。而深拷贝是指经过new 对象得到的拷贝结果。
>参考 http://www.jb51.net/article/48201.htm

###  public Object[] toArray(){...}
得到对象数组。注意可不能直接返回`ArrayList`中的缓存，得返回其副本。
```java
public Object[] toArray() {
    return Arrays.copyOf(elementData, size);
}
```

注意该方法的返回值是 `Object`类型的数组，我们无法用强制类型转化的方法得到想要的对象数组。比如某个`list`存储`String`类型，使用语句`(String[]) list.toArray()`会报错，该语句不正确，正确的写法是`list.toArray(new String[list.size()])`。

## public E get(int index){...}
取得某个位置上的元素值。
```java
public E get(int index) {
    rangeCheck(index);

    return elementData(index);
}
```

## public E set(int index, E element){...}
更改某个位置上的元素的值。
```java
public E set(int index, E element) {
    rangeCheck(index);

    E oldValue = elementData(index);
    elementData[index] = element;
    return oldValue;
}
```
> `ArrayList`中还有一些其他方法，大部分是一些1.8的新特性，不是很常用的，这里先暂时不做学习。

## 总结
`ArrayList`是最基础也是最常用的Java集合框架类，通过读源码加深了对之前只能从书本上获取到的信息，比如何如初始化，如何扩容，这些都能从源码中找到最清晰原始的答案。另外还有一些编程方法指值得学习，比如成块的数组元素的移动可以借助`System.arraycopy`方法，在删除多个元素的时候，使用赋值的方法可以在O(n)的时内完成。这些问题的解决办法都值得学习。另外注意到`ArrayList`是线程不安全的，它的操作中没有使用同步方法，而`Vector`类是线程安全的，实际上，两者的区别只在于线程安全性上，除此之外，`ArrayList`完全可以替代`Vector`。另外，`Collections.synchronizedList`可以用来创建线程安全的`list`。






## 迭代器
`ArrayList`通过内部类`Itr`实现了 `Iterator`接口，注意其类本身没有实现`Iterator`接口。
这个内部类的定义如下：
```java
private class Itr implements Iterator<E> {
   int cursor;       // index of next element to return
   int lastRet = -1; // index of last element returned; -1 if no such
   int expectedModCount = modCount;

   public boolean hasNext() {
       return cursor != size;
   }

   @SuppressWarnings("unchecked")
   public E next() {
       checkForComodification();
       int i = cursor;
       if (i >= size)
           throw new NoSuchElementException();
       Object[] elementData = ArrayList.this.elementData;
       if (i >= elementData.length)
           throw new ConcurrentModificationException();
       cursor = i + 1;
       return (E) elementData[lastRet = i];
   }

   public void remove() {
       if (lastRet < 0)
           throw new IllegalStateException();
       checkForComodification();

       try {
           ArrayList.this.remove(lastRet);
           cursor = lastRet;
           lastRet = -1;
           expectedModCount = modCount;
       } catch (IndexOutOfBoundsException ex) {
           throw new ConcurrentModificationException();
       }
   }

   @Override
   @SuppressWarnings("unchecked")
   public void forEachRemaining(Consumer<? super E> consumer) {
       Objects.requireNonNull(consumer);
       final int size = ArrayList.this.size;
       int i = cursor;
       if (i >= size) {
           return;
       }
       final Object[] elementData = ArrayList.this.elementData;
       if (i >= elementData.length) {
           throw new ConcurrentModificationException();
       }
       while (i != size && modCount == expectedModCount) {
           consumer.accept((E) elementData[i++]);
       }
       // update once at end of iteration to reduce heap write traffic
       cursor = i;
       lastRet = i - 1;
       checkForComodification();
   }

   final void checkForComodification() {
       if (modCount != expectedModCount)
           throw new ConcurrentModificationException();
   }
}
```
首先这类有三个重要的属性：
1. `cursor`： 游标，迭代器始终是往前走的，`cursor`使用是++。
2. `lastRet`：末尾标识，标识了最后一个返回的元素的索引位置。-1表示这个元素不存在。
3. `int expectedModCount = modCount;`：操作数标识。**用来校验在使用iteration期间，是否存在非iteration的操作对ArrayList进行了修改**。modCount这个变量在之前`add`、`addAll`、`remove`、`removeAll`、`retainAll`的操作中指定++操作。

接着这个类有三个重要的方法：
1. `checkForComodification`：在这个内部类中，几乎所有的都在调用它，目的在于用来校验在使用iteration期间，是否存在非iteration的操作对ArrayList进行了修改。Iterator的游标特性决定了它对ArrayList中元素在这一时刻的位置很敏感，如果当前游标在index位置，而有其他操作在index-1的位置上插入了一个元素，那么调用iterator的next()方法，返回的还是当前这个元素，这样就乱了。为了避免这个情况发生，需要在这个期间把ArrayList“锁住“。它并没有实现真正的锁，所以采用了这个校验的方式。
2. `next`：返回游标位置的元素。注意这里面第一个if，游标移动到一个不存在数据的地方会抛出异常，而并不是返回null，这就是我们为什么在使用`iterator`的时候，不用用`(null == iterator.next())`来判断的原因，正确的做法是在每次循环开始的时候判断`iterator.hasNext()`。
3. `remove`：删除`lastRet`所标识位置的元素。可以理解为删除当前元素。在try之前有一个校验，保证元素没有被改动过。在try块中，首先删除了`lastRet`标识的元素，然后让游标指向了这个位置。我们知道删除了元素之后，这个位置有了新元素。**这样再次调动next()的时候就不会出现空指针异常，更不会跳过一个元素**。最后`expectedModCount = modCount`，这句话相当于释放了锁。

对于 `ArrayList`，它还定义了一种内部迭代器`ListItr`，这是一个功能更强大的迭代器，该内部类的定义如下：
```java
private class ListItr extends Itr implements ListIterator<E> {
    ListItr(int index) {
        super();
        cursor = index;
    }

    public boolean hasPrevious() {
        return cursor != 0;
    }

    public int nextIndex() {
        return cursor;
    }

    public int previousIndex() {
        return cursor - 1;
    }

    @SuppressWarnings("unchecked")
    public E previous() {
        checkForComodification();
        int i = cursor - 1;
        if (i < 0)
            throw new NoSuchElementException();
        Object[] elementData = ArrayList.this.elementData;
        if (i >= elementData.length)
            throw new ConcurrentModificationException();
        cursor = i;
        return (E) elementData[lastRet = i];
    }

    public void set(E e) {
        if (lastRet < 0)
            throw new IllegalStateException();
        checkForComodification();

        try {
            ArrayList.this.set(lastRet, e);
        } catch (IndexOutOfBoundsException ex) {
            throw new ConcurrentModificationException();
        }
    }

    public void add(E e) {
        checkForComodification();

        try {
            int i = cursor;
            ArrayList.this.add(i, e);
            cursor = i + 1;
            lastRet = -1;
            expectedModCount = modCount;
        } catch (IndexOutOfBoundsException ex) {
            throw new ConcurrentModificationException();
        }
    }
}
```
可以看到，这个迭代器继承自`Itr`，可以双向迭代，还能进行`add`和`set`操作【实际上其本质还是调用的外部类`ArrayList`的`add`和`set`方法】。

`ArrayList`类提供了对迭代器的获取，一共有三个方法：
#### public Iterator<E> iterator(){...}
获取一个基本的`Itr`类型的迭代器。定义如下：
```java
public Iterator<E> iterator() {
    return new Itr();
}
```
###  public ListIterator<E> listIterator(){...}
获取一个`ListItr`类型的迭代器，默认游标的位置是0。定义如下：
```java
public ListIterator<E> listIterator() {
   return new ListItr(0);
}
```
### public ListIterator<E> listIterator(int index){...}
获取一个`ListItr`类型的迭代器，指定游标的初始位置。
```java
public ListIterator<E> listIterator(int index) {
    if (index < 0 || index > size)
        throw new IndexOutOfBoundsException("Index: "+index);
    return new ListItr(index);
}
```

### public List<E> subList(int fromIndex, int toIndex){...}
获取子集合。定义如下：
```java
public List<E> subList(int fromIndex, int toIndex) {
    subListRangeCheck(fromIndex, toIndex, size);
    return new SubList(this, 0, fromIndex, toIndex);
}
```
首先进行了越界检查：
```java

static void subListRangeCheck(int fromIndex, int toIndex, int size) {
if (fromIndex < 0)
    throw new IndexOutOfBoundsException("fromIndex = " + fromIndex);
if (toIndex > size)
    throw new IndexOutOfBoundsException("toIndex = " + toIndex);
if (fromIndex > toIndex)
    throw new IllegalArgumentException("fromIndex(" + fromIndex +
                                       ") > toIndex(" + toIndex + ")");
}
```
接着返回一个`SubList`的实例。这又是一个内部类，类定义如下：
```java
private class SubList extends AbstractList<E> implements RandomAccess {
private final AbstractList<E> parent;
private final int parentOffset;
private final int offset;
int size;

SubList(AbstractList<E> parent,
        int offset, int fromIndex, int toIndex) {
    this.parent = parent;
    this.parentOffset = fromIndex;
    this.offset = offset + fromIndex;
    this.size = toIndex - fromIndex;
    this.modCount = ArrayList.this.modCount;
}

public E set(int index, E e) {
    rangeCheck(index);
    checkForComodification();
    E oldValue = ArrayList.this.elementData(offset + index);
    ArrayList.this.elementData[offset + index] = e;
    return oldValue;
}

public E get(int index) {
    rangeCheck(index);
    checkForComodification();
    return ArrayList.this.elementData(offset + index);
}

public int size() {
    checkForComodification();
    return this.size;
}

public void add(int index, E e) {
    rangeCheckForAdd(index);
    checkForComodification();
    parent.add(parentOffset + index, e);
    this.modCount = parent.modCount;
    this.size++;
}

public E remove(int index) {
    rangeCheck(index);
    checkForComodification();
    E result = parent.remove(parentOffset + index);
    this.modCount = parent.modCount;
    this.size--;
    return result;
}

protected void removeRange(int fromIndex, int toIndex) {
    checkForComodification();
    parent.removeRange(parentOffset + fromIndex,
                       parentOffset + toIndex);
    this.modCount = parent.modCount;
    this.size -= toIndex - fromIndex;
}

public boolean addAll(Collection<? extends E> c) {
    return addAll(this.size, c);
}

public boolean addAll(int index, Collection<? extends E> c) {
    rangeCheckForAdd(index);
    int cSize = c.size();
    if (cSize==0)
        return false;

    checkForComodification();
    parent.addAll(parentOffset + index, c);
    this.modCount = parent.modCount;
    this.size += cSize;
    return true;
}

public Iterator<E> iterator() {
    return listIterator();
}

public ListIterator<E> listIterator(final int index) {
    checkForComodification();
    rangeCheckForAdd(index);
    final int offset = this.offset;

    return new ListIterator<E>() {
        int cursor = index;
        int lastRet = -1;
        int expectedModCount = ArrayList.this.modCount;

        public boolean hasNext() {
            return cursor != SubList.this.size;
        }

        @SuppressWarnings("unchecked")
        public E next() {
            checkForComodification();
            int i = cursor;
            if (i >= SubList.this.size)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (offset + i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i + 1;
            return (E) elementData[offset + (lastRet = i)];
        }

        public boolean hasPrevious() {
            return cursor != 0;
        }

        @SuppressWarnings("unchecked")
        public E previous() {
            checkForComodification();
            int i = cursor - 1;
            if (i < 0)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (offset + i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i;
            return (E) elementData[offset + (lastRet = i)];
        }

        @SuppressWarnings("unchecked")
        public void forEachRemaining(Consumer<? super E> consumer) {
            Objects.requireNonNull(consumer);
            final int size = SubList.this.size;
            int i = cursor;
            if (i >= size) {
                return;
            }
            final Object[] elementData = ArrayList.this.elementData;
            if (offset + i >= elementData.length) {
                throw new ConcurrentModificationException();
            }
            while (i != size && modCount == expectedModCount) {
                consumer.accept((E) elementData[offset + (i++)]);
            }
            // update once at end of iteration to reduce heap write traffic
            lastRet = cursor = i;
            checkForComodification();
        }

        public int nextIndex() {
            return cursor;
        }

        public int previousIndex() {
            return cursor - 1;
        }

        public void remove() {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                SubList.this.remove(lastRet);
                cursor = lastRet;
                lastRet = -1;
                expectedModCount = ArrayList.this.modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }

        public void set(E e) {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                ArrayList.this.set(offset + lastRet, e);
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }

        public void add(E e) {
            checkForComodification();

            try {
                int i = cursor;
                SubList.this.add(i, e);
                cursor = i + 1;
                lastRet = -1;
                expectedModCount = ArrayList.this.modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }

        final void checkForComodification() {
            if (expectedModCount != ArrayList.this.modCount)
                throw new ConcurrentModificationException();
        }
    };
}
```
注意到，这个内部类和`ArrayList`本身通过`parent`属性相关联。而这个内部类本身也有着`ArrayList`的一系列操作。具体的这个内部类能做什么，目前还没有应用到。











参考
* [RandomAccess接口的使用](http://blog.csdn.net/keda8997110/article/details/8635005)
