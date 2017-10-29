# Enum
这是所有Java语言枚举类型的公共基类。 有关枚举的更多信息，包括由编译器合成的隐式声明方法的描述，可以在Java™语言规范的第8.9节中找到。
请注意，当使用枚举类型作为集合的类型或作为映射中的键的类型时，可以使用专门且高效的`java.util.EnumSet`集合和`java.util.EnumMap`映射实现。

结构如下：

![Enum](http://ovn0i3kdg.bkt.clouddn.com/Enum.png)


### public abstract class Enum<E extends Enum<E>> implements Comparable<E>, Serializable
类声明，注意到这是一个抽象类，实现了`Comparable`和`Serializable`接口。枚举类型符合通用模式 Class Enum<E extends Enum<E>>，而E表示枚举类型的名称。

### protected Enum(String name, int ordinal){...}
这个类的构造函数，指定了name和ordinal，`name`表示这个类的名称，`ordinal`表示序数。定义如下：
```java
protected Enum(String name, int ordinal) {
    this.name = name;
    this.ordinal = ordinal;
}
```
然而我们很少直接new一个枚举类的对象，而是经常采取下面这种写法：
```java
public enum EnumTest {
    MON, TUE, WED, THU, FRI, SAT, SUN;
}
```
创建枚举类型要使用`enum`关键字，隐含了所创建的类型都是`java.lang.Enum`类的子类（java.lang.Enum 是一个抽象类）。枚举类型的每一个值都将映射到`protected Enum(String name, int ordinal)` 构造函数中，在这里，每个值的名称都被转换成一个字符串，并且序数设置表示了此设置被创建的顺序。

所以上面这段话调用了7次构造函数，等同于
```java
new Enum<EnumTest>("MON",0);
new Enum<EnumTest>("TUE",1);
new Enum<EnumTest>("WED",2);
```

### public final String name(){...}
取得枚举对象的名称，定义如下：
```java
public final String name() {
    return name;
}
```
### public final int ordinal() {...}
取得枚举对象的序数。
```java
public final int ordinal() {
    return ordinal;
}
```

### public String toString(){...}
取得枚举对象的string值，这里返回的是name字段。

### public final boolean equals(Object other){...}
判断两个枚举对象是不是相等的。定义如下：
```java
public final boolean equals(Object other) {
    return this==other;
}
```
可以看到，内部直接使用了`==`来判断，所以枚举型的相等判断，用`equals`方法和`==`符号是一样的效果。

### public final int hashCode(){...}
重写equals方法一定要重写hashCode方法！实际上并没有做什么改变，返回的父类Object的hashCode方法。定义如下：
```java
public final int hashCode() {
    return super.hashCode();
}
```

### protected final Object clone() throws CloneNotSupportedException{...}
继承自父类的克隆方法。`Enum`不允许克隆。定义如下：
```java
protected final Object clone() throws CloneNotSupportedException {
    throw new CloneNotSupportedException();
}
```
没得商量，直接抛出异常。

### public final int compareTo(E o){...}
比较两个枚举对象的值。定义如下：
```java
public final int compareTo(E o) {
    Enum<?> other = (Enum<?>)o;
    Enum<E> self = this;
    if (self.getClass() != other.getClass() && // optimization
        self.getDeclaringClass() != other.getDeclaringClass())
        throw new ClassCastException();
    return self.ordinal - other.ordinal;
}
```
可以看到，在两个枚举对象同类型的情况下，返回两个对象序号的差值（正数/0/负数）。

### public final Class<E> getDeclaringClass(){...}
返回与此枚举常量的枚举类型相对应的 Class 对象。定义如下：
```java
@SuppressWarnings("unchecked")
public final Class<E> getDeclaringClass() {
   Class<?> clazz = getClass();
   Class<?> zuper = clazz.getSuperclass();
   return (zuper == Enum.class) ? (Class<E>)clazz : (Class<E>)zuper;
}
```
### public static <T extends Enum<T>> T valueOf(Class<T> enumType, String name)
返回带指定名称的指定枚举类型的枚举常量。定义如下：
```java
public static <T extends Enum<T>> T valueOf(Class<T> enumType,
                                               String name) {
   T result = enumType.enumConstantDirectory().get(name);
   if (result != null)
       return result;
   if (name == null)
       throw new NullPointerException("Name is null");
   throw new IllegalArgumentException(
       "No enum constant " + enumType.getCanonicalName() + "." + name);
}
```
这个方法就常常用到了， 比如我们要得到枚举名称为`MON`的枚举常量，就可以这么做： `EnumTest MON = Enum.valueOf(EnumTest.class, "MON")`。


下面是一段`Enum`类中常用方法的实例程序：
```java

public class Test {
    public static void main(String[] args) {
        EnumTest test = EnumTest.TUE;

        //compareTo(E o)
        switch (test.compareTo(EnumTest.MON)) {
        case -1:
            System.out.println("TUE 在 MON 之前");
            break;
        case 1:
            System.out.println("TUE 在 MON 之后");
            break;
        default:
            System.out.println("TUE 与 MON 在同一位置");
            break;
        }

        //getDeclaringClass()
        System.out.println("getDeclaringClass(): " + test.getDeclaringClass().getName());

        //name() 和  toString()
        System.out.println("name(): " + test.name());
        System.out.println("toString(): " + test.toString());

        //ordinal()， 返回值是从 0 开始
        System.out.println("ordinal(): " + test.ordinal());
    }
}
```
指定结果是：
```java

TUE 在 MON 之后
getDeclaringClass(): com.hmw.test.EnumTest
name(): TUE
toString(): TUE
ordinal(): 1
```

### 枚举对象自定义属性和方法
```java
public enum EnumTest {
    MON(1), TUE(2), WED(3), THU(4), FRI(5), SAT(6) {
        @Override
        public boolean isRest() {
            return true;
        }
    },
    SUN(0) {
        @Override
        public boolean isRest() {
            return true;
        }
    };

    private int value;

    private EnumTest(int value) {
        this.value = value;
    }

    public int getValue() {
        return value;
    }

    public boolean isRest() {
        return false;
    }

    public static void main(String[] args) {
        System.out.println("EnumTest.FRI 的 value = " + EnumTest.FRI.getValue());
        System.out.println("EnumTest.SAT 的isSet方法" + EnumTest.SAT.isRest());
    }
}
```
执行结果是：
```java
EnumTest.FRI 的 value = 5
EnumTest.SAT 的isSet方法true
```


### 枚举类的原理分析
enum 的语法结构尽管和 class 的语法不一样，但是经过编译器编译之后产生的是一个class文件。该class文件经过反编译可以看到实际上是生成了一个类，该类继承了`java.lang.Enum<E>。EnumTest`经过反编译(`javap com.hmw.test.EnumTest`命令)之后得到的内容如下：
```java
public class com.hmw.test.EnumTest extends java.lang.Enum{
    public static final com.hmw.test.EnumTest MON;
    public static final com.hmw.test.EnumTest TUE;
    public static final com.hmw.test.EnumTest WED;
    public static final com.hmw.test.EnumTest THU;
    public static final com.hmw.test.EnumTest FRI;
    public static final com.hmw.test.EnumTest SAT;
    public static final com.hmw.test.EnumTest SUN;
    static {};
    public int getValue();
    public boolean isRest();
    public static com.hmw.test.EnumTest[] values();
    public static com.hmw.test.EnumTest valueOf(java.lang.String);
    com.hmw.test.EnumTest(java.lang.String, int, int, com.hmw.test.EnumTest);
}
```
所以，实际上 enum 就是一个 class，只不过 java 编译器帮我们做了语法的解析和编译而已。所以可以把enum看做是一个普通的class，它们都可以定义一些属性和方法，不同的是：enem关键字不能继承其他类，因为enum已经继承了java.lang.Enum（Java是单继承关系）。

> EnumSet和EnumMap的用法等待学习java.util包的时候再学习。

参考
* [java enum(枚举)使用详解 + 总结](http://www.cnblogs.com/hemingwang0902/archive/2011/12/29/2306263.html#title-1)
* [Java 枚举(enum) 详解7种常见的用法](http://blog.csdn.net/qq_27093465/article/details/52180865)
