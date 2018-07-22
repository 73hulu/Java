# Arrays
<!-- toc -->
<!-- tocstop  -->

`Arrays`类是Java集合框架的一部分，提供了对数组的处理方法。注意，`Arrays`是java.util中的一个工具类，在java.lang.reflect中，有一个`Array`类，这两种的关系，与`Objects`和`Object`、`Collections`和`Collection`非常类似。`Arrays`是一个工具类，提供了对数组的操作，比如排序、查找、与线性表的转化，而`Array`则**定义**和操作数组，可以用`Array.newInstance`来创建数组（虽然我们平常不使用这种方法来创建数组），操作数组当然是通过`get`和`set`方法。

> Arrays 无公共的构造方法，私有构造方法的方法体为空。

`Arrays`中提供了众多的数组操作方法，大部分是已经类型的重载，太多了所以就不截图了，主要提供了一下类别的方法：
1. 查找方法
2. 排序方法
3. 填充方法
4. 拷贝方法
5. 哈希方法
6. 判别相等方法
7. 打印方法
8. 转化为线性表

> `Arrays`中重载主要是针对数据类型（eg. int[], double[], float[], long[]，对象数组）和操作范围（left, right）。

## 查找方法
`Arrays`中采取的是二分法查找，二分法有一个前提是必须有序，算法的时间复杂度是O(logn)。
下面是一个int类型数数组的二分查找算法：
```Java
public static int binarySearch(int[] a, int fromIndex, int toIndex,
                               int key) {
    rangeCheck(a.length, fromIndex, toIndex);
    return binarySearch0(a, fromIndex, toIndex, key);
}

private static void rangeCheck(int arrayLength, int fromIndex, int toIndex) {
    if (fromIndex > toIndex) {
        throw new IllegalArgumentException(
                "fromIndex(" + fromIndex + ") > toIndex(" + toIndex + ")");
    }
    if (fromIndex < 0) {
        throw new ArrayIndexOutOfBoundsException(fromIndex);
    }
    if (toIndex > arrayLength) {
        throw new ArrayIndexOutOfBoundsException(toIndex);
    }
}

private static int binarySearch0(int[] a, int fromIndex, int toIndex,
                                 int key) {
    int low = fromIndex;
    int high = toIndex - 1;

    while (low <= high) {
        int mid = (low + high) >>> 1;
        int midVal = a[mid];

        if (midVal < key)
            low = mid + 1;
        else if (midVal > key)
            high = mid - 1;
        else
            return mid; // key found
    }
    return -(low + 1);  // key not found.
}
```
如果找到则返回元素下标，如果没有找到则返回元素应该插入的位置下标。注意到该方法在取mid位置的时候并可能会存在整数溢出问题！而`Collections`中的`binarySearch`则考虑到这个问题，所以更好的取mid的方法是：
```Java
int mid = left + (right - left) >>> 1 ;
```

## 排序方法
`Arrays`定义了两类排序的方法：`sort`和`parallelSort`方法。

### sort方法
sort方法可以用来对基本类型或对象数组进行排序，可自定义排序方位，但是不能自定义比较器（默认是升序）。下面是对int数组的排序：
```Java
public static void sort(int[] a, int fromIndex, int toIndex) {
    rangeCheck(a.length, fromIndex, toIndex);
    DualPivotQuicksort.sort(a, fromIndex, toIndex - 1, null, 0, 0);
}
```
同样首先进行范围检查，然后进行真正的排序。

在1.7之前，`Arrays.sort`使用的是我们普通的快速排序方法，即单轴快速排序。但是在1.7之后使用的是`DualPivotQuicksort`中的`sort`方法。该类名翻译过来就是“双轴快速排序”，顾名思义即基于两个轴来进行比较，而普通的快排是选择一个点来作为轴点。该方法的具体实现比较复杂，请参考`DualPivotQuicksort`类的说明。

`sort`方法同样可以作用域对象数组，这个可以定下面是对对象数组的排序操作：
```Java
public static <T> void sort(T[] a, int fromIndex, int toIndex,
                                Comparator<? super T> c) {
    if (c == null) {
        sort(a, fromIndex, toIndex);
    } else {
        rangeCheck(a.length, fromIndex, toIndex);
        if (LegacyMergeSort.userRequested)
            legacyMergeSort(a, fromIndex, toIndex, c);
        else
            TimSort.sort(a, fromIndex, toIndex, c, null, 0, 0);
    }
}

static final class LegacyMergeSort {
    private static final boolean userRequested =
        java.security.AccessController.doPrivileged(
            new sun.security.action.GetBooleanAction(
                "java.util.Arrays.useLegacyMergeSort")).booleanValue();
}
```
可以看到，对于对象数组，我们可以自定义比较器，当比较器赋值为null的时候，采用自然排序。如果定义了比较器，那么需要对`LegacyMergeSort.userRequested`进行判断，这个的意思大概就是“用户请求传统归并排序”，我们可以通过以下设置将这个变量设置为true：
```Java
System.setProperty("java.util.Arrays.useLegacyMergeSort", "true");  
```
当这个值为true的时候，采用的是传统的归并排序，`legacyMergeSort`的定义如下：
```Java
private static <T> void legacyMergeSort(T[] a, int fromIndex, int toIndex,
                                            Comparator<? super T> c) {
    T[] aux = copyOfRange(a, fromIndex, toIndex);
    if (c==null)
        mergeSort(aux, a, fromIndex, toIndex, -fromIndex);
    else
        mergeSort(aux, a, fromIndex, toIndex, -fromIndex, c);
}

private static void mergeSort(Object[] src,
                                  Object[] dest,
                                  int low,
                                  int high,
                                  int off) {
    int length = high - low;

    // 小数组（小于7）采取插入排序
    if (length < INSERTIONSORT_THRESHOLD) {
        for (int i=low; i<high; i++)
            for (int j=i; j>low &&
                     ((Comparable) dest[j-1]).compareTo(dest[j])>0; j--)
                swap(dest, j, j-1);
        return;
    }

    // Recursively sort halves of dest into src
    int destLow  = low;
    int destHigh = high;
    low  += off;
    high += off;
    int mid = (low + high) >>> 1;
    mergeSort(dest, src, low, mid, -off);
    mergeSort(dest, src, mid, high, -off);

    // If list is already sorted, just copy from src to dest.  This is an
    // optimization that results in faster sorts for nearly ordered lists.
    //如果序列已经有序，那么直接将src拷贝到desc中，这个方法对于原本就接近有序的数组十分有效
    if (((Comparable)src[mid-1]).compareTo(src[mid]) <= 0) {
        System.arraycopy(src, low, dest, destLow, length);
        return;
    }

    // Merge sorted halves (now in src) into dest
    for(int i = destLow, p = low, q = mid; i < destHigh; i++) {
        if (q >= high || p < mid && ((Comparable)src[p]).compareTo(src[q])<=0)
            dest[i] = src[p++];
        else
            dest[i] = src[q++];
    }
}
```
然而事实上，这里实际上并不是严格意义上的”归并排序”，当待排序的对象数组很小（长度小于7）的时候要进行插入排序。其他情况采取归并排序，并且这个过程也做了优化。我们知道归并排序是将先细分再合并，当原数组拆分为两个数组，并各自进行排序，然后将两个有序数组进行合并。这里做出的优化是，先看下这个数组是不是已经有序了，依据是左边最大值和右边最小值是否有序，即src[mid-1]小于src[mid]，如果是的话，我们就可以直接将src复制拷贝到desc中，这里用的是`System.arraycopy`方法，而不用在进行遍历赋值。这种情况对轻微乱序的数组比较高效。如果没有有序，那就合并，合并的原理也很简单，就是将两个数组的依次比较，然后按照大小依次赋值到desc数组中。

`sort`方法可以总结如下：

![Arrays.sort](http://ovn0i3kdg.bkt.clouddn.com/Arrays.sort)

### parallelSort
意思是“并行排序”，暂时不知道有何用，先不看了。



## 填充方法
填充就是将数组全部赋值为一个值。这个算法很简单，就是循环遍历然后赋值。这里需要注意一下对于对象数组的填充：
```Java
public static void fill(Object[] a, int fromIndex, int toIndex, Object val) {
    rangeCheck(a.length, fromIndex, toIndex);
    for (int i = fromIndex; i < toIndex; i++)
        a[i] = val;
}
```
可以看到，数组的所有元素都赋值为同一个元素，下面是一个测试程序：
```Java
private static class  Person{
        private String name;
        private short age;

        public Person(String name, short age) {
            this.name = name;
            this.age = age;
        }
        // omit setter and getter

        @Override
        public String toString() {
            return "Person{" +
                    "name='" + name + '\'' +
                    ", age=" + age +
                    '}';
        }
    }

  public static void main(String[] args) {

      Person person = new Person("baobao", (short) 18);

      Person[] people = new Person[3];


      Arrays.fill(people, person);

      System.out.println(Arrays.toString(people)); //[Person{name='baobao', age=18}, Person{name='baobao', age=18}, Person{name='baobao', age=18}]

      people[0].setAge((short) 17);

      System.out.println(Arrays.toString(people)); //[Person{name='baobao', age=17}, Person{name='baobao', age=17}, Person{name='baobao', age=17}]

      String str = "aaa";

      String[] strArr = new String[3];

      Arrays.fill(strArr, str);

      System.out.println(Arrays.toString(strArr)); //[aaa, aaa, aaa]

      strArr[0] = "bbb";

      System.out.println(Arrays.toString(strArr)); //[bbb, aaa, aaa]    
  }
```
`Person`对象数组果然跟我们想的一样，指向的是同一个对象，所以改变这个对象，所有的元素都受到影响。但是`String`数组为什么没有这种结果？因为`String`是不可变类，同理对于基本类型包装类的缓存区间也有这种效果。

## 拷贝方法
`Arrays`提供了`copyOf`和`copyOfRange`方法，或者定义了拷贝的范围，比前者多了一个范围检查。对于`copyOf`方法，同样可分为对基本数据类型数组的处理和对对象数据类型的处理：
```Java
//处理基本数据类型数组
public static int[] copyOf(int[] original, int newLength) {
    int[] copy = new int[newLength];
    System.arraycopy(original, 0, copy, 0,
                     Math.min(original.length, newLength));
    return copy;
}

//对于对象数组
public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
    @SuppressWarnings("unchecked")
    T[] copy = ((Object)newType == (Object)Object[].class)
        ? (T[]) new Object[newLength]
        : (T[]) Array.newInstance(newType.getComponentType(), newLength);
    System.arraycopy(original, 0, copy, 0,
                     Math.min(original.length, newLength));
    return copy;
}
```
两个方法的根源都是调用了`System`的数组拷贝方法`arraycopy`，但是对于对象数组，可以指定拷贝类型，这里用到了`Array`创建数组的方法。那么有一个疑问，`System.arraycopy`对于对象数组到底是浅拷贝还是深拷贝呢？做一个实验：
```Java
public static void main(String[] args) {

    String[] origin = new String[]{"a", "b", "c"};

    display(origin); //[a, b, c]

    String[] copy = Arrays.copyOf(origin, origin.length, String[].class);

    display(copy);//[a, b, c]

    System.out.print(origin[0] == copy[0]); //true
}

public static void display(String[] arr){
    System.out.println(Arrays.toString(arr));
}
```
证明这是浅拷贝。


## 哈希方法

哈希方式将提供整个数组的哈希方法。对于数组A，设其中元素A[i]的哈希值为hash(A[i])，那么`Arrays`提供的哈希函数定义为
```
int hashValue = ∑ hash(A[i]) * 31 ^(n-i)。
```
是不是有点眼熟，其实这就是`String`求哈希的方法。hash(A[i])实际上解决的是数组元素自己求哈希的方法。
`Arrays`提供了两种求数组哈希值的方法：`hashValue`和`deepHashCode`。这两个的差别在于后者常用来处理多维数组。
### hashCode
下面是对于long类型数组的哈希方法：
```Java
public static int hashCode(long a[]) {
    if (a == null)
        return 0;

    int result = 1;
    for (long element : a) {
        int elementHash = (int)(element ^ (element >>> 32));
        result = 31 * result + elementHash;
    }
    return result;
}
```
其中`elementHash`就是单个元素的哈希值。

对于对象数组，哈希函数的定义如下：
```Java
public static int hashCode(Object a[]) {
    if (a == null)
        return 0;

    int result = 1;

    for (Object element : a)
        result = 31 * result + (element == null ? 0 : element.hashCode());

    return result;
}
```
同样很好理解。

### deepHashCode
`deepHashCode`可以用来处理多维数组，其定义如下：
```Java
public static int deepHashCode(Object a[]) {
    if (a == null)
        return 0;

    int result = 1;

    for (Object element : a) {
        int elementHash = 0;
        if (element instanceof Object[])
            elementHash = deepHashCode((Object[]) element);
        else if (element instanceof byte[])
            elementHash = hashCode((byte[]) element);
        else if (element instanceof short[])
            elementHash = hashCode((short[]) element);
        else if (element instanceof int[])
            elementHash = hashCode((int[]) element);
        else if (element instanceof long[])
            elementHash = hashCode((long[]) element);
        else if (element instanceof char[])
            elementHash = hashCode((char[]) element);
        else if (element instanceof float[])
            elementHash = hashCode((float[]) element);
        else if (element instanceof double[])
            elementHash = hashCode((double[]) element);
        else if (element instanceof boolean[])
            elementHash = hashCode((boolean[]) element);
        else if (element != null)
            elementHash = element.hashCode();

        result = 31 * result + elementHash;
    }

    return result;
}
```
可以看到，对于普通的对象数组和基本类型数组，这个方法和`hashCode`无差别，但是对于多维数组而言，其`elementHash`是递归的。

## 判别相等方法
提供了`equals`方法和`deepEquals`方法。
### equals
这个算法实现很简单，就是依次比较对数组元素是否相等，对于基本数据类型，这里的“相等”用的是“=”号，对于对象，这里的“相等”用的是`equals`：
```Java
public static boolean equals(Object[] a, Object[] a2) {
    if (a==a2)
        return true;
    if (a==null || a2==null)
        return false;

    int length = a.length;
    if (a2.length != length)
        return false;

    for (int i=0; i<length; i++) {
        Object o1 = a[i];
        Object o2 = a2[i];
        if (!(o1==null ? o2==null : o1.equals(o2)))
            return false;
    }

    return true;
}
```
可以注意到，当数组是多维数组的时候，该比较方法不太好。所以才有了`deepEquals`方法。

### deepEquals
和`deepHashCode`方法一样，这个方法的设计专门服务于多维数组：
```Java
public static boolean deepEquals(Object[] a1, Object[] a2) {
    if (a1 == a2)
        return true;
    if (a1 == null || a2==null)
        return false;
    int length = a1.length;
    if (a2.length != length)
        return false;

    for (int i = 0; i < length; i++) {
        Object e1 = a1[i];
        Object e2 = a2[i];

        if (e1 == e2)
            continue;
        if (e1 == null)
            return false;

        // Figure out whether the two elements are equal
        boolean eq = deepEquals0(e1, e2);

        if (!eq)
            return false;
    }
    return true;
}

static boolean deepEquals0(Object e1, Object e2) {
    assert e1 != null;
    boolean eq;
    if (e1 instanceof Object[] && e2 instanceof Object[])
        eq = deepEquals ((Object[]) e1, (Object[]) e2);
    else if (e1 instanceof byte[] && e2 instanceof byte[])
        eq = equals((byte[]) e1, (byte[]) e2);
    else if (e1 instanceof short[] && e2 instanceof short[])
        eq = equals((short[]) e1, (short[]) e2);
    else if (e1 instanceof int[] && e2 instanceof int[])
        eq = equals((int[]) e1, (int[]) e2);
    else if (e1 instanceof long[] && e2 instanceof long[])
        eq = equals((long[]) e1, (long[]) e2);
    else if (e1 instanceof char[] && e2 instanceof char[])
        eq = equals((char[]) e1, (char[]) e2);
    else if (e1 instanceof float[] && e2 instanceof float[])
        eq = equals((float[]) e1, (float[]) e2);
    else if (e1 instanceof double[] && e2 instanceof double[])
        eq = equals((double[]) e1, (double[]) e2);
    else if (e1 instanceof boolean[] && e2 instanceof boolean[])
        eq = equals((boolean[]) e1, (boolean[]) e2);
    else
        eq = e1.equals(e2);
    return eq;
}
```
可以看到，对于多维数组，它的`deepEquals`比较是递归的。
下面是一个测试程序。
```Java
public static void main(String[] args) {
    int[][] arr1 = new int[][]{
            {1,1,1},
            {2,3,2},
    };
    int[][] arr2 = new int[][]{
            {1,1,1},
            {2,3,2},
    };

    System.out.println(Arrays.equals(arr1, arr2)); //false
    System.out.println(Arrays.deepEquals(arr1, arr2)); //true
}
```

## 打印方法
提供了`toString`和`deepToString`两个方法，同样后者专门服务与多维数组
```Java
public static String toString(long[] a) {
    if (a == null)
        return "null";
    int iMax = a.length - 1;
    if (iMax == -1)
        return "[]";

    StringBuilder b = new StringBuilder();
    b.append('[');
    for (int i = 0; ; i++) {
        b.append(a[i]);
        if (i == iMax)
            return b.append(']').toString();
        b.append(", ");
    }
}

public static String deepToString(Object[] a) {
    if (a == null)
        return "null";

    int bufLen = 20 * a.length;
    //有长度限制
    if (a.length != 0 && bufLen <= 0)
        bufLen = Integer.MAX_VALUE;
    StringBuilder buf = new StringBuilder(bufLen);
    deepToString(a, buf, new HashSet<Object[]>());
    return buf.toString();
}
```


下面是一个测试程序
```Java
public static void main(String[] args) {
    int[][] arr1 = new int[][]{
            {1,1,1},
            {2,3,2},
    };

    System.out.println(arr1.toString());//[[I@2503dbd3
    System.out.println(Arrays.toString(arr1)); //[[I@4b67cf4d, [I@7ea987ac]
    System.out.println(Arrays.deepToString(arr1)); //[[1, 1, 1], [2, 3, 2]]
}
```
可见数组本身的toString只能打印出该对象的十六进制的地址，而`Arrays.toString`能打印出一维数组的准确元素值，对于多维数组，只能用`Arrays.deepToString`方法打印全部值。

> 这个方法要眼熟啊，以前经常自己写for循环，太傻了这有现成的。

## 转为线性表
`asList`，将数组转为线性表，准确来说是`ArrayList`，是加上就是调用了`ArrayList`的构造方法。
```java
@SafeVarargs
 @SuppressWarnings("varargs")
 public static <T> List<T> asList(T... a) {
     return new ArrayList<>(a);
 }
```
需要注意的是，这个方法接受的不是一个数组，而是一个操作表，所以写法如下：
```Java
List<Integer> list1 = Arrays.asList(new int[]{1,2,3}); //错误
List<Integer> list2 = Arrays.asList(1, 2, 3); //正确
```

BUT! `ArrayList.asList()`返回的结果并不是`java.util.ArrayList`的实例，而是`java.util.Arrays.ArrayList`的实例，这个内部类的定义如下：
```java
private static class ArrayList<E> extends AbstractList<E>
        implements RandomAccess, java.io.Serializable
{
    private static final long serialVersionUID = -2764017481108945198L;
    private final E[] a;

    ArrayList(E[] array) {
        a = Objects.requireNonNull(array);
    }

    @Override
    public int size() {
        return a.length;
    }

    @Override
    public Object[] toArray() {
        return a.clone();
    }

    @Override
    @SuppressWarnings("unchecked")
    public <T> T[] toArray(T[] a) {
        int size = size();
        if (a.length < size)
            return Arrays.copyOf(this.a, size,
                                 (Class<? extends T[]>) a.getClass());
        System.arraycopy(this.a, 0, a, 0, size);
        if (a.length > size)
            a[size] = null;
        return a;
    }

    @Override
    public E get(int index) {
        return a[index];
    }

    @Override
    public E set(int index, E element) {
        E oldValue = a[index];
        a[index] = element;
        return oldValue;
    }

    @Override
    public int indexOf(Object o) {
        E[] a = this.a;
        if (o == null) {
            for (int i = 0; i < a.length; i++)
                if (a[i] == null)
                    return i;
        } else {
            for (int i = 0; i < a.length; i++)
                if (o.equals(a[i]))
                    return i;
        }
        return -1;
    }

    @Override
    public boolean contains(Object o) {
        return indexOf(o) != -1;
    }

    @Override
    public Spliterator<E> spliterator() {
        return Spliterators.spliterator(a, Spliterator.ORDERED);
    }

    @Override
    public void forEach(Consumer<? super E> action) {
        Objects.requireNonNull(action);
        for (E e : a) {
            action.accept(e);
        }
    }

    @Override
    public void replaceAll(UnaryOperator<E> operator) {
        Objects.requireNonNull(operator);
        E[] a = this.a;
        for (int i = 0; i < a.length; i++) {
            a[i] = operator.apply(a[i]);
        }
    }

    @Override
    public void sort(Comparator<? super E> c) {
        Arrays.sort(a, c);
    }
}
```
可以看到，这个类的实例只有一些面向位置和判断的操作，但是没有`add`、`remove`等修改线性表的方法，所以，我们用上面的方法得到的实际上并不是我们想要的。那么如何取得`java.util.ArrayList`的实例呢。首先想到的方法就是用`Arrays.asList`的结果作为`ArrayList`的构造参数，如下：
```java
ArrayList<String> list = new ArrayList(Arrays.asList("a", "b", "c"));
```
实际上一个更加高效的代码是：
```java
String[] array = new String[]{"a", "b", "c"};
ArrayList<String> list = new ArrayList<String>(array.length);
Collections.addAll(list, Arrays.asList(array));
```

以上是数组转化为线性表的方法，那么如何逆操作呢？即将线性表转为数组？调用线性表的`toArray`方法！
