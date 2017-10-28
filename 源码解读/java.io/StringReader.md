# StringReader

针对源是字符串的字符输入流。是`Reader`的直接子类。

![StringReader](http://ovn0i3kdg.bkt.clouddn.com/StringReader.png)

### public StringReader(String s){...}
构造函数，定义如下：
```java
public StringReader(String s) {
    this.str = s;
    this.length = s.length();
}
```
其中`str`和`length`是该类的两个私有属性，`str`就是要读取的字符串，`length`表示字符串的长度。


### private void ensureOpen() throws IOException{...}
保证流未关闭。
```java
private void ensureOpen() throws IOException {
   if (str == null)
       throw new IOException("Stream closed");
}
```
这里流没有关闭的意思是str不为null。

### public int read() throws IOException{..}
读取单个字符。本质就是取出字符串中特定位置上的字符。
```java
public int read() throws IOException {
    synchronized (lock) {
        ensureOpen();
        if (next >= length)
            return -1;
        return str.charAt(next++);
    }
}
```
其中，`next`是下一次要读字符的位置，初始化为0： ` private int next = 0;`。

###  public int read(char cbuf[], int off, int len) throws IOException{...}
重写的抽象函数。把字符写入到字符数组中的特定位置。
```java
public int read(char cbuf[], int off, int len) throws IOException {
   synchronized (lock) {
       ensureOpen();
       if ((off < 0) || (off > cbuf.length) || (len < 0) ||
           ((off + len) > cbuf.length) || ((off + len) < 0)) {
           throw new IndexOutOfBoundsException();
       } else if (len == 0) {
           return 0;
       }
       if (next >= length)
           return -1;
       int n = Math.min(length - next, len);
       str.getChars(next, next + n, cbuf, off);
       next += n;
       return n;
   }
}
```
在各种越界检查之后，将能够取得的字符赋值到cbuf数组。


###  public long skip(long ns) throws IOException{...}
跳过若干个字符。特别注意的是，这个参数ns可能是负数。
```java
public long skip(long ns) throws IOException {
    synchronized (lock) {
        ensureOpen();
        if (next >= length)
            return 0;
        // Bound skip by beginning and end of the source
        long n = Math.min(length - next, ns);
        n = Math.max(-next, n);
        next += n;
        return n;
    }
}
```
方法中考虑到负数的处理，取得n值的过程很玄妙，学着点。


### public boolean ready() throws IOException{...}
流是否已经准备好。定义如下:
```java
public boolean ready() throws IOException {
   synchronized (lock) {
   ensureOpen();
   return true;
   }
}
```

### public boolean markSupported(){...}
判断流是否可以标记，对于该类来说，总是返回true。

### public void mark(int readAheadLimit) throws IOException{...}
```java
public void mark(int readAheadLimit) throws IOException {
   if (readAheadLimit < 0){
       throw new IllegalArgumentException("Read-ahead limit < 0");
   }
   synchronized (lock) {
       ensureOpen();
       mark = next;
   }
}
```
把当前next的值设置为mark的值。

### public void reset() throws IOException{...}
回退流，这个简单，把当前next的位置设置为mark的当前即可。

### public void close(){...}
关闭流，把源str设置为null即可。

> 学到这个类的时候才对流有了一点认识。流是一个抽象的概念，其实并没有那么高深。对于`StringReader`，实际上就是读一个长的字符串，在上面套接一个`BufferedReader`，就能一段一段地读，就是这么个道理。
