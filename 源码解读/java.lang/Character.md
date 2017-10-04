# Character
学习这个类的时候真的好想死一死，因为列表真的好长啊，哎：

![Character_structure1](http://ovn0i3kdg.bkt.clouddn.com/1.png?imageView/2/w/400/q/90)
![Character_structure2](http://ovn0i3kdg.bkt.clouddn.com/2.png?imageView/2/w/400/q/90)
![Character_structure3](http://ovn0i3kdg.bkt.clouddn.com/3.png?imageView/2/w/400/q/10)
![Character_structure4](http://ovn0i3kdg.bkt.clouddn.com/4.png?imageView/2/w/400/q/40)
![Character_structure5](http://ovn0i3kdg.bkt.clouddn.com/5.png?imageView/2/w/400/q/40)
![Character_structure6](http://ovn0i3kdg.bkt.clouddn.com/6.png?imageView/2/w/400/q/40)

只看一些常用的。

### 常量
```java
public static final int MIN_RADIX = 2;
public static final int MAX_RADIX = 36;
public static final char MIN_VALUE = '\u0000';
public static final char MAX_VALUE = '\uFFFF';
```
前两个量常常在之前的几个数值包装类中见过，常用来限定进制的范围，不小于2不大于36的2的倍数。后面两个就是`Character`能表示的范围，可以看出`Character`占16位。


### public static Character valueOf(char c) {...}
Character同样采取了缓存的自动装箱方式，定义如下：
```java
public static Character valueOf(char c) {
    if (c <= 127) { // must cache
        return CharacterCache.cache[(int)c];
    }
    return new Character(c);
}
```
其中静态内部缓存类的定义如下：
```java
private static class CharacterCache {
    private CharacterCache(){}

    static final Character cache[] = new Character[127 + 1];

    static {
        for (int i = 0; i < cache.length; i++)
            cache[i] = new Character((char)i);
    }
}
```
可见，缓存的范围是[0,127]，超出这个范围时，返回一个新new的对象。

### public static int hashCode(char value){...}
```java
public static int hashCode(char value) {
    return (int)value;
}
```
`Character`的`hashCode`也很简单粗暴，直接返回其value的int值。

### public String toString(){...}
```java
public String toString() {
    char buf[] = {value};
    return String.valueOf(buf);
}
```
这个初始化了一个字符数字，然后用String类的valueOf方法返回字符串。


### 几个常用方法
剩下的方法太多也不常用，我这里就偷懒先不看了，下面列出的是几个常用的方法，在脑子里有个印象，知道有这么个事就行了。

* public static boolean isUpperCase(char ch)：判断给定的字符是否是大写字符
* public static boolean isLowerCase(char ch)：判断给定的字符是否是小写字符
* public static boolean isDigit(char ch)：判断给定的字符是否是数字字符
* public static char toUpperCase(char ch)：把给定的字符转换为大写字符   
* public static char toLowerCase(char ch)：把给定的字符转换成小写字符
