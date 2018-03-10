# Collections
`Collections`是一个包装类，里面封装了很多集合框架的操作，注意这个类不能实例化，因为没有公开的构造方法，该类服务于Java的Collection框架。
> Java中有很多对类与相对应的服务类，例如`Collection`和`Collections`，`Array`和`Arrays`，`Object`和`Objects`。

该类的结构如下：

![Collections](http://ovn0i3kdg.bkt.clouddn.com/Collections_1.png?imageView/2/w/400)
![Collections](http://ovn0i3kdg.bkt.clouddn.com/Collections_2.png?imageView/2/w/400)
![Collections](http://ovn0i3kdg.bkt.clouddn.com/Collections_3.png?imageView/2/w/400)

总的来说，`Collections`为`Collection`框架提供了两类方法：
1. 提供对集合进行包装的静态方法，比如把指定的集合包装成线程安全的集合、包装成不可修改的集合、包装成类型安全的集合等。
2. 提供了若干简单而又有用的算法，比如二分查找、求最大值或最小值。

下面就这两类方法做源码解读：
## 集合包装方法
在`Collections`类结构中，我们可以看到非常多的内部类，他们的名字具有有这样的特点：分别前缀(Unmodified-、Synchronized-、Checked-、Empty-、Singleton-)和后缀（`Collection`框架中的接口或类），分别对应了不可变集合、线程安全集合、类型安全集合、空集合类、单元素集合类、。

### 不可变集合
首先是不可变集合，名字统一为`UnmodifiedXXX`，顾名思义，当集合被包装成不可变集合的时候，如果对集合进行操作，将抛出`UnsupportedOperationException`。例如下面这个例子：
```java
public static void main(String[] args) {
    List<String> list = new ArrayList<>();
    list.add("a");
    list.add("b");
    list.add("c");

    List<String> unmodifiableList = Collections.unmodifiableList(list);
    System.out.print(list); //[a, b, c]

    try {
        unmodifiableList.add("d");//此处将抛出异常
    }catch (Exception e){
        System.out.println(e.getMessage());
    }

    list.add("d");
    System.out.println(list);  //[a, b, c, d, d]
}
```
上面这个测试程序中，当我们使用`Collections.unmodifiableList`对`list`进行包装后，取得该方法的返回值。当我们尝试就该不可变集合进行操作时候，会抛出异常，这一点是符合我们预期的。然而奇怪的是，我们仍旧可以对原集合进行操作，这就说明`Collections`类提供的对集合的不可变包装实际上是一个假的包装。为什么会这样？我们从源码就可以看出：
```java
static class UnmodifiableList<E> extends UnmodifiableCollection<E>
                                  implements List<E> {
    private static final long serialVersionUID = -283967356065247728L;

    final List<? extends E> list;

    UnmodifiableList(List<? extends E> list) {
        super(list);
        this.list = list;
    }

    public boolean equals(Object o) {return o == this || list.equals(o);}
    public int hashCode()           {return list.hashCode();}

    public E get(int index) {return list.get(index);}
    public E set(int index, E element) {
        throw new UnsupportedOperationException();
    }
    public void add(int index, E element) {
        throw new UnsupportedOperationException();
    }
    public E remove(int index) {
        throw new UnsupportedOperationException();
    }
    public int indexOf(Object o)            {return list.indexOf(o);}
    public int lastIndexOf(Object o)        {return list.lastIndexOf(o);}
    public boolean addAll(int index, Collection<? extends E> c) {
        throw new UnsupportedOperationException();
    }

    @Override
    public void replaceAll(UnaryOperator<E> operator) {
        throw new UnsupportedOperationException();
    }
    @Override
    public void sort(Comparator<? super E> c) {
        throw new UnsupportedOperationException();
    }

    public ListIterator<E> listIterator()   {return listIterator(0);}

    public ListIterator<E> listIterator(final int index) {
        return new ListIterator<E>() {
            private final ListIterator<? extends E> i
                = list.listIterator(index);

            public boolean hasNext()     {return i.hasNext();}
            public E next()              {return i.next();}
            public boolean hasPrevious() {return i.hasPrevious();}
            public E previous()          {return i.previous();}
            public int nextIndex()       {return i.nextIndex();}
            public int previousIndex()   {return i.previousIndex();}

            public void remove() {
                throw new UnsupportedOperationException();
            }
            public void set(E e) {
                throw new UnsupportedOperationException();
            }
            public void add(E e) {
                throw new UnsupportedOperationException();
            }

            @Override
            public void forEachRemaining(Consumer<? super E> action) {
                i.forEachRemaining(action);
            }
        };
    }

    public List<E> subList(int fromIndex, int toIndex) {
        return new UnmodifiableList<>(list.subList(fromIndex, toIndex));
    }

    /**
     * UnmodifiableRandomAccessList instances are serialized as
     * UnmodifiableList instances to allow them to be deserialized
     * in pre-1.4 JREs (which do not have UnmodifiableRandomAccessList).
     * This method inverts the transformation.  As a beneficial
     * side-effect, it also grafts the RandomAccess marker onto
     * UnmodifiableList instances that were serialized in pre-1.4 JREs.
     *
     * Note: Unfortunately, UnmodifiableRandomAccessList instances
     * serialized in 1.4.1 and deserialized in 1.4 will become
     * UnmodifiableList instances, as this method was missing in 1.4.
     */
    private Object readResolve() {
        return (list instanceof RandomAccess
                ? new UnmodifiableRandomAccessList<>(list)
                : this);
    }
}
```
以上是`UnmodifiableList`内部类的源码实现，可见该类实例的`add`方法内直接抛出了异常，由此实现“不可变”，但是原来的list并收到任何影响，可以这么想，内存中有一片区域保存了一个集合，有`list`和`unmodifiedList`两个指针指向这个地方，其中`unmodifiedList`指针的`add`方法被禁止，但是`list`指针仍旧可用，所以仍能操作这片内存。

另外需要注意的点，当使用`ArrayList`的实例和`LinkedList`的实例调用`Collections.unmodifiedList`创建得到的不可变集合是不一样的，该方法源码如下：
```java
public static <T> List<T> unmodifiableList(List<? extends T> list) {
    return (list instanceof RandomAccess ?
            new UnmodifiableRandomAccessList<>(list) :
            new UnmodifiableList<>(list));
}
```
因为`ArrayList`实现了`RandomAccess`接口，所以返回的是`UnmodifiableRandomAccessList`实例，而后者返回的是`UnmodifiableList`实例。

在`Collections`中，可以通过以下方法得到相应的不可变集合类实例：
* unmodifiableCollection
* unmodifiableList
* unmodifiableMap
* unmodifiableSet
* unmodifiableSortedMap
* unmodifiableSortedSet

### 线程安全集合
阅读源码我们可以知道，`Collections`中我们常用的一些容器类是非线程安全的，有些线程安全类比如`HashTable`我们又不常用，所以`Collections`就将非线程安全的容器类包装成线程安全类。他们的命名是"SynchronizedXXX"。以`SynchronizedList`内部类为例，源码如下：
```Java
static class SynchronizedList<E>
    extends SynchronizedCollection<E>
    implements List<E> {
    private static final long serialVersionUID = -7754090372962971524L;

    final List<E> list;

    SynchronizedList(List<E> list) {
        super(list);
        this.list = list;
    }
    SynchronizedList(List<E> list, Object mutex) {
        super(list, mutex);
        this.list = list;
    }

    public boolean equals(Object o) {
        if (this == o)
            return true;
        synchronized (mutex) {return list.equals(o);}
    }
    public int hashCode() {
        synchronized (mutex) {return list.hashCode();}
    }

    public E get(int index) {
        synchronized (mutex) {return list.get(index);}
    }
    public E set(int index, E element) {
        synchronized (mutex) {return list.set(index, element);}
    }
    public void add(int index, E element) {
        synchronized (mutex) {list.add(index, element);}
    }
    public E remove(int index) {
        synchronized (mutex) {return list.remove(index);}
    }

    public int indexOf(Object o) {
        synchronized (mutex) {return list.indexOf(o);}
    }
    public int lastIndexOf(Object o) {
        synchronized (mutex) {return list.lastIndexOf(o);}
    }

    public boolean addAll(int index, Collection<? extends E> c) {
        synchronized (mutex) {return list.addAll(index, c);}
    }

    public ListIterator<E> listIterator() {
        return list.listIterator(); // Must be manually synched by user
    }

    public ListIterator<E> listIterator(int index) {
        return list.listIterator(index); // Must be manually synched by user
    }

    public List<E> subList(int fromIndex, int toIndex) {
        synchronized (mutex) {
            return new SynchronizedList<>(list.subList(fromIndex, toIndex),
                                        mutex);
        }
    }

    @Override
    public void replaceAll(UnaryOperator<E> operator) {
        synchronized (mutex) {list.replaceAll(operator);}
    }
    @Override
    public void sort(Comparator<? super E> c) {
        synchronized (mutex) {list.sort(c);}
    }

    /**
     * SynchronizedRandomAccessList instances are serialized as
     * SynchronizedList instances to allow them to be deserialized
     * in pre-1.4 JREs (which do not have SynchronizedRandomAccessList).
     * This method inverts the transformation.  As a beneficial
     * side-effect, it also grafts the RandomAccess marker onto
     * SynchronizedList instances that were serialized in pre-1.4 JREs.
     *
     * Note: Unfortunately, SynchronizedRandomAccessList instances
     * serialized in 1.4.1 and deserialized in 1.4 will become
     * SynchronizedList instances, as this method was missing in 1.4.
     */
    private Object readResolve() {
        return (list instanceof RandomAccess
                ? new SynchronizedRandomAccessList<>(list)
                : this);
    }
}
```
很容易明白该类是如何实现线程安全的，就是将每个方法体都包含进同步块中，这样就实现了同步。

与不可变集合一样，线程安全集合同样存在“假安全”的问题。另外，对于`SynchronizedList`实例的创建同样考虑了`RandomAccess`接口。

在`Collections`中，可以通过以下方法得到相应的线程安全实例：
* synchronizedCollection
* synchronizedSet
* synchronizedSortedSet
* synchronizedNavigableSet
* synchronizedList
* synchronizedMap
* synchronizedSortedMap
* synchronizedNavigableMap

### 类型安全集合
什么叫做类型安全？就是在插入元素的时候会检查元素类型？比如元素被定义为String类型，当插入的元素是其他类型的时候就会抛出`ClassCastExceptions`异常。
>  难道泛型的使用没有解决这个问题么？

在`Collections`中，可以通过以下方法得到相应的类型安全集合实例：
* checkedCollection
* checkedList
* checkedMap
* checkedSet
* checkedSortedMap
* checkedSortedSet

### 空集合
空集合是没有元素在这些集合中，特别需要主要的是返回的集合都是**只读**的。只要更改值就会抛出UnsupportedOperationException异常。
> 集合为空且只读？那要它何用？

`Collections`中定义了三个静态常量：
```Java
public static final List EMPTY_LIST = new EmptyList<>();
public static final Map EMPTY_MAP = new EmptyMap<>();
public static final Set EMPTY_SET = new EmptySet<>();
```
我们可以使用以下方法将其返回：
* `Collections.emptyList()`—返回只读的空LIST 集合
* `Collections.emptyMap()`——返回只读的空MAP集合
* `Collections.emptySet()`——返回只读的空SET集合


### 单元素集合
`Collections`中的单元素集合指的是集合只有一个元素而且集合只读。内部类命名是`SingletonXXX`。内部类和对应的创建方法如下：

| 内部类名称 | 获取方法  |
| :------------- | :------------- |
| SingletonSet      | singleton()     |
| singletonIterator  |  Iterator() |
|   SingletonList| singletonList()  |
|SingletonMap   | singletonMap()  |

> 这个有什么用呢?


## 简单算法
`Collections`类在开头定义了这样几个常量：
```java
private static final int BINARYSEARCH_THRESHOLD   = 5000;
private static final int REVERSE_THRESHOLD        =   18;
private static final int SHUFFLE_THRESHOLD        =    5;
private static final int FILL_THRESHOLD           =   25;
private static final int ROTATE_THRESHOLD         =  100;
private static final int COPY_THRESHOLD           =   10;
private static final int REPLACEALL_THRESHOLD     =   11;
private static final int INDEXOFSUBLIST_THRESHOLD =   35;
```
虽然暂时不懂这些数字有什么用（名字听起来好像是算法的阈值），但是至少这些命名告诉我们`Collections`工具类提供了哪些算法。下面我们就看看这些算法的实现。

### 二分查找(binarySearch)
这是查找算法的最简单有效的算法，二分查找必须要求list是有序的，在无需的集合中进行二分查找没有任何意义。如果list中有多个key，不能保证哪个key被找到。
在`Collections`中重载了两个`binarySearch`方法：
```Java
/**
 *
 * 使用二分查找在指定List中查找指定元素key。 List中的元素必须是有序的。如果List中有多个key，不能确保哪个key值被找到。
 * 如果List不是有序的，返回的值没有任何意义
 *  
 * 对于随机访问列表来说，时间复杂度为O(log(n)),比如1024个数只需要查找log2(1024)=10次，
 * log2(n)是最坏的情况，即最坏的情况下都只需要找10次
 * 对于链表来说，查找中间元素的时间复杂度为O(n),元素比较的时间复杂度为O(log(n))
 *  
 * @return 查找元素的索引。如果返回的是负数表明找不到此元素，但可以用返回值计算
 *         应该将key插入到集合什么位置，任然能使集合有序(如果需要插入key值的话)。 公式：point = -i - 1
 */
public static <T> int binarySearch(List<? extends Comparable<? super T>> list, T key) {
   if (list instanceof RandomAccess || list.size()<BINARYSEARCH_THRESHOLD)
       return Collections.indexedBinarySearch(list, key);
   else
       return Collections.iteratorBinarySearch(list, key);
}

public static <T> int binarySearch(List<? extends T> list, T key, Comparator<? super T> c) {
    if (c==null)
        return binarySearch((List<? extends Comparable<? super T>>) list, key);

    if (list instanceof RandomAccess || list.size()<BINARYSEARCH_THRESHOLD)
        return Collections.indexedBinarySearch(list, key, c);
    else
        return Collections.iteratorBinarySearch(list, key, c);
}
```
第一个`binarySearch`可以接受元素类型为`Comparable`实例的list集合，而具体的实现方法是类似。我们可以看到，当`list`是`RandomAccess`实例的时候，比如`list`的类型为`Arraylist`，或者集合元素小于阈值，即5000的时候，将采用`indexedBinarySearch`方法，否则采用`iteratorBinarySearch`方法。这两个方法有什么不同呢？为什么要区别对待。

```Java
//indexedBinarySearch采取索引化二分查找
private static <T> int indexedBinarySearch(List<? extends Comparable<? super T>> list, T key) {  
    int low = 0; // 元素所在范围的下界  
    int high = list.size() - 1;  
    // 上界  

    while (low <= high) {  
        int mid = (low + high) >>> 1;  
        Comparable<? super T> midVal = list.get(mid);  
        // 中间值  
        int cmp = midVal.compareTo(key);  
        // 指定元素与中间值比较  

        if (cmp < 0)  
            low = mid + 1;  
        // 重新设置上界和下界  
        else if (cmp > 0)  
            high = mid - 1;  
        else  
            return mid; // key found  
    }  
    return -(low + 1); // key not found  
}

//iteratorBinarySearch采取迭代式二分查找。
private static <T> int iteratorBinarySearch(List<? extends Comparable<? super T>> list, T key) {  
    int low = 0;  
    int high = list.size() - 1;  
    ListIterator<? extends Comparable<? super T>> i = list.listIterator();  

    while (low <= high) {  
        int mid = (low + high) >>> 1;  
        Comparable<? super T> midVal = get(i, mid);  
        int cmp = midVal.compareTo(key);  

        if (cmp < 0)  
            low = mid + 1;  
        else if (cmp > 0)  
            high = mid - 1;  
        else  
            return mid; // key found  
    }  
    return -(low + 1); // key not found  
}

private static <T> T get(ListIterator<? extends T> i, int index) {
    T obj = null;
    int pos = i.nextIndex();
    if (pos <= index) {
        do {
            obj = i.next();
        } while (pos++ < index);
    } else {
        do {
            obj = i.previous();
        } while (--pos > index);
    }
    return obj;
}
```
可以看到两个方法的之处在于如何取得位于mid位置的元素，数组本身就是一个索引，所以使用下标访问实现随机存取。而对于链表来说，如果从链表头开始遍历，那么会非常耗时，所以借助迭代器进行遍历。这里的迭代器是双向迭代器，能很快找到对应位置上的元素。

### 反转(reverse)
反转即将集合的顺序逆置。
```Java
public static void reverse(List<?> list) {  
    int size = list.size();  
    // 如果是size小于18的链表或是基于随机访问的列表  
    if (size < REVERSE_THRESHOLD || list instanceof RandomAccess) {  
        for (int i = 0, mid = size >> 1, j = size - 1; i < mid; i++, j--)  
            // 第一个与最后一个，依次交换  
            swap(list, i, j); // 交换i和j位置的值  
    } else { // 基于迭代器的逆序排列算法  
        ListIterator fwd = list.listIterator();  
        ListIterator rev = list.listIterator(size);  
        for (int i = 0, mid = list.size() >> 1; i < mid; i++) {  
            // 这..，一个思想你懂得  
            Object tmp = fwd.next();  
            fwd.set(rev.previous());  
            rev.set(tmp);  
        }  
    }  
}  

public static void swap(List<?> list, int i, int j) {
    // instead of using a raw type here, it's possible to capture
    // the wildcard but it will require a call to a supplementary
    // private method
    final List l = list;
    l.set(i, l.set(j, l.get(i)));
}
```
可见，反转的思想是交换。对于随机访问列表比较好理解，对于链表而言，我们以迭代器的来实现交换。注意的是迭代器在取得值的同时，会修改图标，所以我们不用认为控制，但是需要控制停下来的时候，以免走过头。

### 混排（shuffle）
混排的意思就是随意排列，如果随机源是公平的，那么随便哪种顺序都是有可能的。这种算法在碰运气或洗牌的程序中比较有用。`Collections`定义了两个重载的`shuffle`，区别在于是否任务定义随机源：
```Java
/**
 *  
 * 对指定列表中的元素进行混排
 */
public static void shuffle(List<?> list) {
    Random rnd = r;
    if (rnd == null)
        r = rnd = new Random(); // harmless race.
    shuffle(list, rnd);
}

private static Random r;  

/**
*  
* 提供一个随机数生成器对指定List进行混排， 时间复杂度O(n)
*  
* 基本算法思想为： 逆向遍历list，从最后一个元素到第二个元素，然后重复交换当前位置 与随机产生的位置的元素值。
*
* 如果list不是基于随机访问并且其size>5,会先把List中的复制到数组中， 然后对数组进行混排，再把数组中的元素重新填入List中。
* 这样做为了避免迭代器大跨度查找元素影响效率
*/  
public static void shuffle(List<?> list, Random rnd) {  
   int size = list.size();  
   if (size < SHUFFLE_THRESHOLD || list instanceof RandomAccess) {  
       for (int i = size; i > 1; i--) // 从i-1个位置开始与随机位置元素交换值  
           swap(list, i - 1, rnd.nextInt(i));  
   } else {  
       Object arr[] = list.toArray(); // 先转化为数组  

       // 对数组进行混排  
       for (int i = size; i > 1; i--)  
           swap(arr, i - 1, rnd.nextInt(i));  

       // 然后把数组中的元素重新填入List  
       ListIterator it = list.listIterator();  
       for (int i = 0; i < arr.length; i++) {  
           it.next();  
           it.set(arr[i]);  
       }  
   }  
}  
```
注释已经写的很明白了，它的原理同样是交换。

#### 填充(fill)
使用指定元素替换指定列表中的所有元素，注意是所有元素。定义如下：
```java
/**
 *  
 * 用obj替换List中的所有元素 依次遍历赋值即可
 */
public static <T> void fill(List<? super T> list, T obj) {
      int size = list.size();

      if (size < FILL_THRESHOLD || list instanceof RandomAccess) {
          for (int i=0; i<size; i++)
              list.set(i, obj);
      } else {
          ListIterator<? super T> itr = list.listIterator();
          for (int i=0; i<size; i++) {
              itr.next();
              itr.set(obj);
          }
      }
  }
```
思想就是遍历赋值。

### 轮换（rotate）
轮换的意思是根据指定的距离轮转指定列表中的元素。比如类似于`[t, a, n, k, s , w] `指定长度为2或者-4的轮换之后，将变成`[s, w, t, a, n , k]`。这个算法经常被考到。
```Java
/**
 *  
 * 旋转移位List中的元素通过指定的distance。每个元素移动后的位置为： (i +
 * distance)%list.size.此方法不会改变列表的长度
 * distance表示移动的位置，可以这么理解：正数表示向后移，负数表示向前移，0表示不移动
 *
 */  
public static void rotate(List<?> list, int distance) {  
    if (list instanceof RandomAccess || list.size() < ROTATE_THRESHOLD)  
        rotate1((List) list, distance);  
    else  
        rotate2((List) list, distance);  
}  

/**
 * 对于随机访问列表，先计算出移动后的位置，然后赋值即可。
 */
private static <T> void rotate1(List<T> list, int distance) {  
    int size = list.size();  
    if (size == 0)  
        return;  
    distance = distance % size; // distance始终处于0到size(不包括)之间  
    if (distance < 0)  
        distance += size; // 还是以向后移来计算的  
    if (distance == 0)  
        return;  

    for (int cycleStart = 0, nMoved = 0; nMoved != size; cycleStart++) {  
        T displaced = list.get(cycleStart);  
        int i = cycleStart;  
        do {  
            i += distance; // 求新位置  
            if (i >= size)  
                i -= size; // 超出size就减去size  
            displaced = list.set(i, displaced);  
            // 为新位置赋原来的值  
            nMoved++; // 如果等于size证明全部替换完毕  
        } while (i != cycleStart); // 依次类推，求新位置的新位置！
    }  
}  

private static void rotate2(List<?> list, int distance) {  
    int size = list.size();  
    if (size == 0)  
        return;  
    int mid = -distance % size;  
    if (mid < 0)  
        mid += size;  
    if (mid == 0)  
        return;  
    // 好神奇啊  
    reverse(list.subList(0, mid));  
    reverse(list.subList(mid, size));  
    reverse(list);  
}  
```
这里特别注意对于链表的操作，非常巧妙！对于[1,2,3,4,5,6]来说，当distance为2时候，`-2%6`的结果是-2，最后取得mid=4，两次reverse过后变成[5,6,1,2,3,4]，最后一次reverse之后变成[5,6,1,2,3,4]，非常神奇！

### 拷贝方法（copy）
这个没什么好解释的，就是遍历赋值。同样对随机存取的数组和链表做了不同的处理。定义如下：
```Java
/**
*  
* 复制源列表的所有元素到目标列表， 如果src.size > dest.size 将抛出一个异常 如果src.size < dest.size
* dest中多出的元素将不受影响 同样是依次遍历赋值
*/  
public static <T> void copy(List<? super T> dest, List<? extends T> src) {  
   int srcSize = src.size();  
   if (srcSize > dest.size())  
       throw new IndexOutOfBoundsException("Source does not fit in dest");  

   if (srcSize < COPY_THRESHOLD || (src instanceof RandomAccess && dest instanceof RandomAccess)) {  
       for (int i = 0; i < srcSize; i++)  
           dest.set(i, src.get(i));  
   } else { // 一个链表一个线性表也可以用迭代器赋值  
       ListIterator<? super T> di = dest.listIterator();  
       ListIterator<? extends T> si = src.listIterator();  
       for (int i = 0; i < srcSize; i++) {  
           di.next();  
           di.set(si.next());  
       }  
   }  
}
```

### 替代方法（replaceAll）
把指定集合中所有与oladVal相等的元素替换成newVal 只要list发生了改变就返回true。定义如下：
```Java
public static <T> boolean replaceAll(List<T> list, T oldVal, T newVal) {  
    boolean result = false;  
    int size = list.size();  
    if (size < REPLACEALL_THRESHOLD || list instanceof RandomAccess) {  
        if (oldVal == null) {  
            for (int i = 0; i < size; i++) {  
                if (list.get(i) == null) {  
                    list.set(i, newVal);  
                    result = true;  
                }  
            }  
        } else {  
            for (int i = 0; i < size; i++) {  
                if (oldVal.equals(list.get(i))) {  
                    list.set(i, newVal);  
                    result = true;  
                }  
            }  
        }  
    } else {  
        ListIterator<T> itr = list.listIterator();  
        if (oldVal == null) {  
            for (int i = 0; i < size; i++) {  
                if (itr.next() == null) {  
                    itr.set(newVal);  
                    result = true;  
                }  
            }  
        } else {  
            for (int i = 0; i < size; i++) {  
                if (oldVal.equals(itr.next())) {  
                    itr.set(newVal);  
                    result = true;  
                }  
            }  
        }  
    }  
    return result;  
}  
```
遍历比较，匹配则赋值。非常好理解。

### 子串匹配（indexOfSubList和lastIndexOfSubList）
`indexOfSubList`用来返回指定源列表中第一次出现指定目标列表的起始位置，如果没有出现这样的列表，则返回-1；`lastIndexOfSubList`用来返回指定源列表中最后一次出现指定目标列表的起始位置，如果没有出现这样的列表，则返回-1。
```Java
/**
 *  
 * target是否是source的子集，如果是返回target第一个元素的索引， 否则返回-1。
 * 其实这里和串的模式匹配差不多。这里使用的是基本的回溯法。
 *  
 */  
public static int indexOfSubList(List<?> source, List<?> target) {  
    int sourceSize = source.size();  
    int targetSize = target.size();  
    int maxCandidate = sourceSize - targetSize;  

    if (sourceSize < INDEXOFSUBLIST_THRESHOLD  
            || (source instanceof RandomAccess && target instanceof RandomAccess)) {  
        nextCand: for (int candidate = 0; candidate <= maxCandidate; candidate++) {  
            for (int i = 0, j = candidate; i < targetSize; i++, j++)  
                if (!eq(target.get(i), source.get(j)))  
                    continue nextCand; // 元素失配，跳到外部循环  
            return candidate; // All elements of candidate matched target  
        }  
    } else { // Iterator version of above algorithm  
        ListIterator<?> si = source.listIterator();  
        nextCand: for (int candidate = 0; candidate <= maxCandidate; candidate++) {  
            ListIterator<?> ti = target.listIterator();  
            for (int i = 0; i < targetSize; i++) {  
                if (!eq(ti.next(), si.next())) {  
                    // 回溯指针，然后跳到外部循环继续执行  
                    for (int j = 0; j < i; j++)  
                        si.previous();  
                    continue nextCand;  
                }  
            }  
            return candidate;  
        }  
    }  
    return -1; // 没有找到匹配的子串返回-1  
}

/**
 * 如果有一个或多个字串，返回最后一个出现的子串的第一个元素的索引
 */  
public static int lastIndexOfSubList(List<?> source, List<?> target) {  
    int sourceSize = source.size();  
    int targetSize = target.size();  
    int maxCandidate = sourceSize - targetSize;  

    if (sourceSize < INDEXOFSUBLIST_THRESHOLD || source instanceof RandomAccess) { // Index  
                                                                                    // access  
                                                                                    // version  
        nextCand: for (int candidate = maxCandidate; candidate >= 0; candidate--) {  
            for (int i = 0, j = candidate; i < targetSize; i++, j++)  
                if (!eq(target.get(i), source.get(j)))  
                    // 从source的maxCandidate位置开始比较。然后是maxCandidate-1，依次类推  
                    continue nextCand; // Element mismatch, try next cand  
            return candidate; // All elements of candidate matched target  
        }  
    } else { // Iterator version of above algorithm  
        if (maxCandidate < 0)  
            return -1;  
        ListIterator<?> si = source.listIterator(maxCandidate);  
        nextCand: for (int candidate = maxCandidate; candidate >= 0; candidate--) {  
            ListIterator<?> ti = target.listIterator();  
            for (int i = 0; i < targetSize; i++) {  
                if (!eq(ti.next(), si.next())) {  
                    if (candidate != 0) {  
                        // Back up source iterator to next candidate  
                        for (int j = 0; j <= i + 1; j++)  
                            si.previous();  
                    }  
                    continue nextCand;  
                }  
            }  
            return candidate;  
        }  
    }  
    return -1; // No candidate matched the target  
}
```

### 排序（Sort）
`sort`方法实现对集合的排序。定义如下
```Java
public static <T extends Comparable<? super T>> void sort(List<T> list) {
    list.sort(null);
}
```
在JDK1.8中，`List`接口通过`default`关键字实现了`sort`方法：
```Java
@SuppressWarnings({"unchecked", "rawtypes"})
default void sort(Comparator<? super E> c) {
    Object[] a = this.toArray();
    Arrays.sort(a, (Comparator) c);
    ListIterator<E> i = this.listIterator();
    for (Object e : a) {
        i.next();
        i.set((E) e);
    }
}
```
可见，底层实现的时候是调用的`Arrays`的`Sort`方法，然后在将排好序的数组重新赋值。

### 最大(max)和最小(min)
求出集合中的最大元素和最小元素，方法很简单，就是用一个变量保存当前最小或最大值，然后遍历集合，更新该值，最后返回这个值即可。定义如下：
```Java
/**
*  
* 返回集合中的最小元素。前提是其中的元素都是可比的，即实现了Comparable接口 找出一个通用的算法其实不容易，尽管它的思想不难。
* 反正要依次遍历完所有元素，所以直接用了迭代器
*/  
public static <T extends Object & Comparable<? super T>> T min(Collection<? extends T> coll) {  
   Iterator<? extends T> i = coll.iterator();  
   T candidate = i.next();  
   while (i.hasNext()) {  
       T next = i.next();  
       if (next.compareTo(candidate) < 0)  
           candidate = next;  
   }  
   return candidate;  
}  

/**
* 根据提供的比较器求最小元素
*/  
public static <T> T min(Collection<? extends T> coll, Comparator<? super T> comp) {  
   if (comp == null)  
       // 返回默认比较器，其实默认比较器什么也不做，只是看集合元素是否实现了Comparable接口，  
       // 否则抛出ClassCastException  
       return (T) min((Collection<SelfComparable>) (Collection) coll);  

   Iterator<? extends T> i = coll.iterator();  
   T candidate = i.next();  
   // 假设第一个元素为最小元素  

   while (i.hasNext()) {  
       T next = i.next();  
       if (comp.compare(next, candidate) < 0)  
           candidate = next;  
   }  
   return candidate;  
}  

/**
* 求集合中最大元素
*/  
public static <T extends Object & Comparable<? super T>> T max(Collection<? extends T> coll) {  
   Iterator<? extends T> i = coll.iterator();  
   T candidate = i.next();  

   while (i.hasNext()) {  
       T next = i.next();  
       if (next.compareTo(candidate) > 0)  
           candidate = next;  
   }  
   return candidate;  
}  

/**
* 根据指定比较器求集合中最大元素
*/  
public static <T> T max(Collection<? extends T> coll, Comparator<? super T> comp) {  
   if (comp == null)  
       return (T) max((Collection<SelfComparable>) (Collection) coll);  

   Iterator<? extends T> i = coll.iterator();  
   T candidate = i.next();  

   while (i.hasNext()) {  
       T next = i.next();  
       if (comp.compare(next, candidate) > 0)  
           candidate = next;  
   }  
   return candidate;  
}
```

> 阅读源码可以知道，`Collections`实现了一些简单有用的算法，其中对随机访问的列表和链表分别做了特殊处理，前者采用索引式存取，后者采取迭代器，提高了效率。同时其中一些算法的思路值得好好咀嚼，比如对于链表的rotate的处理，非常巧妙！



参考
* [Java Collections和Arrays工具类剖析](http://blog.csdn.net/excellentyuxiao/article/details/52344594)
* [理解不可变集合 | Guava Immutable与JDK unmodifiableList](https://www.jianshu.com/p/bf2623f18d6a)
* [Collections源码](http://blog.csdn.net/u014082714/article/details/51459189)
