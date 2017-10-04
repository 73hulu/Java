# Short
Short类的结构如下：

![short_class](http://ovn0i3kdg.bkt.clouddn.com/short_structure.png)

Short和Byte类大部分都是类似的，这里只挑拣几个不同的地方讲。

### public static Short valueOf(short s){..}
和Byte一样，Short类也采取了缓存，但是稍微有点不一样。valueOf方法的定义如下：
```java
public static Short valueOf(short s) {
   final int offset = 128;
   int sAsInt = s;
   if (sAsInt >= -128 && sAsInt <= 127) { // must cache
       return ShortCache.cache[sAsInt + offset];
   }
   return new Short(s);
}
```
其中内部静态类`ShortCache`的定义如下：

```java
private static class ShortCache {
    private ShortCache(){}

    static final Short cache[] = new Short[-(-128) + 127 + 1];

    static {
        for(int i = 0; i < cache.length; i++)
            cache[i] = new Short((short)(i - 128));
    }
}
```
乍一看好像和Byte没什么不一样，但是就是不一样，因为Short占16位，但是只缓存了[-128,127]，所以在自动装箱的时候就有了区别，例如下面这段程序：
```java
public class ShortTest {
    public static void main(String[] args) {
        Short s1 = 100;
        Short s2 = 100;

        System.out.println(s1 == s2);

        Short s3 = 200;
        Short s4 = 200;

        System.out.println(s3 == s4);


    }
}
```
输出结果如下：
```java
true
false
```
为什么两个输出结果不一样呢？因为s1和s2的值都是100，在[-128, 127]范围内，所以自动装箱返回的其实是`cache`数组中的同一个对象，而s3和s4的值都是200，不在[-128, 127]范围内，每次返回的都是新new出来的对象。

### public static short reverseBytes(short i){..}
这个方法在Byte中没有，用来干什么的呢？字面上翻译就是"位翻转"，就是讲int党委二进制，左边的位与右边的位进行互换，reverse是按位进行互换，reverseBytes是按照byte进行互换，看它的定义：
```java
public static short reverseBytes(short i) {
   return (short) (((i & 0xFF00) >> 8) | (i << 8));
}
```
该方法返回的值是得到2的补码表示指定的字节的顺序颠倒过来的short值。额其实我没看懂移位运算怎么回事，看来要想读懂源码还得恶补位运算，/(ㄒoㄒ)/~~ 位运算真的好烦躁啊

### public xxx xxxValue(short s){...}
还是强制转换，这里注意`byteValue`方法会损失精度，别的方法倒不会。
