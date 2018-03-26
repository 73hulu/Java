# SocketChannel

`SocketChannel`是nio中连接到TCP网络套接字的一个通道。学习这个类可以类比于`DatagramChannel`这个类。

![SocketChannel](http://ovn0i3kdg.bkt.clouddn.com/SocketChannel.png?imageView/2/w/400)


可以使用两种方式创建`SocketChannel`：
1. 打开一个SocketChannel并连接到互联网上的某台服务器。
2. 一个新连接到达ServerSocketChannel时，会创建一个SocketChannel。

下面是一些`SocketChannel`的常用操作：
## 打开`SocketChannel`
```JAVA
SocketChannel sockChannel = SocketChannel.open();

sockChannel.connect(new InetSocketAddress("http://www.baidu.com", 80));
```

## 关闭SocketChannel
当用完关闭
```JAVA
socketChannel.close();
```

## 读取数据
```JAVA
ByteBuffer buf = ByteBuffer.allocation(48);
int byteRead = sockChannel.read(buf);
```

首选分配一个Buffer，从socketChanel中读取的数据将会放到buf中，然后调用`SocketChannel.read()`将数据从`SocketChannel`中读取到buffer中。`read`方法返回了读入的数据的长度。如果是-1，表明已经读到了流的末尾。


## 写入SocketChannel
写数据到SocketChannle用的是`write`方法。如下：
```JAVA
String newData = "This is String to write";

ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
buf.put(newData.getBytes());
buf.flip();

while(buf.hasRemaining()){
  channel.write(buf);
}
```
注意SocketChannel.write()方法的调用是在一个while循环中的。Write()方法无法保证能写多少字节到SocketChannel。所以，我们重复调用write()直到Buffer没有要写的字节为止。

## 非阻塞模式
可以设置 SocketChannel 为非阻塞模式（non-blocking mode）.设置之后，就可以在异步模式下调用connect(), read() 和write()了。

### connect()
如果SocketChannel在非阻塞模式下，此时调用connect()，该方法可能在连接建立之前就返回了。为了确定连接是否建立，可以调用finishConnect()的方法。像这样：
```JAVA
socketChannel.configureBlocking(false);
socketChannel.connect(new InetSocketAddress("http://jenkov.com", 80));

while(! socketChannel.finishConnect() ){
    //wait, or do something else...
}
```
### write()
非阻塞模式下，write()方法在尚未写出任何内容时可能就返回了。所以需要在循环中调用write()。前面已经有例子了，这里就不赘述了。

### read()
非阻塞模式下,read()方法在尚未读取到任何数据时可能就返回了。所以需要关注它的int返回值，它会告诉你读取了多少字节。

### 非阻塞模式与选择器
非阻塞模式与选择器搭配会工作的更好，通过将一或多个SocketChannel注册到Selector，可以询问选择器哪个通道已经准备好了读取，写入等。


参考
* [Java NIO系列教程（八） SocketChannel](http://ifeve.com/socket-channel/)
