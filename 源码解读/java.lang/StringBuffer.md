# StringBuffer
`StringBuffer`经常和`StringBuilder`混到一起讲，先来看看结构：

![StringBuffer](http://ovn0i3kdg.bkt.clouddn.com/StringBuffer_structure_1.png)
![StringBuffer](http://ovn0i3kdg.bkt.clouddn.com/StringBuffer_structure_2.png)

从结构可以看出来`StringBuffer`和`StringBuilder`确实师承一脉，那么两者的区别在什么地方。

##  public final class StringBuffer extends AbstractStringBuilder implements java.io.Serializable, CharSequence

和`StringBuilder` 一样，继承`AbstractStringBuilder`，实现`Serializable`和`CharSequence`接口。

## 构造函数
三个构造函数，分别接受无参、初始容量和`String`类型参数，默认的初始容量是16。

##  private transient char[] toStringCache;
注意到`StringBuffer`中的这个成员变量，在`StringBuilder`中并没有，那么这个变量的作用是什么？官方解释是：由toString返回的最后一个值得缓存。每当StringBuffer被修改的时候清空。

从哪里体现呢？看看`toString`方法的定义：
```java
@Override
public synchronized String toString() {
   if (toStringCache == null) {
       toStringCache = Arrays.copyOfRange(value, 0, count);
   }
   return new String(toStringCache, true);
}
```
可以看到，在返回字符串之前，先在`toStringCache`中缓存了值。
那么在一些需改`StringBuffer`内容的函数中，比如append、replace、delete等，`toStringCache`都是被置为null，比如：
```java
@Override
public synchronized void setCharAt(int index, char ch) {
    if ((index < 0) || (index >= count))
        throw new StringIndexOutOfBoundsException(index);
    toStringCache = null;
    value[index] = ch;
}
```

## 线程安全

`StringBuffer`和`StringBuilder`最重要的区别在什么地方？常听到这样的回答：

"StringBuffer是线程安全的，而StringBuilder不是线程安全的"

答案正确，那么源码中哪里有体现呢？

虽然StringBuffer和StringBuilder师承一脉，但是StringBuffer大部分对内容进行修改的方法上都有`synchronized`关键词的修饰，当然有些操作并没有`synchronized`修饰，为什么搞特殊？这些方法都有是三行特别的注释，比如：
```java
public StringBuffer insert(int dstOffset, CharSequence s) {
    // Note, synchronization achieved via invocations of other StringBuffer methods
    // after narrowing of s to specific type
    // Ditto for toStringCache clearing
    super.insert(dstOffset, s);
    return this;
}
```
这里解释说这些方法的同步化是通过调用其他`StringBuffer`方法来实现的。但是我看着这源码，调用的是父类的方法，并没有实现同步啊？怎么肥四？？？
