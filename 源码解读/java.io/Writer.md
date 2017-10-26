# Writer

`Writer`是字符输出流的基础。结构如下：

![Writer](http://ovn0i3kdg.bkt.clouddn.com/Writer.png)

可以看到方法和`Reader`基本上是对应的。同样是一个抽象类，子类只用实现`write(char[], int, int)`和`flush()`和`close()`方法即可。但是一般子类为了有更好的实现，会重新覆盖其他方法，并加入自己的实现。


### public abstract class Writer implements Appendable, Closeable, Flushable
类声明，抽象类，实现了`Appendable`、`Closeable`和`Flushable`接口。

### 构造方法
与`Writer`一样重载了两个构造方法，无参方法将自身作为同步对象，有参构造方法指定同步对象。


### write方法
重载了5个write方法。
#### abstract public void write(char cbuf[], int off, int len) throws IOException;
将字符输出中的内容从off位置开始的，len长度的字符输出到输出流中。

#### public void write(String str, int off, int len) throws IOException{...}
将字符串中的字符输出到输出流中，定义如下：
```java
public void write(String str, int off, int len) throws IOException {
   synchronized (lock) {
       char cbuf[];
       if (len <= WRITE_BUFFER_SIZE) {
           if (writeBuffer == null) {
               writeBuffer = new char[WRITE_BUFFER_SIZE];
           }
           cbuf = writeBuffer;
       } else {    // Don't permanently allocate very large buffers.
           cbuf = new char[len];
       }
       str.getChars(off, (off + len), cbuf, 0);
       write(cbuf, 0, len);
   }
}
```
可以看到该方法线程安全，其中`WRITE_BUFFER_SIZE`是Java规定的缓冲区的最大尺寸，默认1024。其中`writeBuffer`是缓冲区。在这个方法里， 对cbuf做了处理，如果要写的字符长度小于1024，就之间将writeBuffer的地址赋值给cbuf而不是重新申请内容空间，如果大于1024，则重新空间。可以回想下`Reader`中`skipBuffer`申请空间的时机。`skipBuffer`只有每次不够用的时候才会重新申请内存。

#### public void write(int c) throws IOException {...}
写入单个字符。
```java
public void write(int c) throws IOException {
   synchronized (lock) {
       if (writeBuffer == null){
           writeBuffer = new char[WRITE_BUFFER_SIZE];
       }
       writeBuffer[0] = (char) c;
       write(writeBuffer, 0, 1);
   }
}
```
#### public void write(char cbuf[]) throws IOException{...}
将整个字符串数组写入输出流。
```java
public void write(char cbuf[]) throws IOException {
       write(cbuf, 0, cbuf.length);
   }
```

#### public void write(String str) throws IOException{...}
将整个字符串写入输出流。
```java
public void write(String str) throws IOException {
    write(str, 0, str.length());
}
```


### append方法
这是在实现`Appendable`接口，重载了3个方法。需要注意的是这个方法的返回值是this，所以这个方法可以用链式的写法。

#### public Writer append(CharSequence csq) throws IOException {...}
将`CharSequence`类对象的值写入输出流。在空指针判断之后调用以string对象为参数的write方法。定义如下：
```java
public Writer append(CharSequence csq) throws IOException {
    if (csq == null)
        write("null");
    else
        write(csq.toString());
    return this;
}
```
#### public Writer append(CharSequence csq, int start, int end) throws IOException{...}
与上面的append很不同的一点是，当csq是空指针的时，方法仍然能够继续，因为此时会给scq赋值为“null”？？？为什么要这样?
```java
public Writer append(CharSequence csq, int start, int end) throws IOException {
    CharSequence cs = (csq == null ? "null" : csq);
    write(cs.subSequence(start, end).toString());
    return this;
}
```

### public Writer append(char c) throws IOException{...}
参数是单个字符。定义如下：
```java
public Writer append(char c) throws IOException {
    write(c);
    return this;
}
```

### abstract public void flush() throws IOException;
刷新流。 如果流已经从缓冲区中的各种write（）方法保存了任何字符，请将它们立即写入到其预期目的地。 然后，如果该目标是另一个字符或字节流，请将其清空。 因此，一个flush（）调用将刷新Writers和OutputStreams链中的所有缓冲区。

如果该流的预期目的地是由底层操作系统（例如文件）提供的抽象，那么刷新流仅保证先前写入流的字节被传递到操作系统进行写入; 它并不保证它们实际上被写入物理设备，如磁盘驱动器。

### abstract public void close() throws IOException;
关闭流。
