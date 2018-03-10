# StringBuilder 和 AbstractStringBuilder

趁着刚学习完String的热乎劲，接着学StringBuilder吧。老规矩先上结构图：

![StringBuilder](http://ovn0i3kdg.bkt.clouddn.com/StringBuilder_structure.png?imageView/2/w/400/q/90)

终于可以截全了（其实没有，还差一个serialVersionUID，不过这个不要紧了）

可以看到除了构造方法之外，其他方法都是覆盖了其父类`AbstractStringBuilder`的方法。所以在学习`StringBuilder`之前还是先学习下`AbstractStringBuilder`抽象类，其结构图如下:
![AbstractStringBuilder_1](http://ovn0i3kdg.bkt.clouddn.com/AbstractStringBuilder_structure_1.png?imageView/2/w/400/q/90)
![AbstractStringBuilder_2](http://ovn0i3kdg.bkt.clouddn.com/AbstractStringBuilder_structure_2.png?imageView/2/w/400/q/90)

## abstract class AbstractStringBuilder implements Appendable, CharSequence
`AbstractStringBuilder`是一个抽象类，实现了`Appendable`接口和`CharSequence`接口。

## private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
定义了能接受的最大字符数目。

## char[] value;
这个是真正用来存储字符的地方，一开始没有分配存储空间。

## int count;
用来计数，计算当前已经存储了多少个字符。

## AbstractStringBuilder(){...} 和 AbstractStringBuilder(int capacity){...}
两个重载的构造方法，前者方法体是空的，后者接受参数指定初始化空间大小，定义如下：
```java
AbstractStringBuilder(int capacity) {
    value = new char[capacity];
}
```
## public int length(){...}
用来返回字符长度，直接返回count值就可以了。
> 才注意到，`value`和`count`都没有修饰符，包内访问。

##  public int capacity() {...}
返回的是空间总长度，定义如下：
```java
public int capacity() {
   return value.length;
}
```
这个时候要搞清楚`count`和`value.length`的区别了，前者是真正存储的字符的个数，后者是分配的总空间大小。


## public void ensureCapacity(int minimumCapacity){...}
这个方法确保容量至少等于指定的最小值。定义如下：
```java
public void ensureCapacity(int minimumCapacity) {
    if (minimumCapacity > 0)
        ensureCapacityInternal(minimumCapacity);
}
```
当参数为正数的时候，调用`ensureCapacityInternal`方法，否则什么也不做就返回。`ensureCapacityInternal`方法定义如下：
```java
private void ensureCapacityInternal(int minimumCapacity) {
    // overflow-conscious code
    if (minimumCapacity - value.length > 0) {
        value = Arrays.copyOf(value,
                newCapacity(minimumCapacity));
    }
}
```
如果这个方法接受的参数是负数，那么会抛出`OutOfMemoryError`错误，当然在现在我们讨论的这个范围内，不会有这个错误。从定义中可以看出，当实际的长度小于参数的时候，会给value重新分配地址，这个调用的`newCapacity`方法定义如下：
```java
private int newCapacity(int minCapacity) {
    // overflow-conscious code
    int newCapacity = (value.length << 1) + 2;
    if (newCapacity - minCapacity < 0) {
        newCapacity = minCapacity;
    }
    return (newCapacity <= 0 || MAX_ARRAY_SIZE - newCapacity < 0)
        ? hugeCapacity(minCapacity)
        : newCapacity;
}
```
从这个方法可以看出，容量的增长呈现1，4，10，22，...这种趋势，乘法和加法一块使用，可以在前期增长的快一些，后期则基本是两倍关系。这里还调用了`hugeCapacity`，定义如下：
```java
private int hugeCapacity(int minCapacity) {
    if (Integer.MAX_VALUE - minCapacity < 0) { // overflow
        throw new OutOfMemoryError();
    }
    return (minCapacity > MAX_ARRAY_SIZE)
        ? minCapacity : MAX_ARRAY_SIZE;
}
```
什么作用？前面说道，value的最大空间大小不能超过MAX_ARRAY_SIZE的大小。

## public void trimToSize() {...}
用于该字符序列的方法尝试减少存储。如果缓冲区大于必要保持其当前的字符序列，那么它可能会调整大小，以成为更有效的空间。
简单来说，就是讲value.length的值变成count，怎么变？拷贝一个副本赋值给value就行，定义如下：
```java
public void trimToSize() {
    if (count < value.length) {
        value = Arrays.copyOf(value, count);
    }
}
```

## public void setLength(int newLength) {...}
设置的字符序列的长度。该序列被改变到一个新的字符序列的参数所指定的长度。定义如下：
```java
public void setLength(int newLength) {
    if (newLength < 0)
        throw new StringIndexOutOfBoundsException(newLength);
    ensureCapacityInternal(newLength);

    if (count < newLength) {
        Arrays.fill(value, count, newLength, '\0');
    }

    count = newLength;
}
```
这里调用了前面介绍的`ensureCapacityInternal`方法，扩充之后，用'\0'填充字符数组，然后将count置为新长度。

## public void getChars(int srcBegin, int srcEnd, char[] dst, int dstBegin){...} 和 public void setCharAt(int index, char ch) {...}
这两个方法和String的就很像了，都是在进行越界检查后把字符数组中的某位返回或置位。方法略。

## append方法
重载太多了，最重要的是下面这种:
```java
public AbstractStringBuilder append(String str) {
  if (str == null)
      return appendNull();
  int len = str.length();
  ensureCapacityInternal(count + len);
  str.getChars(0, len, value, count);
  count += len;
  return this;
}
```
如果传入了null对象的话，调用`appendNull`方法，定义如下：
```java
private AbstractStringBuilder appendNull() {
    int c = count;
    ensureCapacityInternal(c + 4);
    final char[] value = this.value;
    value[c++] = 'n';
    value[c++] = 'u';
    value[c++] = 'l';
    value[c++] = 'l';
    count = c;
    return this;
}
```
嗯？直接在后面加上字符串"null"，我以为会啥也不添加就返回，这是什么骚操作？？？为什么要这样？

> 这个方法中还发现一个考点啊。value用final修饰但是居然可以改变值！不是说final修饰的量不变么？诶这里就对”不变“这个词没有弄明白意思了，final修饰的是对象引用的话，引用的地址不能变，但是没有说这个地址里的内容不能改变啊。那既然这样，为什么还要中间定义一个value的局部变量呢？直接给this.value添加四个字符不就完了？？？ 没有弄明白这个设计的用意。

append重载的方法太多了，套路都是一个：改变长度，保障长度，数组拷贝赋值。不一一列出了。

## public AbstractStringBuilder deleteCharAt(int index){...}
方法用于删除某个位置上的字符，定义如下：
```java
public AbstractStringBuilder deleteCharAt(int index) {
   if ((index < 0) || (index >= count))
       throw new StringIndexOutOfBoundsException(index);
   System.arraycopy(value, index+1, value, index, count-index-1);
   count--;
   return this;
}
```
可以看到`arraycopy`方法的源数组和目的数组都是value本身，厉害了。贪吃蛇式的赋值。

## public AbstractStringBuilder replace(int start, int end, String str) {...}
替换某段位置上的字符，定义如下：
```java
public AbstractStringBuilder replace(int start, int end, String str) {
     if (start < 0)
         throw new StringIndexOutOfBoundsException(start);
     if (start > count)
         throw new StringIndexOutOfBoundsException("start > length()");
     if (start > end)
         throw new StringIndexOutOfBoundsException("start > end");

     if (end > count)
         end = count;
     int len = str.length();
     int newCount = count + len - (end - start);
     ensureCapacityInternal(newCount);

     System.arraycopy(value, end, value, start + len, count - end);
     str.getChars(value, start);
     count = newCount;
     return this;
 }
```
注意这里的`arraycopy`和`getChars`方法配合的很巧妙，比如原来value的值是['h', 'e', 'l', 'l', 'o']，想要在'e'和两个'l'的位置换成"me"，那么就是怎么调用的：
`System.arraycopy(value, 4，value, 1 + 2, 1)`;结果value变成['h', 'e', 'l', 'o', 'o']; 接着执行`str.getChars(value, 1)`,就是讲str复制到value数组中1开始的位置，最后变成['h', 'm', 'e', o, 'o']， 接着count值就变成 5 + 2 - （4 - 1） = 4，只取字符数组前4个就行。这样避免了数组元素的移动，很奇妙。


## public String substring(int start, int end){...}
以字符串形式返回部分字符数组的内容，定义如下：
```java
public String substring(int start, int end) {
    if (start < 0)
        throw new StringIndexOutOfBoundsException(start);
    if (end > count)
        throw new StringIndexOutOfBoundsException(end);
    if (start > end)
        throw new StringIndexOutOfBoundsException(end - start);
    return new String(value, start, end - start);
}
```

## insert方法
往字符数组中插入字符，重载的方法很多，大都是转为String类型后调用下面这个方法的：
```java
public AbstractStringBuilder insert(int offset, String str) {
    if ((offset < 0) || (offset > length()))
        throw new StringIndexOutOfBoundsException(offset);
    if (str == null)
        str = "null";
    int len = str.length();
    ensureCapacityInternal(count + len);
    System.arraycopy(value, offset, value, offset + len, count - offset);
    str.getChars(value, offset);
    count += len;
    return this;
}
```
这里同样学习下`arraycopy`和`getChars`两者配合使用的奇妙之处。

## indexOf 和 lastIndexOf
方法本质上和String中的同名方法一样，不细说了。

##  public AbstractStringBuilder reverse(){...}
这应该是非常有用的方法了：倒置。 实现如下：（不知道那个版本的，反正不是1.8）
```java
public AbstractStringBuilder reverse() {
    //是否含代理字符
    //高代理highSurrogate和低代理lowSurrogate概念请另查询char与Unicode字符
    boolean hasSurrogate = false;
    //定义一个变量表示长度-1
    int n = count - 1;
    //j初始化，长度-2再算术右移一位 j = (count-2)/2
    //偶数长度，遍历一半次数，对调替换
    //奇数长度，遍历一半-1次数，对调替换，中间值不用替换
    for (int j = (n-1) >> 1; j >= 0; --j) {
        char temp = value[j];
        char temp2 = value[n - j];
        //如果无代理
        if (!hasSurrogate) {
            hasSurrogate = (temp >= Character.MIN_SURROGATE && temp <= Character.MAX_SURROGATE)
                || (temp2 >= Character.MIN_SURROGATE && temp2 <= Character.MAX_SURROGATE);
        }
        value[j] = temp2;
        value[n - j] = temp;
    }
    if (hasSurrogate) {
        // Reverse back all valid surrogate pairs
        //反转回所有有效代理对
        //高代理+低代理组合表示一个字符
        for (int i = 0; i < count - 1; i++) {
            char c2 = value[i];
            if (Character.isLowSurrogate(c2)) {
                char c1 = value[i + 1];
                if (Character.isHighSurrogate(c1)) {
                    value[i++] = c1;
                    value[i] = c2;
                }
            }
        }
    }
    return this;
}
```
平常也有经常遇到倒置的问题，从这个源码中希望可以获得启发。

## public abstract String toString();
没有方法体。


## final char[] getValue(){...}
返回字符数组value就行。略。


# StringBuilder

StringBuilder除了构造方法和不常用的两个方法(writeObject和readObject)外，其他方法都是覆盖了`AbstractStringBuilder`，并且实现很简单，多是调用父类方法。这里只介绍下构造方法就行。


## public final class StringBuilder extends AbstractStringBuilder implements java.io.Serializable, CharSequence
还是从类声明开始，类被final修饰，表示不能被继承。类继承了`AbstractStringBuilder`抽象类，实现了`Serializable`接口和`CharSequence`接口。

## 构造方法
`StringBuilder`定义了四种构造方法，
### public StringBuilder() {..}
无参数构造方法，定义如下：
```java
public StringBuilder() {
    super(16);
}
```
无参的`StringBuilder`构造方法默认了初始大小为16。

> 为什么用16这个数字呢？

### public StringBuilder(int capacity){...}
接收指定空间大小初始化，定义如下：
```java
public StringBuilder(int capacity) {
    super(capacity);
}
```

### public StringBuilder(String str) {...}
接收字符串为参数，定义如下：
```java
public StringBuilder(String str) {
    super(str.length() + 16);
    append(str);
}
```
将初始化的空间大小定位字符串的长度再加上16。

### public StringBuilder(CharSequence seq) {...}
接收`CharSequence`对象作为参数，定义与上面的方法类似，不多说了。
