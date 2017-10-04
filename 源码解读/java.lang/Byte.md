# Byte
Byte是最最基本的整型数值包装类，其结构如下：

![byte_class](http://ovn0i3kdg.bkt.clouddn.com/byte_structure.png)

有些内容很好理解，也与Boolean的内容重复了，这里只挑不同的或者比较重要的来讲。

### public final class Byte extends Number implements Comparable<Byte>

类声明被final修饰，表示不能被继承。本身继承了`Number`类，实现了`Comparable`接口。

### 属性变量
````
  public static final byte   MIN_VALUE = -128;
  public static final byte   MAX_VALUE = 127;
  @SuppressWarnings("unchecked")
    public static final Class<Byte>     TYPE = (Class<Byte>) Class.getPrimitiveClass("byte");
   public static final int SIZE = 8;
   public static final int BYTES = SIZE / Byte.SIZE;
   private final byte value;
   private static final long serialVersionUID = -7183698231559129828L;

````
前两句话定义了Byte类型对象表示的范围是[-128,127]。
被注解为对将警告保持静默的这句话是获得该类的原始类，这里用到了反射。之后定义了Byte的大小为8位，1字节。

### public Byte(byte value) {...} 和 public Byte(String s) throws NumberFormatException  {...}
  两种构造方法，分别支持byte型基本数值类型和String类型，后者定义如下：
  ```java
  public Byte(String s) throws NumberFormatException {
       this.value = parseByte(s, 10);
   }
  ```
  这里做了限制，传入的必须是能转为数字的字符串，不满足条件则抛出异常。这里调用了`parseByte`方法，以10进制转换数值，定义如下：
  ```java
  public static byte parseByte(String s, int radix)
     throws NumberFormatException {
     int i = Integer.parseInt(s, radix);
     if (i < MIN_VALUE || i > MAX_VALUE)
         throw new NumberFormatException(
             "Value out of range. Value:\"" + s + "\" Radix:" + radix);
     return (byte)i;
 }
```
先将该数值转化为10进制的int型，如果该值超不在[-128, 127]内，则重新抛出异常。


### public static String toString(byte b){...} 和 public String toString(){..}
前者定义如下：
```java
public static String toString(byte b) {
    return Integer.toString((int)b, 10);
}
```
后者重写从Object继承的toString方法，定义如下：
```java
public String toString() {
    return Integer.toString((int)value);
}

```
它们实则都是在调用Integer的toString方法。

### public static Byte valueOf(byte b) {...} 和   public static Byte valueOf(String s) throws NumberFormatException {..} 和   public static Byte valueOf(String s, int radix) throws NumberFormatException {..}
三种自动装箱方法，第一种组重要，很重要！！！定义如下：
```java
public static Byte valueOf(byte b) {
    final int offset = 128;
    return ByteCache.cache[(int)b + offset];
}
```
从Byte开始，基本整型数值包装类都使用了缓存的方法。这里ByteCache是Byte中的内部静态类，而cache是ByteCache中的静态常量数组，定义如下:
```java
private static class ByteCache {
  private ByteCache(){}

  static final Byte cache[] = new Byte[-(-128) + 127 + 1];

  static {
      for(int i = 0; i < cache.length; i++)
          cache[i] = new Byte((byte)(i - 128));
  }
}
```
当Byte类被加载的时候，这个内部类不会初始化，什么时候初始化呢？在valueOf被调用的时候，这个内才被加载、初始化。可见，如果传入`valueOf`方法的参数相同，最终取到的对象值是**同一个**！


后两者接收字符串作为被转化的量，其中一个默认进制是10， 另一种方法可指定进制，其定义分别如下：
```java
public static Byte valueOf(String s) throws NumberFormatException {
    return valueOf(s, 10);
}
```
```java
public static Byte valueOf(String s, int radix)
   throws NumberFormatException {
   return valueOf(parseByte(s, radix));
}
```

###  public static Byte decode(String nm) throws NumberFormatException{...}
decode 用来分析数字，定义如下：
```java
public static Byte decode(String nm) throws NumberFormatException {
    int i = Integer.decode(nm);
    if (i < MIN_VALUE || i > MAX_VALUE)
        throw new NumberFormatException(
                "Value " + i + " out of range from input " + nm);
    return valueOf((byte)i);
}
```
又是在调用Integer类的静态decode方法得到int型变量后在判断是否在Byte的表示范围内，不再则抛出异常。哎，数值类型Integer还是老大哥啊，很多关键的问题还要靠他来解决。

### public xxx xxxValue(){...}
这一类都是重写了其父类`Number`的方法，进行强制转化后返回相应的值，这一系列有哪些呢？`byteValue()`、`shortValue`、`intValue`、`longValue`、`floatValue`、`doubleValue`，比如`intValue`定义如下：
```java
public long longValue() {
    return (long)value;
}
```
Byte是最基本的数值包装类型，所以这样的强制转化没有什么精度损失。

### public int hashCode{..} 和 public static int hashCode(byte value) {..}
前者调用后者，后者定如下：
```java
public static int hashCode(byte value) {
    return (int)value;
}
```
将其value值转为int型然后返回了，就是这么简单。

### public int compareTo(Byte anotherByte){...} 和 public static int compare(byte x, byte y){..}
前者调用后者，后者定义如下：
```java
public static int compare(byte x, byte y) {
    return x - y;
}
```
这次是真的在比小大了，返回的是主调与参数的差值。

### public static int toUnsignedInt(byte x){...} 和 public static long toUnsignedLong(byte x){...}
从名字就能看出来了，转化为无符号的int型和long型，前者定义如下：
```java
public static int toUnsignedInt(byte x) {
   return ((int) x) & 0xff;
}
```
后者定义如下：
```java
public static long toUnsignedLong(byte x) {
    return ((long) x) & 0xffL;
}
```
用到位与运算。

> & 运算：两个操作数都为1，结果才为1，否则为0。
