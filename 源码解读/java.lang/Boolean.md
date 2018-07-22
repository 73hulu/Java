# Boolean
Boolean是最简单的数值包装类型，其结果如下：

![Boolean_structure](http://ovn0i3kdg.bkt.clouddn.com/Boolean_structure.png)

包装类最重要的作用就是提供类型转换，从上图可以看到，大部分的属性和方法都是为了这个目的存在的。

## public final class Boolean implements java.io.Serializable, Comparable<Boolean>

首先注意到的是类声明，被`final`修饰，说明该类不可被继承。第二这个类实现了`Serializable`和`Comparable`接口，说明该类可被序列化和比较。

## private static final long serialVersionUID = -3665804199014368530L;

`serialVersionUID`是做什么的呢？前面说到这个类实现了`Serializable`接口，那么这个常量用来进行序列化类的时候使用。具体怎么用，暂时不关心，看看就好，可以参考 http://blog.csdn.net/yuexuanyu/article/details/30035153

## private final boolean value;
这就是包装类的值，没什么好说的，在实例化的时候，将基本类型的boolean值赋值给这个属性就可以了。

##   public static final Class<Boolean> TYPE = (Class<Boolean>) Class.getPrimitiveClass("boolean");

又用到了反射的知识了，基本数据类型和其包装类怎么取得联系呢？就是靠这句话了。

## public static final Boolean FALSE = new Boolean(false);
## public static final Boolean TRUE = new Boolean(true);

这两个量在自动装箱函数`valueOf()`中用到，到时候再说。

## public Boolean(boolean value){..} 和  public Boolean(String s){...}

两个重载的构造函数，可以接受一个boolean型的参数和一个String型的参数，
前者定义如下：
```java
public Boolean(boolean value) {
    this.value = value;
}
```
将布尔值赋值给value属性，初始化就完成了。后者定义如下：
```java
public Boolean(String s) {
    this(parseBoolean(s));
}
```
这里调用了一个`parseBoolean(..)`方法，得到一个boolean值，再调用之前的那个构造函数。`parseBoolean`这个方法的定义如下：
```java
public static boolean parseBoolean(String s) {
    return ((s != null) && s.equalsIgnoreCase("true"));
}
```
可见，字符串为"true"的一些大小写变体时会得到true，其他情况（包括null）将会得到false。

> 可见Boolean类没有无参构造函数，`Boolean b = new Boolean()`这种事编译不通过的。

##  public static boolean parseBoolean(String s) {...}

将String类型转为boolean基本数值类型的，前面讲过，不多说了。

## public boolean booleanValue(){...}

获取对象的基本数值，很简单，直接返回value属性值，定义如下：
```java
public boolean booleanValue() {
    return value;
}
```

##  public static Boolean valueOf(boolean b){...}
##  public static Boolean valueOf(String s){...}
`valueOf`是基本包装类中最重要的方法，涉及到"自动装箱"的概念，即如何由基本数值类型获得其包装类型。诶，new一个对象出来不就行了么？说的没错，但是跟这边将的不是一个概念。我们在使用基本数值包装类的时候，往往并没有那么刻意去new，而多是采取`Boolean b = false;`这种写法，这里没有new，没有调用构造函数啊，怎么就获得到一个对象呢？因为这种写法偷偷调用的`valueOf`方法，得到一个包装类的对象，这就是“自动装箱”。

Boolean定义了两个`valueOf`方法，前者定义如下：
```java
public static Boolean valueOf(boolean b) {
     return (b ? TRUE : FALSE);
 }
```
这里返回的`TRUE`和`FALSE`是前面说到的私有静态常量。如果多次调用`valueOf`方法，只要参数一样，得到的实际上是同一个对象！注意，是**同一个**对象。看下面这段程序：
```java
public class BooleanTest {

    public static void main(String[] args) {
        Boolean b1 = false;
        Boolean b2 = false;
        Boolean b3 = new Boolean(false);
        System.out.println(b1 == b2 );
        System.out.println(b2 == b3);

    }
}
```
注意`==`运算符比较的是数值，当两个参数都是对象引用的时候，比较的是引用的地址，所以上例输出：
```java
true
false
```

还有一个以String类型对象作为参数的`valueOf`方法，定义如下：
```java
public static Boolean valueOf(String s) {
    return parseBoolean(s) ? TRUE : FALSE;
}
```
调用了`parseBoolean`方法，很简单，不再赘述。

## public String toString(){...} 和 public static String toString(boolean b){...}
一个非静态，一个静态，方法体都很简单，前者：
```java
public String toString() {
   return value ? "true" : "false";
}
```
后者：
```java
public static String toString(boolean b) {
   return b ? "true" : "false";
}
```
没什么好说的。

## public boolean equals(Object obj){...}
定义如下：
```java
public boolean equals(Object obj) {
    if (obj instanceof Boolean) {
        return value == ((Boolean)obj).booleanValue();
    }
    return false;
}
```
先比较类型，再比较数值。注意，equals方法没有进行类型转化哦。

## public static int hashCode(boolean value){...} 和 public int hashCode(){...}
一个静态，一个非静态，前者调用后者，前者定义：
```java

@Override
public int hashCode() {
    return Boolean.hashCode(value);
}
```
后者定义：
```java
public static int hashCode(boolean value) {
    return value ? 1231 : 1237;
}
```
可见，`true`的哈希值为1231，`false`的哈希值为1237。



## public static boolean getBoolean(String name){...}
这里的name是某个系统属性的名称，定义如下：
```java
public static boolean getBoolean(String name) {
    boolean result = false;
    try {
        result = parseBoolean(System.getProperty(name));
    } catch (IllegalArgumentException | NullPointerException e) {
    }
    return result;
}
```
如果没有叫叫这个名字的系统属性，或者值为空，返回false。

##  public int compareTo(Boolean b){..}
两个Boolean对象的比较，实际上比较的是两个对象的值，定义如下:
```java
public int compareTo(Boolean b) {
    return compare(this.value, b.value);
}
```
这里调用的`compare`是一个静态方法，定义如下：
```java
public static int compare(boolean x, boolean y) {
    return (x == y) ? 0 : (x ? 1 : -1);
}
```
x.compareTo(y)，如果两者相等，返回0。否则当x不为0的时候返回1，为0的时候返回-1。

## public static int compare(boolean x, boolean y){...}
上面将多了，不多说。

## public static boolean logicalAnd(boolean a, boolean b){...} 和 public static boolean logicalOr(boolean a, boolean b){...} 和 public static boolean logicalXor(boolean a, boolean b){...}
逻辑位运算，分别返回`x & y`， `x | y`， `x ^ y`。
