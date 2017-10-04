# Double
浮点型默认为double，所以Double也比Float常用，其结构如下：

![Double_structure](http://ovn0i3kdg.bkt.clouddn.com/Double_structure.png)

和Float没差多少。

### public static Double valueOf(double d){...}
同样没有缓存，每次自动装箱都返回一个新new的对象。

### public static int hashCode(double value){...}
定义如下：
```java
public static int hashCode(double value) {
   long bits = doubleToLongBits(value);
   return (int)(bits ^ (bits >>> 32));
}
```
和Float类似，调用`doubleToLongBits`方法得到一个long类型值，返回让该值的高32位和低32位做OR运算得到哈希值。
