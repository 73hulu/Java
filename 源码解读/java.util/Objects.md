# Objects

`Objects`和我们学到的`Object`类可不是同一个东西，后者是Java中所有类的父类，而前者是对象操作的工具类而已。没错，`Objects`类之于`Object`，就类似于`Arrays`之于`Array`，`Collections`之于`Collection`一样，只是一个工具类而已。

`Objects`工具类提供了对象操作的方法，它提供的方法都是静态方法。比如比较两个对象是否相等，返回对象的哈希码，取得对象的string值等等。

> 其实仔细想想这些方法是怎么实现的呢？还是调用了`Object`对象本身的方法。既然`Object`就可以解决的事情，为什么还要另外写一个工具类。原因大概有二：一是再封装一层，工具类的作用不就是为了方便么，比如说如果我们判断一个对象是不是null, 那么就每个地方都写`object == null`，这不是low的不是？二是为了解决特殊的对象null可能造成的`NullPointException`，这个工具类都提前帮你处理好的。

类结构如下：

![Objects](http://ovn0i3kdg.bkt.clouddn.com/Objects.png)

### public final class Objects
类声明，被final修饰，说明不能被继承。


### private Objects() {...}
构造方法，是私有的，说明这个类是没有办法实例化的。而这个私有构造方法是怎么实现的呢？
```Java
private Objects() {
   throw new AssertionError("No java.util.Objects instances for you!");
}
```
实际上，这个类不能被继承，构造方法私有，按道理将说，不会被调用才是，但是这个方法仍旧在方法体中抛出了异常，这是防止有人调用它进行实例化？那什么时候可能被调用呢？难道是反射？

另外需要注意的是，这个类抛出的异常是`AssertionError`，这是一个`Error`类的子类，属于JVM严重错误了。

### public static boolean equals(Object a, Object b){...}
静态方法，顾名思义，就是比较两个对象是不是相等?什么是相等呢？如果两个对象指向同一个，那么可能相等。如果指向不一样时，对象按照自己的评判标准判定相等，那么它们就是相等的，比如`String`类型，如果对应字符位上是相等的，那么两者就是相等的。实现如下：
```Java
public static boolean equals(Object a, Object b) {
    return (a == b) || (a != null && a.equals(b));
}
```

### public static boolean deepEquals(Object a, Object b){...}
`deepEquals`方法是深层比较的方法，实际上调用的是`Arrays`的`deepEquals0`方法，说明这个方法主要是为数组对象服务的，其结果与`Arrays`的`deepEquals`比较结果一致。如果参数不是数组，那么与`equals`方法的结果是一直的。
```Java
public static boolean deepEquals(Object a, Object b) {
    if (a == b)
        return true;
    else if (a == null || b == null)
        return false;
    else
        return Arrays.deepEquals0(a, b);
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
        eq = e1.equals(e2); // 如果都是非数组参数，那么结果与`equals`无异
    return eq;
}
```

### public static int hashCode(Object o){...}
用来返回对象的哈希值，本质还是调用对象的`hashCode`方法，但是它对null值做了特殊的处理，约定null值的哈希值为0。
```Java
public static int hashCode(Object o) {
    return o != null ? o.hashCode() : 0;
}
```

> 这个hashCode方法被应用到了HashMap的使用中，对于key的哈希计算就是调用了这个方法。

### public static int hash(Object... values){...}
注意这个`hash`方法和`hashCode`方法是不一样的。首先不一样的地方在于参数，它可以接受一组边长参数，最后返回的结果是这组参数共同组成的哈希值。定义如下：
```Java
public static int hash(Object... values) {
    return Arrays.hashCode(values);
}

public static int hashCode(Object a[]) {
    if (a == null)
        return 0;

    int result = 1;

    for (Object element : a)
        result = 31 * result + (element == null ? 0 : element.hashCode());

    return result;
}
```

可以看到，实现原理还是`Arrays`类的静态方法`hashCode`，接受一个对象数组，使用每一个数组对象本身的哈希值，使用特定的规则来创建一个总的哈希值。这个规则是：`a[0] * 31^ (n - 1) + a[1] * 31^ (n-2) + ... + a[n - 1] * 31`，诶是不是有点熟悉？没错`String`类就是同样的规则来构建自己的哈希值的，它的对象数组就是字符数组。



### toString方法
得到对象的String结果。重载了两种方法。

#### public static String toString(Object o){...}
这个比较简单，返回的是参数对象的string值。定义如下：
```Java
public static String toString(Object o) {
    return String.valueOf(o);
}

public static String valueOf(Object obj) {
    return (obj == null) ? "null" : obj.toString();
}
```
而要注意的是，对于null值，输出的结果是"null"。

####  public static String toString(Object o, String nullDefault){...}
第二个方法给了一个对于null值，默认的输出参数。当对象为null时候，就输出这个默认的值。定义如下：
```Java
public static String toString(Object o, String nullDefault) {
   return (o != null) ? o.toString() : nullDefault;
}
```
这种设计也很熟悉，想起了`HashMap`的`getOrDefault`方法。

### public static <T> int compare(T a, T b, Comparator<? super T> c){...}
比较两个对象，并且需要指定比较器。定义如下：
```Java
public static <T> int compare(T a, T b, Comparator<? super T> c) {
   return (a == b) ? 0 :  c.compare(a, b);
}
```
很好理解，不多说。

### requireNonNull方法
`requireNonNull` == "要求不是null"，如果参数是null就抛出异常。重载了三个方法。以下两个方法的区别是：第二个参数可以传递抛出的异常的message。
```Java
public static <T> T requireNonNull(T obj) {
    if (obj == null)
        throw new NullPointerException();
    return obj;
}

public static <T> T requireNonNull(T obj, String message) {
    if (obj == null)
        throw new NullPointerException(message);
    return obj;
}
```

另外一个`requireNonNull`方法是JDK1.8新加入的方法，定义如下：
```java
public static <T> T requireNonNull(T obj, Supplier<String> messageSupplier) {
    if (obj == null)
        throw new NullPointerException(messageSupplier.get());
    return obj;
}
```
同样还是抛出空指针异常，与`requireNonNull(Object, String)`方法不同，本方法允许将消息的创建延迟，直到空检查结束之后。虽然在非空例子中这可能会带来性能优势， 但是决定调用本方法时应该小心，创建message supplier的开销低于直接创建字符串消息。　　

### public static boolean isNull(Object obj){...}
判断一个对象是不是null，实现非常简单了不贴了。这个是JDK1.8的方法，被用到了JDK1.8函数式编程包中`java.util.function.Predicate`类的实现。



> 注意这个类提供的所有可用方法都是静态方法，毕竟是工具类么。这些方法有什么作用呢？一方面是在一些方面代替了`Object`对象的一些方法，避免了产生空指针异常的可能。另外它提供的对于null值检查的几种方法，广泛用于参数合法性校验。可以直接用，也可以再进行一层的封装，其成果仍旧是一工具类，比如`StringUtil`。
