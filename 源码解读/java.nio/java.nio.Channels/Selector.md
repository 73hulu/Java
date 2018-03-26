# Selector

Selector（选择器）是Java NIO中能够检测一到多个NIO通道，并能够知晓通道是否为诸如读写事件做好准备的组件。这样，一个单独的线程可以管理多个channel，从而管理多个网络连接。


![Selector](http://ovn0i3kdg.bkt.clouddn.com/Selector.png?imageView/2/w/400)

## 为什么要用Selector
Selector提供了询问通道是否已经准备好执行每个I/O操作的能力。Selector 允许单线程处理多个Channel。仅用单个线程来处理多个Channels的好处是，只需要更少的线程来处理通道。事实上，可以只用一个线程处理所有的通道，这样会大量的减少线程之间上下文切换的开销。

其中有三个重要的概念：
* 选择器（Selector）
Selector选择器类管理着一个被注册的通道集合的信息和它们的就绪状态。通道是和选择器一起被注册的，并且使用选择器来更新通道的就绪状态。当这么做的时候，可以选择将被激发的线程挂起，直到有就绪的的通道。

* 可选择通道（SelectableChannel）
SelectableChannel这个抽象类提供了实现通道的可选择性所需要的公共方法。它是所有支持就绪检查的通道类的父类。**因为FileChannel类没有继承SelectableChannel，因此是不是可选通道**，而所有socket通道都是可选择的，包括从管道(Pipe)对象的中获得的通道。SelectableChannel可以被注册到Selector对象上，同时可以指定对那个选择器而言，那种操作是感兴趣的。一个通道可以被注册到多个选择器上，但对每个选择器而言只能被注册一次。

* 选择键（SelectionKey）
选择键封装了特定的通道与特定的选择器的注册关系。选择键对象被SelectableChannel.register() 返回并提供一个表示这种注册关系的标记。选择键包含了两个比特集(以整数的形式进行编码)，指示了该注册关系所关心的通道操作，以及通道已经准备好的操作。

下面是使用Selector管理多个channel的示意图：

![Selector管理多个channel](http://img.blog.csdn.net/20151214194029453)

下面我们仍然从使用的角度看看Seletor的用法。


## 创建Selector
`Selector`这个类是抽象类，不能实例化，我们使用静态工厂方法来创建实例。
```JAVA
public static Selector open() throws IOException {
    return SelectorProvider.provider().openSelector();
}
```

## 将Channel注册到Selector
为了能让Channel被Selector管理，Channel应该被注册到Selector上。

```JAVA
channel.configureBlocking(false);
SelectionKey key= channel.register(selector,SelectionKey,OP_READ);
```

通过调用通道的`register()`方法会将它注册到一个选择器上。与Selector一起使用时，Channel必须处于**非阻塞模式**下，否则将抛出`IllegalBlockingModeException`异常，这意味着不能将**FileChannel与Selector一起使用**，因为FileChannel不能切换到非阻塞模式，而套接字通道都可以。另外**通道一旦被注册，将不能再回到阻塞状态，此时若调用通道的configureBlocking(true)将抛出BlockingModeException异常。**

`register()`方法的第二个参数是“interest集合”，表示选择器所关心的通道操作，它实际上是一个表示选择器在检查通道就绪状态时需要关心的操作的比特掩码。比如一个选择器对通道的`read`和`write`操作感兴趣，那么选择器在检查该通道时，只会检查通道的`read`和`write`操作是否已经处在就绪状态。 这个"interest集合"有四种取值：
* Connect 连接 —— SelectionKey.OP_CONNECT
* Accept 接受——SelectionKey.OP_ACCEPT
* Read 读——SelectionKey.OP_RED
* Write 写——SelectionKey.OP_WRITE
需要注意并非所有的操作在所有的可选择通道上都能被支持，比如`ServerSocketChannel`支持`Accept`，而`SocketChannel`中不支持。我们可以通过通道上的`validOps()`方法来获取特定通道下所有支持的操作集合。


如果Selector对通道的多操作类型感兴趣，可以用“位或”操作符来实现：`int interestSet=SelectionKey.OP_READ|SelectionKey.OP_WRITE;`
当通道触发了某个操作之后，表示该通道的某个操作已经就绪，可以被操作。因此，某个`SocketChannel`成功连接到另一个服务器称为“连接就绪”(`OP_CONNECT`)。一个`ServerSocketChannel`准备好接收新进入的连接称为“接收就绪”（`OP_ACCEPT`）。一个有数据可读的通道可以说是“读就绪”(`OP_READ`)。等待写数据的通道可以说是“写就绪”(`OP_WRITE`)。

我们注意到`register()`方法会返回一个`SelectionKey`对象，我们称之为键对象。该对象包含了以下四种属性：
* interest集合
* read集合
* Channel
* Selector

interest集合是Selector感兴趣的集合，用于指示选择器对通道关心的操作，可通过SelectionKey对象的`interestOps()`获取。最初，该兴趣集合是通道被注册到`Selector`时传进来的值。该集合不会被选择器改变，但是可通过`interestOps()`改变。我们可以通过以下方法来判断Selector是否对Channel的某种事件感兴趣：
```Java
int interestSet=selectionKey.interestOps();
   boolean isInterestedInAccept  = (interestSet & SelectionKey.OP_ACCEPT) == SelectionKey.OP_ACCEPT；
```

read集合是通道已经就绪的操作的集合，表示一个通道准备好要执行的操作了,可通过`SelctionKey`对象的`readyOps()`来获取相关通道已经就绪的操作。它是interest集合的子集，并且表示了interest集合中从上次调用`select()`以后已经就绪的那些操作。（比如选择器对通道的read,write操作感兴趣，而某时刻通道的read操作已经准备就绪可以被选择器获知了，前一种就是interest集合，后一种则是read集合。）。JAVA中定义以下几个方法用来检查这些操作是否就绪：
```java
//int readSet=selectionKey.readOps();
   selectionKey.isAcceptable();//等价于selectionKey.readyOps()&SelectionKey.OP_ACCEPT
   selectionKey.isConnectable();
   selectionKey.isReadable();
   selectionKey.isWritable();
```

需要注意的是，通过相关的选择键的readyOps()方法返回的就绪状态指示只是一个提示，底层的通道在任何时候都会不断改变，而其他线程也可能在通道上执行操作并影响到它的就绪状态。另外，我们不能直接修改read集合。

取出SelectionKey所关联的Selector和Channel
通过SelectionKey访问对应的Selector和Channel：
```java
Channel channel =selectionKey.channel();
Selector selector=selectionKey.selector();
```

关于取消SelectionKey对象的那点事
我们可以通过SelectionKey对象的`cancel()`方法来取消特定的注册关系。该方法调用之后，该SelectionKey对象将会被”拷贝”至已取消键的集合中，该键此时已经失效，但是该注册关系并不会立刻终结。在下一次select()时，已取消键的集合中的元素会被清除，相应的注册关系也真正终结。


Seletor完整的代码
```JAVA
// 服务端
public class ServerSocketChannelTest {

    private int size = 1024;
    private ServerSocketChannel socketChannel;
    private ByteBuffer byteBuffer;
    private Selector selector;
    private final int port = 8998;
    private int remoteClientNum=0;

    public ServerSocketChannelTest() {
        try {
            initChannel();
        } catch (Exception e) {
            e.printStackTrace();
            System.exit(-1);
        }
    }

    public void initChannel() throws Exception {
        socketChannel = ServerSocketChannel.open();
        socketChannel.configureBlocking(false);
        socketChannel.bind(new InetSocketAddress(port));
        System.out.println("listener on port:" + port);
        selector = Selector.open();
        socketChannel.register(selector, SelectionKey.OP_ACCEPT);
        byteBuffer = ByteBuffer.allocateDirect(size);
        byteBuffer.order(ByteOrder.BIG_ENDIAN);
    }

    private void listener() throws Exception {
        while (true) {
           //阻塞到至少有一个通道在你注册的事件上就绪了
            int n = selector.select();
            if (n == 0) {
                continue;
            }
            //访问已选择键集合
            Iterator<SelectionKey> ite = selector.selectedKeys().iterator();
            while (ite.hasNext()) {
                SelectionKey key = ite.next();
                //a connection was accepted by a ServerSocketChannel.
                if (key.isAcceptable()) {
                    ServerSocketChannel server = (ServerSocketChannel) key.channel();
                    SocketChannel channel = server.accept();
                    registerChannel(selector, channel, SelectionKey.OP_READ);
                    remoteClientNum++;
                    System.out.println("online client num="+remoteClientNum);
                    replyClient(channel);
                }
                //a channel is ready for reading
                if (key.isReadable()) {
                    readDataFromSocket(key);
                }

                ite.remove();//must
            }

        }
    }

    protected void readDataFromSocket(SelectionKey key) throws Exception {
        SocketChannel socketChannel = (SocketChannel) key.channel();
        int count;
        byteBuffer.clear();
        while ((count = socketChannel.read(byteBuffer)) > 0) {
            byteBuffer.flip(); // Make buffer readable
            // Send the data; don't assume it goes all at once
            while (byteBuffer.hasRemaining()) {
                socketChannel.write(byteBuffer);
            }
            byteBuffer.clear(); // Empty buffer
        }
        if (count < 0) {
            socketChannel.close();
        }
    }

    private void replyClient(SocketChannel channel) throws IOException {
        byteBuffer.clear();
        byteBuffer.put("hello client!\r\n".getBytes());
        byteBuffer.flip();
        channel.write(byteBuffer);
    }

    private void registerChannel(Selector selector, SocketChannel channel, int ops) throws Exception {
        if (channel == null) {
            return;
        }
        channel.configureBlocking(false);
        channel.register(selector, ops);
    }


    public static void main(String[] args) {
        try {
            new ServerSocketChannelTest().listener();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

```JAVA
//客户端
public class SocketChannelTest {

    private int size = 1024;
    private ByteBuffer byteBuffer;
    private SocketChannel socketChannel;

    public void connectServer() throws IOException {
        socketChannel = SocketChannel.open();
        socketChannel.connect(new InetSocketAddress("127.0.0.1", 8998));
        byteBuffer = ByteBuffer.allocate(size);
        byteBuffer.order(ByteOrder.BIG_ENDIAN);
        receive();
    }

    private void receive() throws IOException {
        while (true) {
            int count;
            byteBuffer.clear();
            while ((count = socketChannel.read(byteBuffer)) > 0) {
                byteBuffer.flip();
                while (byteBuffer.hasRemaining()) {
                    System.out.print((char) byteBuffer.get());
                }
                //send("send data to server\r\n".getBytes());
                byteBuffer.clear();
            }
        }
    }

    private void send(byte[] data) throws IOException {
        byteBuffer.clear();
        byteBuffer.put(data);
        byteBuffer.flip();
        socketChannel.write(byteBuffer);
    }

    public static void main(String[] args) throws IOException {
        new SocketChannelTest().connectServer();
    }
}
```
