# NIO

NIO即New IO，这个库是在JDK1.4中才引入的。NIO和IO有相同的作用和目的，但实现方式不同。在Java API中提供了两套NIO，一套是针对标准输入输出NIO，另一套就是网络编程NIO。

实际上，我们常常听到另外两个词，BIO和AIO。它们有什么区别呢？在分辨这几个概念之前，我们需要回答以下几个问题：
1. 什么是异步和同步？
2. 什么是阻塞和非阻塞？
3. 什么是同步阻塞、什么是同步非阻塞、什么是异步非阻塞？

## 同步和异步、阻塞和非阻塞
这个问题在知乎[怎样理解阻塞非阻塞与同步异步的区别？](https://www.zhihu.com/question/19732473)这里有透彻的解释，读后豁然开朗，将内容总结如下：

有两个角色，调用者和被调用者。

* 同步和异步
同步和异步描述的是一种行为，它关注的是一种**消息通信机制（synchronous communication/ asynchronous communication）** 。
同步是指当调用者发出一个调用之后，在没有得到**调用结果**之前，这个调用就**不返回**，但是一旦**返回**，就得到**返回结果**了。也就是说，调用者一直在**主动等待**调用结果。
异步是指当调用者发出一个调用之后，这个调用**立刻返回**了，但是此时是没有**返回结果的**。那么这个返回结果就没有办法取得了么？不是的，得靠着**被调用者**通过通知、状态来通知调用者，或者通过回调函数来处理这个调用。典型的异步编程模式比如node.js。

* 阻塞和非阻塞
阻塞和非阻塞关系的是程序**在等待调用结果（消息、返回值）时候的状态**。
阻塞只指当调用结果返回之前，当前线程就会被挂起，调用线程只有在得到结果之后才会返回。（？怎么越说越想是同步呢？不一样的，好好体会一下）
非阻塞是指在不能立刻得到结果之前，该调用不会阻塞当前线程。

知乎上有一个例子讲的很明白：
老张爱喝茶，他用一个普通的茶壶烧水。

第一种方式，水在烧，老张在等，等到水烧开了再去干别的事情。我们从老张的角度看，在等待茶烧开的时间里，他自个儿什么都没有做，是“挂起”状态，这就是**阻塞**，并且一直在**主动查询**水烧开的状态，这就是**同步**。所以这方式是**同步阻塞**。

第二种方式，水在烧，但是老张并没有一直在等，他做别的事情去了，但是**时不时**过来看看水烧开了没有。这个过程我们可以看到，老张一直在**定时轮询**水/茶壶（被调用者）的状态，这就是**同步**，但是这段时间他做别的事情也没有耽误，所以这是**非阻塞状态**。所以这种方式是**同步非阻塞**。

第三种方式，老张买了一个会响的水壶，水一开就会响。用了这个水壶，老张仍在等待这个水开，什么都没干，可能放空状态，但是也没去主动查询结果，最后“水开了”这个讯息还是水壶告诉老张的（所以老张什么都不敢光等了，这不是傻么），所以这种方式是**异步阻塞**。

第四种方式，用的还是会响的水壶。老张也学聪明了，水放上去之后就干自己的事儿去了，水开了水壶就会响，中间老张是不会去查询这个水壶的状态的，这样效率就非常高了。这种方式叫做**异步非阻塞**。


同步异步阻塞和非阻塞组合有4种情况，其中同步阻塞和异步非阻塞都非常有道理，问题在“同步非阻塞”和“异步阻塞”有点疑惑。

首先“同步非阻塞”这个能实现呢？即不断去查询状态，但是还在做别的事情？可以的。这就是**协程**。

另外“异步阻塞”，这个方式就有点傻了，感觉有点浪费资源，又存在的必要么？还真的有，Java NIO就是这种存在！

## BIO、NIO和AIO

上文对同步和异步、阻塞和非阻塞进行了辨析，那么BIO、NIO和AIO的具体含义就呼之欲出了：
* 同步阻塞IO —— BIO：即synchronous Blocking IO
  同步并阻塞，服务器实现模式为一个连接一个线程，即客户端有连接请求时服务器端就需要启动一个线程进行处理，如果这个连接不做任何事情会造成不必要的线程开销，当然可以通过线程池机制改善。

* 同步非阻塞IO(Java NIO)—— NIO：即synchronous Non blocking IO
  同步非阻塞，服务器实现模式为一个请求一个线程，即客户端发送的连接请求都会注册到多路复用器上，多路复用器轮询到连接有I/O请求时才启动一个线程进行处理。用户进程也需要时不时的询问IO操作是否就绪，这就要求用户进程不停的去询问。

* 异步阻塞IO —— JAVA NIO
 此种方式下是指应用发起一个IO操作以后，不等待内核IO操作的完成，等内核完成IO操作以后会通知应用程序，这其实就是同步和异步最关键的区别，同步必须等待或者主动的去询问IO是否完成，那么为什么说是阻塞的呢？因为此时是通过select系统调用来完成的，而select函数本身的实现方式是阻塞的，而采用select函数有个好处就是它可以同时监听多个文件句柄（如果从UNP的角度看，select属于同步操作。因为select之后，进程还需要读写数据），从而提高系统的并发性！  

* 异步非阻塞IO——Java AIO（Java NIO2.0）：即Asynchronous non blocking IO
 在此种模式下，用户进程只需要发起一个IO操作然后立即返回，等IO操作真正的完成以后，应用程序会得到IO操作完成的通知，此时用户进程只需要对数据进行处理就好了，不需要进行实际的IO读写操作，因为真正的IO读取或者写入操作已经由内核完成了。 Java中的`AsynchronousServerSocketChannel`。

## Java NIO
### Java NIO与BIO的区别

| IO | NIO |
| :------------- | :------------- |
| 面向流     |面向缓冲      |
|阻塞IO   |  非阻塞IO |
|  无  |  选择器 |

### 面向流和面向缓冲
Java IO和NIO之间第一个最大的区别是，**IO是面向流的，NIO是面向缓冲区的**。 Java IO面向流意味着每次从流中读一个或多个字节，直至读取所有字节，**它们没有被缓存在任何地方**。此外，它**不能前后移动流中的数据**。如果需要前后移动从流中读取的数据，需要先将它缓存到一个缓冲区。 而Java NIO的缓冲导向方法略有不同。数据读取到一个它稍后处理的缓冲区，需要时可在缓冲区中前后移动。这就增加了处理过程中的灵活性。但是，还需要检查是否该缓冲区中包含所有您需要处理的数据。而且，需确保当更多的数据读入缓冲区时，不要覆盖缓冲区里尚未处理的数据。

标准的IO基于**字节流和字符流**进行操作的，而NIO是基于**通道（Channel）和缓冲区（Buffer）** 进行操作，数据总是从通道读取到缓冲区中，或者从缓冲区写入到通道中

### 阻塞和非阻塞
**Java IO的各种流是阻塞的**。这意味着，当一个线程调用read() 或 write()时，该线程被阻塞，直到有一些数据被读取，或数据完全写入。该线程在此期间不能再干任何事情了。
**Java NIO的非阻塞模式**，使一个线程从某通道发送请求读取数据，但是它仅能得到目前可用的数据，如果目前没有数据可用时，就什么都不会获取，而不是保持线程阻塞，所以直至数据变的可以读取之前，该线程可以继续做其他的事情。 非阻塞写也是如此。一个线程请求写入一些数据到某通道，但不需要等待它完全写入，这个线程同时可以去做别的事情。 线程通常将非阻塞IO的空闲时间用于在其它通道上执行IO操作，所以一个单独的线程现在可以管理多个输入和输出通道（channel）。

Asynchronous IO（异步IO）：Java NIO可以让你异步的使用IO，例如：当线程从通道读取数据到缓冲区时，线程还是可以进行其他事情。当数据被写入到缓冲区时，线程可以继续处理它。从缓冲区写入通道也类似。


### 选择器Selector
Java NIO的选择器允许**一个单独的线程来监视多个输入通道**，你可以注册多个通道使用一个选择器，然后使用一个单独的线程来“选择”通道：这些通道里已经有可以处理的输入，或者选择已准备写入的通道。这种选择机制，使得**一个单独的线程很容易来管理多个通道。**

Java NIO引入了选择器的概念，选择器用于监听多个通道的事件（比如：连接打开，数据到达）。因此，单个的线程可以监听多个数据通道。

> NIO将阻塞交给了后台线程执行

### 服务器模式
BIO的服务器实现模式为一个连接一个线程，NIO服务器实现模式为一个请求一个线程;


### Java NIO
![NIO](http://ovn0i3kdg.bkt.clouddn.com/Java%20NIO.jpg)

其中有三个重要的概念：Channel、Buffer和Selector。
channel翻译为“通道”，和标准IO中的"Stream"是差不多一个等级的，区别在于stream是单向的，比如`InputStream`、`OutputStream`，要么只能读，要么只能写。但是`Channel`是双向的，既可以读又可以写。java.nio.Channels包中提供了四种类型的channle，

`FileChannel`提供了对于文件的读写通道，我们经常使用`RandowmAccesFile`或`FileInputStream`的实例方法`getChannel`来获取实例。其中`write`和`read`分别用来表示读和写。

`DatagramChannel`是UDP的的读写通道。我们使用`DatagramChannel`的静态方法`open`来获取实例，`write`和`read`分别用来表示读和写。

`SocketChannel`和`ServerSocketChannel`是TCP的读写通道，前者的`write`和`read`方法用来读和写，后者可以监听新的`SocketChannel`。

需要注意的，**应用程序不能和channel直接相连**, 他们只能从buffer中读取数据，再写入buffer，所以他们之间的关系如下：

![Channel和Buffer](http://img.blog.csdn.net/20160519212332738)

buffer意为“缓冲区”，java.nio中定义了7种基本数据类型（除了BooleanBuffer），这些都是抽象类，真正的实现类有`MappedByteBuffer`、`HeapByteBuffer`、`DirectByteBuffer`等。缓冲区既可以被读，也可以被写，这两种操作的主体都是channel。实现读写的原理是buffer由四个参数控制，mark、position、limit和capacity，有很多方法来控制缓冲区的读写，比如`clear`就是情况缓冲区，等待写入。`flip`就是将写入模式转化成读取模式，`compact`和`clear`类似，但是会将一些剩余的数据复制到0-position的位置。需要注意的是，不管是clear还是compact方法，都没有将数据清除，它们只是调整了指针，下次数据再输入的时候，将会覆盖旧值。

前面说到channel必须和buffer配合使用，下面就是一个`FileChannel`的使用例子：
```java
public class FileChannelTest{
  private String path = "src/nio_test.txt";

  public void readFileViaNIO(){
    try(RandowmAccesFile aFile = new RandowmAccesFile(path);
      FileChannel channel = aFile.getChannel()){
      ByteBuffer buf = ByteBuffer.allocation(1024);

      int byteRead = channel.read(buf);
      while(byteRead != -1){
        //将buf由写入模式转为读取模式
        buf.flip();
        while(buf.hasRemaining()){
          System.out.print((char)buf.get() + " ");
        }
        buf.compact();
        byteRead = channel.read(buf);
      }
    }catch(IOException e){
      e.printStackTrace();
    }
  }
}
```

两个channel之间如何通信呢？用`transferTo`方法。
例如下面这个两个文件拷贝的例子：
```java

public void copyFileViaNIO(String fromPath, String toPath) thrws IOException{
  File from = new File(fromPath);
  File to = new File(toPath);
  if (!form.exist()) {
    throw new IOException("from file is not exist");
  }
  if (to.exist()) {
    to.createFile();
  }

  try(FileChannel in = new FileInputStream(from);
      FileChannel out = new FileInputStream(to);){
    in.transferTo(0, in.size(), out);
  }catch(IOException e){
    e.printStackTrace();
  }
}
```




## Selector
Selector运行单线程处理多个Channel，如果你的应用打开了多个通道，但每个连接的流量都很低，使用Selector就会很方便。例如在一个聊天服务器中。要使用Selector, 得向Selector注册Channel，然后调用它的select()方法。这个方法会一直阻塞到某个注册的通道有事件就绪。一旦这个方法返回，线程就可以处理这些事件，事件的例子有如新的连接进来、数据接收等。


参考
* [Java NIO 与 IO之间的区别](http://blog.csdn.net/evan_man/article/details/50910542)
* [通俗编程——白话NIO之Selector](https://blog.csdn.net/dd864140130/article/details/50299687)
* [NIO编程之ServerSocketChannel用法详解](https://blog.csdn.net/kavu1/article/details/53212178)
