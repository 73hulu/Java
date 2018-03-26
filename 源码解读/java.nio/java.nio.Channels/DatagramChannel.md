# DatagramChannel
`DatagramChannel`是一个能收发UDP包的通道。因为UDP是无连接的网络协议，所以不能像其它通道那样读取和写入。它发送和接收的是数据包。


这里又一次说到了TCP和UDP。我们回顾一下两者的区别。
TCP是面向连接的可靠传输层协议，适用于一些对传输准确性高或者本身有连接概念，但是对实时性要求不高的场景，在复杂较差的网络中也能保证准确性。
UDP是无连接不可靠的传输层协议，适用于对传输实时性要求高，但对传输准确性要求比较低的场景，如果网络比较差，那么准确性就真的不能保证了。


基于两者的特点，TCP可以用于网络数据库，分布式高精度计算系统的数据传输；UDP可以用于服务系统内部之间的数据传输，因为数据可能比较多，内部系统局域网内的丢包错包率又很低，即便丢包，顶多是操作无效，这种情况下，UDP经常被使用。

该类结果如下：

![DatagramChannel](http://ovn0i3kdg.bkt.clouddn.com/DatagramChannel.png?imageView/2/w/400)

## public abstract class DatagramChannel extends AbstractSelectableChannel implements ByteChannel, ScatteringByteChannel, GatheringByteChannel, MulticastChannel

类声明，抽象类，不可实例化，我们已经可以预见，这个类会提供静态方法（open）取得实例。

该类的实现类如下:
![DatagramChannel的实现类](http://ovn0i3kdg.bkt.clouddn.com/DatagramChannel%E7%9A%84%E5%AE%9E%E7%8E%B0%E7%B1%BB.png)

注意这两个实现类在sun包下。

## open
打开通道的方法：

```JAVA
public static DatagramChannel open() throws IOException {
    return SelectorProvider.provider().openDatagramChannel();
}

public static DatagramChannel open(ProtocolFamily family) throws IOException {
    return SelectorProvider.provider().openDatagramChannel(family);
}
```

实际上，这个方法就是所谓的创建对象的方法，总之我们现在已经得到了一个`DatagramChannel`的实例。

## socket方法
在`DatagramChannel`中，这是一个抽象方法，由子类去实现：
```JAVA
public abstract DatagramSocket socket();
```
取得一个`DatagramSocket`的实例。如果我们要**接受**来自一个UDP端口的数据，需要使用这个实例的`bind`方法将其绑定到一个端口上，一个完整的程序如下：
```java
DatagramChannel channel = DatagramChannel.open();
channel.socket().bind(new InetSocketAddress(9999);
```
这样`channel`就能接受来自端口9999的数据了。注意这时候是作为**服务端**。

## connect方法
这是作为**客户端**，连接到网络中特定的地址：
```JAVA
public abstract DatagramChannel connect(SocketAddress remote)
        throws IOException;
```
注意，由于UDP是无连接的，连接到特定地址并不会像TCP通道那样创建一个真正的连接。而是锁住DatagramChannel ，让其只能从特定地址收发数据。例如：
```JAVA
channel.connect(new InetSocketAddress("jenkov.com", 80));
```


## 例子

打开 DatagramChannel
```
DatagramChannel channel = DatagramChannel.open();
channel.socket().bind(new InetSocketAddress(9999));
```
接受数据：
```JAVA
ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
channel.receive(buf);
```
receive()方法会将接收到的数据包内容复制到指定的Buffer. 如果Buffer容不下收到的数据，多出的数据将被丢弃。

发送数据：
```JAVA
String newData = "New String to write to file..." + System.currentTimeMillis();

ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
but.put(newData.getBytes());
buf.flip();

int bytesSent = channel.send(buf, new InetSocketAddress("jenkov.com", 80));
```

这个例子发送一串字符到”jenkov.com”服务器的UDP端口80。 因为服务端并没有监控这个端口，所以什么也不会发生。也不会通知你发出的数据包是否已收到，因为UDP在数据传送方面没有任何保证。

连接到特定的地址：
可以将DatagramChannel“连接”到网络中的特定地址的。由于UDP是无连接的，连接到特定地址并不会像TCP通道那样创建一个真正的连接。而是锁住DatagramChannel ，让其只能从特定地址收发数据。例如：
```java
channel.connect(new InetSocketAddress("jenkov.com", 80));
```
当连接后，也可以使用read()和write()方法，就像在用传统的通道一样。只是在数据传送方面没有任何保证。这里有几个例子：
```JAVA
int bytesRead = channel.read(buf);
int bytesWritten = channel.write(but);
```

以下是一个服务器端和客户端发送UDP消息的例子：
```JAVA
// UDP协议服务器端
public class DatagramChannelServerDemo {
    private int port = 9975;
    DatagramChannel channel;
    private Charset charset = Charset.forName("UTF-8");
    private Selector selector = null;

    public DatagramChannelServerDemo() throws IOException {
        try {
            selector = Selector.open();
            channel = DatagramChannel.open();
        } catch (Exception e) {
            selector = null;
            channel = null;
            System.out.println("超时");
        }
        System.out.println("服务器启动");
    }

    /* 编码过程 */
    public ByteBuffer encode(String str) {
        return charset.encode(str);
    }

    /* 解码过程 */
    public String decode(ByteBuffer bb) {
        return charset.decode(bb).toString();
    }

    /* 服务器服务方法 */
    public void service() throws IOException {
        if(channel==null || selector==null) return;
        //设置为非阻塞模式 DatagramChannel类的configureBlocking(blooean block)block为true则通道处于阻塞模式，block为false则通道处于非阻塞模式
        channel.configureBlocking(false);
        //绑定到特定的端口上
        channel.socket().bind(new InetSocketAddress(port));
        // channel.write(ByteBuffer.wrap(new String("aaaa").getBytes()));
        channel.register(selector, SelectionKey.OP_READ);
        /** 外循环，已经发生了SelectionKey数目 */
        while (selector.select() > 0) {
            System.out.println("有新channel加入");
            /* 得到已经被捕获了的SelectionKey的集合 */
            Iterator iterator = selector.selectedKeys().iterator();
            while (iterator.hasNext()) {
                SelectionKey key = null;
                try {
                    key = (SelectionKey) iterator.next();
                    iterator.remove();

                    if (key.isReadable()) {
                        reveice(key);
                    }
                    if (key.isWritable()) {
                        // send(key);
                    }
                } catch (IOException e) {
                    e.printStackTrace();
                    try {
                        if (key != null) {
                            key.cancel();
                            key.channel().close();
                        }
                    } catch (ClosedChannelException cex) {
                        e.printStackTrace();
                    }
                }
            }
            /* 内循环完 */
        }
        /* 外循环完 */
    }

    /*
     * 接收 用receive()读IO
     * 作为服务端一般不需要调用connect()，如果未调用<span style="font-family: Arial, Helvetica, sans-serif;">connect()时调</span><span style="font-family: Arial, Helvetica, sans-serif;">用read()\write()读写，会报java.nio.channels</span>
     * .NotYetConnectedException 只有调用connect()之后,才能使用read和write.
     */
    synchronized public void reveice(SelectionKey key) throws IOException {
        if (key == null)
            return;
        // ***用channel.receive()获取客户端消息***//
        // ：接收时需要考虑字节长度
        DatagramChannel sc = (DatagramChannel) key.channel();
        String content = "";
        // create buffer with capacity of 48 bytes
        ByteBuffer buf = ByteBuffer.allocate(1024);// java里一个(utf-8)中文3字节,gbk中文占2个字节
        buf.clear();
        SocketAddress address = sc.receive(buf); // read into buffer. 返回客户端的地址信息
        String clientAddress = address.toString().replace("/", "").split(":")[0];
        String clientPost = address.toString().replace("/", "").split(":")[1];

        buf.flip(); // make buffer ready for read
        while (buf.hasRemaining()) {
            buf.get(new byte[buf.limit()]);// read 1 byte at a time
            content += new String(buf.array());
        }
        buf.clear(); // make buffer ready for writing
        System.out.println("接收：" + content.trim());
        // 第一次发；udp采用数据报模式，发送多少次，接收多少次
        ByteBuffer buf2 = ByteBuffer.allocate(65507);
        buf2.clear();
        buf2
                .put("消息推送内容 abc..UDP是一个非连接的协议，传输数据之前源端和终端不建立连接，当它想传送时就简单地去抓取来自应用程序的数据，并尽可能快地把它扔到网络上。在发送端UDP是一个非连接的协议，传输数据之前源端和终端不建立连接，当它想传送时就简单地去抓取来自应用程序的数据，并尽可能快地把它扔到网络上。在发送端UDP是一个非连接的协议，传输数据之前源端和终端不建立连接，当它想传送时就简单地去抓取来自应用程序的数据，并尽可能快地把它扔到网络上。在发送端@Q"
                        .getBytes());
        buf2.flip();
        channel.send(buf2, new InetSocketAddress(clientAddress,Integer.parseInt(clientPost))); // 将消息回送给客户端

        // 第二次发
        ByteBuffer buf3 = ByteBuffer.allocate(65507);
        buf3.clear();
        buf3.put("任务完成".getBytes());
        buf3.flip();
        channel.send(buf3, new InetSocketAddress(clientAddress, Integer.parseInt(clientPost))); // 将消息回送给客户端
    }

    int y = 0;

    public void send(SelectionKey key) {
        if (key == null)
            return;
        // ByteBuffer buff = (ByteBuffer) key.attachment();
        DatagramChannel sc = (DatagramChannel) key.channel();
        try {
            sc.write(ByteBuffer.wrap(new String("aaaa").getBytes()));
        } catch (IOException e1) {
            e1.printStackTrace();
        }
        System.out.println("send2() " + (++y));
    }

    /* 发送文件 */
    public void sendFile(SelectionKey key) {
        if (key == null)
            return;
        ByteBuffer buff = (ByteBuffer) key.attachment();
        SocketChannel sc = (SocketChannel) key.channel();
        String data = decode(buff);
        if (data.indexOf("get") == -1)
            return;
        String subStr = data.substring(data.indexOf(" "), data.length());
        System.out.println("截取之后的字符串是 " + subStr);
        FileInputStream fileInput = null;
        try {
            fileInput = new FileInputStream(subStr);
            FileChannel fileChannel = fileInput.getChannel();
            fileChannel.transferTo(0, fileChannel.size(), sc);
            fileChannel.close();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                fileInput.close();
            } catch (IOException ex) {
                ex.printStackTrace();
            }
        }
    }
    public static void main(String[] args) throws IOException {
        new DatagramChannelServerDemo().service();
    }
}
```

```JAVA
// UDP协议客户端
public class DatagramChannelClientDemo {
    // UDP协议客户端
    private String serverIp = "127.0.0.1";
    private int port = 9975;
    // private ServerSocketChannel serverSocketChannel;
    DatagramChannel channel;
    private Charset charset = Charset.forName("UTF-8");
    private Selector selector = null;

    public DatagramChannelClientDemo() throws IOException {
        try {
            selector = Selector.open();
            channel = DatagramChannel.open();
        } catch (Exception e) {
            selector = null;
            channel = null;
            System.out.println("超时");
        }
        System.out.println("客户器启动");
    }

    /* 编码过程 */
    public ByteBuffer encode(String str) {
        return charset.encode(str);
    }

    /* 解码过程 */
    public String decode(ByteBuffer bb) {
        return charset.decode(bb).toString();
    }

    /* 服务器服务方法 */
    public void service() throws IOException {
        if(channel==null || selector==null) return;
        channel.configureBlocking(false);
        channel.connect(new InetSocketAddress(serverIp, port));// 连接服务端
        channel.write(ByteBuffer.wrap(new String("客户端请求获取消息").getBytes()));
        channel.register(selector, SelectionKey.OP_READ);
        /** 外循环，已经发生了SelectionKey数目 */
        while (selector.select() > 0) {
            /* 得到已经被捕获了的SelectionKey的集合 */
            Iterator iterator = selector.selectedKeys().iterator();
            while (iterator.hasNext()) {
                SelectionKey key = null;
                try {
                    key = (SelectionKey) iterator.next();
                    iterator.remove();
                    if (key.isReadable()) {
                        reveice(key);
                    }
                    if (key.isWritable()) {
                        // send(key);
                    }
                } catch (IOException e) {
                    e.printStackTrace();
                    try {
                        if (key != null) {
                            key.cancel();
                            key.channel().close();
                        }
                    } catch (ClosedChannelException cex) {
                        e.printStackTrace();
                    }
                }
            }
            /* 内循环完 */
        }
        /* 外循环完 */
    }

    /* 接收 */
    synchronized public void reveice(SelectionKey key) throws IOException {
        String threadName = Thread.currentThread().getName();
        if (key == null)
            return;
        try {
            // ***用channel.receive()获取消息***//
            // ：接收时需要考虑字节长度
            DatagramChannel sc = (DatagramChannel) key.channel();
            String content = "";
            //第一次接；udp采用数据报模式，发送多少次，接收多少次
            ByteBuffer buf = ByteBuffer.allocate(65507);// java里一个(utf-8)中文3字节,gbk中文占2个字节
            buf.clear();
            SocketAddress address = sc.receive(buf); // read into buffer.
            String clientAddress = address.toString().replace("/", "").split(":")[0];
            String clientPost = address.toString().replace("/", "").split(":")[1];
            System.out.println(threadName + "\t" + address.toString());
            buf.flip(); // make buffer ready for read
            while (buf.hasRemaining()) {
                buf.get(new byte[buf.limit()]);// read 1 byte at a time
                byte[] tmp = buf.array();
                content += new String(tmp);
            }
            buf.clear(); // make buffer ready for writing次
            System.out.println(threadName + "接收：" + content.trim());
            //第二次接
            content = "";
            ByteBuffer buf2 = ByteBuffer.allocate(65507);// java里一个(utf-8)中文3字节,gbk中文占2个字节
            buf2.clear();
            SocketAddress address2 = sc.receive(buf2); // read into buffer.
            buf2.flip(); // make buffer ready for read
            while (buf2.hasRemaining()) {
                buf2.get(new byte[buf2.limit()]);// read 1 byte at a time
                byte[] tmp = buf2.array();
                content += new String(tmp);
            }
            buf2.clear(); // make buffer ready for writing次
            System.out.println(threadName + "接收2：" + content.trim());

        } catch (PortUnreachableException ex) {
            System.out.println(threadName + "服务端端口未找到!");
        }
        send(2);
    }

    boolean flag = false;

    public void send(int i) {
        if (flag)
            return;
        try {
            // channel.write(ByteBuffer.wrap(new String("客户端请求获取消息(第"+i+"次)").getBytes()));
            // channel.register(selector, SelectionKey.OP_READ );
            ByteBuffer buf2 = ByteBuffer.allocate(48);
            buf2.clear();
            buf2.put(("客户端请求获取消息(第" + i + "次)").getBytes());
            buf2.flip();
            channel.write(buf2);
            channel.register(selector, SelectionKey.OP_READ );
//          int bytesSent = channel.send(buf2, new InetSocketAddress(serverIp,port)); // 将消息回送给服务端
        } catch (IOException e) {
            e.printStackTrace();
        }
        flag = true;
    }

    int y = 0;

    public void send(SelectionKey key) {
        if (key == null)
            return;
        // ByteBuffer buff = (ByteBuffer) key.attachment();
        DatagramChannel sc = (DatagramChannel) key.channel();
        try {
            sc.write(ByteBuffer.wrap(new String("aaaa").getBytes()));
        } catch (IOException e1) {
            e1.printStackTrace();
        }
        System.out.println("send2() " + (++y));
    }

    /* 发送文件 */
    public void sendFile(SelectionKey key) {
        if (key == null)
            return;
        ByteBuffer buff = (ByteBuffer) key.attachment();
        SocketChannel sc = (SocketChannel) key.channel();
        String data = decode(buff);
        if (data.indexOf("get") == -1)
            return;
        String subStr = data.substring(data.indexOf(" "), data.length());
        System.out.println("截取之后的字符串是 " + subStr);
        FileInputStream fileInput = null;
        try {
            fileInput = new FileInputStream(subStr);
            FileChannel fileChannel = fileInput.getChannel();
            fileChannel.transferTo(0, fileChannel.size(), sc);
            fileChannel.close();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                fileInput.close();
            } catch (IOException ex) {
                ex.printStackTrace();
            }
        }
    }

    public static void main(String[] args) throws IOException {
        new Thread(new Runnable() {
            public void run() {
                try {
                    new DatagramChannelClientDemo().service();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }
}
```










参考
* [Java NIO系列教程（十） Java NIO DatagramChannel](http://ifeve.com/datagram-channel/)
