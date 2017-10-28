# PipedWriter
管道字符输出流、用于将当前线程的指定字符写入到与此线程对应的管道字符输入流中去、所以`PipedReader（pr）`、`PipedWriter（pw）`必须配套使用、缺一不可。

管道字符输出流的本质就是调用pr中的方法将字符或者字符数组写入到pr中、这一点是与众不同的地方。所以pw中的方法很少也很简单、主要就是负责将传入的pr与本身绑定、配对使用、然后就是调用绑定的pr的写入方法、将字符或者字符数组写入到pr的缓存字符数组中。`PipedWriter`的结构如下：

![PipedWriter](http://ovn0i3kdg.bkt.clouddn.com/PipedWriter.png)

可以看到，`PipedWriter`的结构明显比`PipedReader`，两者的方法并不是对称的，那是因为每次`PipedWriter`写数据的都是调用与之连接的`PipedReader`的`receive`方法，并且写入的地方是`PipedReader`中定义的字符数组。所以在下面的很多方法中可以看到，`PipedWriter`的实现大部分都是在调用`PipedReader`中的方法。

### public synchronized void connect(PipedReader snk) throws IOException{...}
该方法用来连接一个字符输入流。这个输入流必须是未连接的，否则的话会抛出`IOException`异常，如果当前输出流已经由输入流关闭，那么不能建立连接。
```Java
public synchronized void connect(PipedReader snk) throws IOException {
   if (snk == null) {
       throw new NullPointerException();
   } else if (sink != null || snk.connected) {
       throw new IOException("Already connected");
   } else if (snk.closedByReader || closed) {
       throw new IOException("Pipe closed");
   }

   sink = snk;
   snk.in = -1;
   snk.out = 0;
   snk.connected = true;
}
```
保障了建立连接的前提之后，需要设置几个量。注意到`PipedReader`的属性`in`表示下一次要写入的位置，`out`表示下一次要读的位置。

> 可以将管道看成这样的一个形式，一个“管道”实际上是一个特定长度的字符数组，而这个数组实际上从属于`PipedReader`。数组从0到n，`PipedWriter`从-1端开始写，`PipedReader`从0开始开始读。咦，这不就是队列吗？

### write方法
虽然“写”这个操作是`PipedWriter`主导的，但是实际上真正操作的一方是`PipedReader`。重载了2个方法。至于，由于`receive`方法是线程安全的，所以`write`方法也是线程安全的。
#### public void write(int c)  throws IOException{...}
向`PipedReader`中的buf写入一个字符。
```java
public void write(int c)  throws IOException {
   if (sink == null) {
       throw new IOException("Pipe not connected");
   }
   sink.receive(c);
}
```
#### public void write(char cbuf[], int off, int len) throws IOException {...}
向`PipedReader`中的buf写入cbuf数组中的一部分数据
```java
public void write(char cbuf[], int off, int len) throws IOException {
    if (sink == null) {
        throw new IOException("Pipe not connected");
    } else if ((off | len | (off + len) | (cbuf.length - (off + len))) < 0) {
        throw new IndexOutOfBoundsException();
    }
    sink.receive(cbuf, off, len);
}
```

### public synchronized void flush() throws IOException{...}
刷新此流并强制写入任何缓冲输出的字符。并且会唤醒在`PipedReader`对象上等待的所有线程。
```java
public synchronized void flush() throws IOException {
   if (sink != null) {
       if (sink.closedByReader || closed) {
           throw new IOException("Pipe closed");
       }
       synchronized (sink) {
           sink.notifyAll();
       }
   }
}
```

### public void close()  throws IOException{...}
关闭此流。
```java
public void close()  throws IOException {
    closed = true;
    if (sink != null) {
        sink.receivedLast();
    }
}
```
