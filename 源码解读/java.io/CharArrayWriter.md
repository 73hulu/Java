# CharArrayWriter

与`CharArrayReader`相对应，要把字符存入字符数组中。这个字符数组的大小会随着数据写入自动增长，最后可以使用`toChartArray()`或`toString()`方法来转化为字符串。

![CharArrayWriter](http://ovn0i3kdg.bkt.clouddn.com/CharArrayWriter.png)


### 构造函数
重载了两个构造函数。区别在于是否指定初始化字符数组的大小。

#### public CharArrayWriter(){...}
无参数构造方法，默认缓冲区的初始化大小是32。
```java
public CharArrayWriter() {
    this(32);
}
```

#### public CharArrayWriter(int initialSize){...}
指定了缓冲区的空间大小。
```java
public CharArrayWriter(int initialSize) {
   if (initialSize < 0) {
       throw new IllegalArgumentException("Negative initial size: "
                                          + initialSize);
   }
   buf = new char[initialSize];
}
```

### write方法
用来将一个或多个字符写入字符数组。重载三个方法。线程安全。
#### public void write(int c){...}
```java
public void write(int c) {
   synchronized (lock) {
       int newcount = count + 1;
       if (newcount > buf.length) {
           buf = Arrays.copyOf(buf, Math.max(buf.length << 1, newcount));
       }
       buf[count] = (char)c;
       count = newcount;
   }
}
```
之前提到过，字符数组的大小会随着字符的写入自动增长，怎么实现的呢？从这个方法就可以找到答案了。当空间不足时候，会重新申请一个更大的空间作为缓冲区，那么这个新的空间到底应该多大呢？这里的策略是取当前空间大小的2倍和（当前空间大小 + 1）中取最大值。所以可以得出结论，如果当初未指定初始化的大小，即初始化大小是默认的32，之后每次都写入1个字符，那么空间的大小变化是32，64，128....，2倍2倍增长。

还有一点需要注意，这里的`count`和`CharArrayReader`中的`count`表示的含义不同。在`CharArrayReader`中，count表示的是流能读取的字符区域的末端索引，是下标值，而`CharArrayWriter`中的`count`表示已经写入的字符的个数，也是下一次要写入字符的位置。

#### public void write(char c[], int off, int len){...}
将字符数组c的一部分区域数据写入缓冲区，定义如下：
```java
public void write(char c[], int off, int len) {
    if ((off < 0) || (off > c.length) || (len < 0) ||
        ((off + len) > c.length) || ((off + len) < 0)) {
        throw new IndexOutOfBoundsException();
    } else if (len == 0) {
        return;
    }
    synchronized (lock) {
        int newcount = count + len;
        if (newcount > buf.length) {
            buf = Arrays.copyOf(buf, Math.max(buf.length << 1, newcount));
        }
        System.arraycopy(c, off, buf, count, len);
        count = newcount;
    }
}
```
同样的空间增长策略。

> 源码已经读了很多了，发现在有些类的实现中会采取缓冲区的方式，实际上就是一个内部的私有数组，在这个缓冲区的初始化和空间的使用、增长策略上都不太一样，听哟䬥的，在阅读集合类的时候可以整理一个专题。

### public void write(String str, int off, int len){...}
同样的道理，只是第一个参数是`String`类型，其实`String`的内部实现也是一个字符数组啊。定义如下：
```java
public void write(String str, int off, int len) {
    synchronized (lock) {
        int newcount = count + len;
        if (newcount > buf.length) {
            buf = Arrays.copyOf(buf, Math.max(buf.length << 1, newcount));
        }
        str.getChars(off, off + len, buf, count);
        count = newcount;
    }
}
```

###  public void writeTo(Writer out) throws IOException{...}
把缓冲区的中的内容写到另一个`Writer`实例的缓冲区中。定义如下：
```java
public void writeTo(Writer out) throws IOException {
   synchronized (lock) {
       out.write(buf, 0, count);
   }
}
```

### append方法
追加数据，对于缓冲区来说是效果是一样的，都是往后写，不同的是接受的参数已经返回值，返回本身，所以append方法可以采取链式写法。重载了3个方法。
#### public CharArrayWriter append(CharSequence csq){...}
将整个`CharSequence`对象添加到缓冲区。
```java
public CharArrayWriter append(CharSequence csq) {
   String s = (csq == null ? "null" : csq.toString());
   write(s, 0, s.length());
   return this;
}
```
很有趣的一点是，如果参数为null，那字符串"null"将被追加到缓冲区，这个情况以前在`StringBuffer`（不知道有没有记错）中也出现过，不知道这样的设计意图是什么？

#### public CharArrayWriter append(CharSequence csq, int start, int end){...}
将`CharSequence`对象的一部分添加到缓冲区
```java
public CharArrayWriter append(CharSequence csq, int start, int end) {
   String s = (csq == null ? "null" : csq).subSequence(start, end).toString();
   write(s, 0, s.length());
   return this;
}
```

#### public CharArrayWriter append(char c){...}
向缓冲区中写入单个字符。定义如下：
```java
public CharArrayWriter append(char c) {
    write(c);
    return this;
}
```

####  public void reset(){...}
重置流。定义如下：
```java
public void reset() {
    count = 0;
}
```
很简单，将`count`置为0，虽然缓冲区中仍然有数据未清除，但是下一次再往缓冲区中写入数据的时候会覆盖这部分的内容。


### public char toCharArray()[]{...}
```java
public char toCharArray()[] {
    synchronized (lock) {
        return Arrays.copyOf(buf, count);
    }
}
```
复制一份缓冲区的副本返回。道理我都懂，但是这个方法签名写法有点怪啊，刚开始我咋一看还以为找到了一个编译器都没有检查出来的JDK源码的BUG（事实证明这是不可能的 :) ），一般来说，写成`public char[] toCharArray(){..}`这种比较好理解吧。

#### public int size(){return count;}
返回缓冲区的大小，说的就是`count`的值。

### public String toString(){...}
取得缓冲区的String值，实现超简单：
```java
public String toString() {
    synchronized (lock) {
        return new String(buf, 0, count);
    }
}
```

### public void flush() { }
刷新流。注意这个方法没有方法体！

### public void close() { }
关闭流。！！！ 没有方法体，也就是说调用这个方法后其实没有什么用，源还是存在的，仍旧可以被使用。
