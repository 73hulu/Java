# Reader

Reader是字符输入流的基础类，结构如下

![Reader](http://ovn0i3kdg.bkt.clouddn.com/Reader.png)

注意到这里只有两个抽象方法`read(char[], int, int)`和`close()`，一般子类只用重写这两个方法即可，但是一般地，子类会重写覆盖其他的一些非抽象方法以得到更好的性能。


## public abstract class Reader implements Readable, Closeable
类声明，注意这是一个抽象类，实现了`Readable`和`Closeable`接口。

## 构造方法
`Reader`类有两个构造方法，全部都是protected。
### protected Reader(){...}
无参数构造方法，定义如下：
```java
protected Reader() {
    this.lock = this;
}
```
这类的lock是一个protected的成员变量，定义如下：
```java
protected Object lock;
```
这个量有什么作用呢？从名字就猜出来了，锁，同步用的。在无参数的构造方法中，这个同步对象指定为自己。能不能指定为别人呢？可以，用另一个有参数的构造方法就行。

### protected Reader(Object lock){...}
这个构造方法使用一个Object实例作为参数，这个实例就是锁。定义如下：
```java
protected Reader(Object lock) {
    if (lock == null) {
        throw new NullPointerException();
    }
    this.lock = lock;
}
```

## read方法
这是Reader中最核心的方法了，一共重载了4个。

### public int read(java.nio.CharBuffer target) throws IOException{...}
该方法尝试将字符读入指定的字符缓冲区。缓冲区用作字符存储库：唯一的更改是put操作的结果。不执行缓冲器的翻转或倒带。
```java
public int read(java.nio.CharBuffer target) throws IOException {
    int len = target.remaining();
    char[] cbuf = new char[len];
    int n = read(cbuf, 0, len);
    if (n > 0)
        target.put(cbuf, 0, n);
    return n;
}
```
这类的参数target就是字符缓冲区，这个类继承`Buffer`抽象类。`target.remaining()`方法得到了缓冲区剩余的大小，其定义如下：
```java
public final int remaining() {
    return limit - position;
}
```
用此长度构造一个字符数组，调用的方法是一个抽象方法`abstract public int read(char cbuf[], int off, int len) throws IOException;`这个方法需要子类去实现，该方法返回的是字符的个数，如果已经达到流的结尾，则返回-1。如果在读的过程中发生错误，则抛出`IOException`异常。如果读取成功，target就把这个字符串数组加入到字符缓冲区，最后返回的是字符的个数。这里有一个疑问，为什么不直接往字符缓冲区中写呢？是怕中间产生读写错误么？

### public int read() throws IOException{..}
无参数的read方法，读取的是单个字符。这个在字符不可用的时候将会阻塞，知道字符可用或者发生IO错误或者到达字符流的结尾。这里面仍旧调用了上面的read方法。方法返回值是-1（已经到达流结尾时）或者这个字符的int值。
```java
public int read() throws IOException {
    char cb[] = new char[1];
    if (read(cb, 0, 1) == -1)
        return -1;
    else
        return cb[0];
}
```
### public int read(char cbuf[]) throws IOException{...}
将字符读取到字符串数组，定义如下：
```java
public int read(char cbuf[]) throws IOException {
   return read(cbuf, 0, cbuf.length);
}
```

### abstract public int read(char cbuf[], int off, int len) throws IOException;
首先方法，子类必须实现。也是其他几种重载的read方法的核心。

## public long skip(long n) throws IOException{...}
方法用来跳过某些字符。参数n就是被跳过的字符的个数。在某个字符可用、发生 I/O 错误或者已到达流的末尾前，此方法一直阻塞。定义如下：
```java
public long skip(long n) throws IOException {
    if (n < 0L)
        throw new IllegalArgumentException("skip value is negative");
    int nn = (int) Math.min(n, maxSkipBufferSize);
    synchronized (lock) {
        if ((skipBuffer == null) || (skipBuffer.length < nn))
            skipBuffer = new char[nn];
        long r = n;
        while (r > 0) {
            int nc = read(skipBuffer, 0, (int)Math.min(r, nn));
            if (nc == -1)
                break;
            r -= nc;
        }
        return n - r;
    }
}
```
这里有一个变量`maxSkipBufferSize`，这是Java定义的能跳过的最大字符个数，定义为`private static final int maxSkipBufferSize = 8192;`其中`skipBuffer`是一个字符数组类型的成员变量，用来保存被跳过的字符。最后返回的是实际上跳过的字符个数。另外注意到跳过的这个过程是线程安全的。

> 这里面的数量关系还挺绕的。比如实际上只有3个字符，但是参数n为5。那么nn=5，r=5, 执行返回的nc=3，r=5-3=2。下一循环时候，nc=-1退出循环，返回的值是5-2=3。即实际上跳过的字符个数是3个。

## public boolean ready() throws IOException{...}
判断是否准备读取此流。返回值：如果保证下一个 read() 不阻塞输入，则返回 True，否则返回 false。注意，返回 false 并不保证阻塞下一次读取。定义如下：
```java
public boolean ready() throws IOException {
    return false;
}
```
注意到，看这个方法只会返回false。也就是说并不保证阻塞下一次读取？？？？不是很理解这个方法设计意图。应该是让子类重写的吧。

## public boolean markSupported(){...}
告诉这个流是否支持`mark()`操作。 默认实现始终返回false。子类应该覆盖此方法。 什么是mark操作？


## public void mark(int readAheadLimit) throws IOException{...}
标记流中的当前位置。 对reset（）的后续调用将尝试将流重新定位到此位置。 并非所有字符输入流都支持mark（）操作。
```java
public void mark(int readAheadLimit) throws IOException {
   throw new IOException("mark() not supported");
}
```

## public void reset() throws IOException{...}
重置流。 如果流已被标记，则尝试在标记处重新定位。 如果流未被标记，则尝试以某种方式将其重置为特定的流，例如通过将其重新定位到其起始点。 并不是所有的字符输入流都支持reset（）操作，有些支持reset（），而不支持mark（）。
```java
public void reset() throws IOException {
   throw new IOException("reset() not supported");
}
```
### abstract public void close() throws IOException;
关闭流并释放与之相关联的任何系统资源。 一旦流已关闭，进一步的read（），ready（），mark（），reset（）或skip（）调用将抛出一个IOException。 关闭以前关闭的流无效。
