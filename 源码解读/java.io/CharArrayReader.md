# CharArrayReader
这个流针对的媒介是字符数组。结构如下：

![CharArrayReader](http://ovn0i3kdg.bkt.clouddn.com/CharArrayReader.png)

### 构造方法
重载了两个构造方法。

####  public CharArrayReader(char buf[]){...}
参数buf就是指定的源。定义如下
```java
public CharArrayReader(char buf[]) {
    this.buf = buf;
    this.pos = 0;
    this.count = buf.length;
}
```
`buf`、`pos`和`count`是类的受保护的属性，分别是字符缓冲区、当前字符缓冲区的位置和字符缓冲区中最后一个字符所在的索引（注意，这个标识的是下标而不是个数）。

#### public CharArrayReader(char buf[], int offset, int length){...}
参数buf指定了源，但是与上一个构造方法不同的是，虽然最终在字符缓冲区中的字符是一样的，但是pos和count最终会导致这个流真正使用的字符区域不同。定义如下：
```java
public CharArrayReader(char buf[], int offset, int length) {
    if ((offset < 0) || (offset > buf.length) || (length < 0) ||
        ((offset + length) < 0)) {
        throw new IllegalArgumentException();
    }
    this.buf = buf;
    this.pos = offset;
    this.count = Math.min(offset + length, buf.length);
    this.markedPos = offset;
}
```

###  private void ensureOpen() throws IOException{..}
保证流打开状态，只要保证源存在即可，定义如下：
```java
private void ensureOpen() throws IOException {
    if (buf == null)
        throw new IOException("Stream closed");
}
```

### read方法
重载了两种read方法。分别用来读取单个字符和多个字符。线程安全。
#### public int read() throws IOException{...}
用来读取单个字符。
```java
public int read() throws IOException {
    synchronized (lock) {
        ensureOpen();
        if (pos >= count)
            return -1;
        else
            return buf[pos++];
    }
}
```
#### public int read(char b[], int off, int len) throws IOException{...}
取得源中的一部分字符到字符数组b中的特定区域。定义如下：
```java
public int read(char b[], int off, int len) throws IOException {
   synchronized (lock) {
       ensureOpen();
       if ((off < 0) || (off > b.length) || (len < 0) ||
           ((off + len) > b.length) || ((off + len) < 0)) {
           throw new IndexOutOfBoundsException();
       } else if (len == 0) {
           return 0;
       }

       if (pos >= count) {
           return -1;
       }
       if (pos + len > count) {
           len = count - pos;
       }
       if (len <= 0) {
           return 0;
       }
       System.arraycopy(buf, pos, b, off, len);
       pos += len;
       return len;
   }
}
```
实现过程很简单。

###  public long skip(long n) throws IOException{...}
跳过若干个字符。返回实际跳过的字符个数。
```java
public long skip(long n) throws IOException {
   synchronized (lock) {
       ensureOpen();
       if (pos + n > count) {
           n = count - pos;
       }
       if (n < 0) {
           return 0;
       }
       pos += n;
       return n;
   }
}
```

### public boolean ready() throws IOException{...}
流是否可读。定义如下：
```java
public boolean ready() throws IOException {
    synchronized (lock) {
        ensureOpen();
        return (count - pos) > 0;
    }
}
```
首先保证了流的源未关闭，然后需要还有内容可读。

### public boolean markSupported(){...}
是否支持可标记，永远返回`true`。

### public void mark(int readAheadLimit) throws IOException{...}
标记，这个方法重写了父类的方法
```java
public void mark(int readAheadLimit) throws IOException {
    synchronized (lock) {
        ensureOpen();
        markedPos = pos;
    }
}
```
并没有用到参数`readAheadLimit`，标记的过程就是将当前读的位置记录下来。

### public void reset() throws IOException{...}
重置流，很简单，有之前标记过的值，把`pos`值重新设置成被标记的值即可。
```java
public void reset() throws IOException {
    synchronized (lock) {
        ensureOpen();
        pos = markedPos;
    }
}
```

### public void close(){...}
关闭流，就是将源设置为null即可。
```java
public void close() {
    buf = null;
}
```
