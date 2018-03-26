# ServerSocketChannel
`ServerSocketChannel`是一个可以监听性一个先进来的TCP连接的通道，就像标准IO中的`ServerSocket`一样。

例如：
```JAVA
ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();

serverSocketChannel.socket().bind(new InetSocketAddress(9999));

while(true){
    SocketChannel socketChannel =
            serverSocketChannel.accept();

    //do something with socketChannel...
}
```


![ServerSocketChannel](http://ovn0i3kdg.bkt.clouddn.com/ServerSocketChannel.png)

参考
* [NIO编程之ServerSocketChannel用法详解](https://blog.csdn.net/kavu1/article/details/53212178)

我们还是从一些使用示例说起：

## 打开ServerSocketChannel
通过调用`ServerSocketChannel.open`方法来打开：
```JAVA
ServerSocketChannel serverSocketChannel = ServerSocket.open();
```

## 关闭ServerSocketChannel
调用`ServerSocketChannel.close`方法来关闭：
```JAVA
serverSocketChannel.close();
```

## 监听新进来的链接
通过`ServerSocketChannel.accept()`方法监听新进来的链接。当`accept()`方法返回的时候，它返回一个包含新进来的连接的`SocketChannel`，因此，`accept`方法会一直阻塞到有新连接到达。

通常不会仅仅只监听一个连接，所以需要在while循环中调用accept方法，如下：
```java
while(true){
  SocketChannel sockChannel = serverSocketChannel.accept();
  //do something with socketChanel;
}
```
当然中途也可以break。

## 非阻塞模式
ServerSocketChannel可以设置成非阻塞模式。在非阻塞模式下，accept() 方法会立刻返回，如果还没有新进来的连接,返回的将是null。 因此，需要检查返回的SocketChannel是否是null.如：
```JAVA
ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();

serverSocketChannel.socket().bind(new InetSocketAddress(9999));
serverSocketChannel.configureBlocking(false);

while(true){
  SocketChannel socketChanel = serverSocketChannel.accpet();
  if(sockChannel != null){
    //do something with sockChannel
  }
}

```

参考
* [Java NIO系列教程（九） ServerSocketChannel](http://ifeve.com/server-socket-channel/)
* [NIO编程之ServerSocketChannel用法详解](https://blog.csdn.net/kavu1/article/details/53212178)
