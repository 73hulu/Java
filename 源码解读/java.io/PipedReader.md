# PipedReader & PipedWriter

管道流是使用与在多个线程之间进行信息传递的Java流，被号称是最难使用的流，被使用的频率也非常低。但是管道流是非常有用的，它提供了多线程间信息传输的一种有效手段。

首先需要明白一个概念：什么是管道？

在一个比较复杂的大型系统中，假如存在某个对象或者数据流需要被进行繁杂的逻辑处理的话，我们可以选择在一个大的组件中进行处理，这种方式简单粗暴，可能会带来某些麻烦，例如要改动、增加或减少一些处理逻辑，可能需要对整个组件进行改动，整个系统看起来没有任何可扩展性和可重用性。

是否有一种模式可以将整个处理流程进行详细划分，划分出的每个小模块互相独立且各自负责一段逻辑处理。这些逻辑处理小模块根据顺序连接起来，前一个模块的输出作为后一个模块的输入，最后一个模块的输出作为最终的处理结果。如果用来，修改逻辑只针对某个模块进行修改，增添或者删除逻辑也可细化到摸个模块颗粒度，并且每个模块都可以课重用，可重用性大大增强？

有，这就是管道模式。

顾名思义，管道模式就像一条管道把多个对象连接起来，整体看起来就像若干个阀门嵌套在管道中，而处理逻辑就放在阀门上，如下图，需要处理的对象进入管道后，分别经过阀门一、阀门二、阀门三、阀门四，每个阀门都会对进入的对象进行一些逻辑处理，经过一层层的处理后从管道尾处理，此时的对象就是已完成处理的目标对象。下面这幅图形象地说明了这种模式。

![Pipe](http://ovn0i3kdg.bkt.clouddn.com/pipe.png)

管道是Linux中很重要的一种通信方式，就是把程序的输出直接到另一个程序的输入。Java提供了管道流用于线程的通信。管道流一共有四个`PipedReader`、`PipedWriter`、`PipedInputStream`、`PipedOutputStream`。前两个是字符流，后面是两个字节流。

管道的实现必须是输入流和输出流共同作用的结果。先学习字符管道流。`PipedReader`的结构如下：

![PipedReader](http://ovn0i3kdg.bkt.clouddn.com/PipedReader.png)

管道字符输入流、用于读取对应绑定的管道字符输出流写入其内置字符缓存数组buffer中的字符、借此来实现线程之间的通信、pr中专门有两个方法供pw调用`receive(char c)`、`receive(char[] b, int off, intlen)`、使得pw可以将字符或者字符数组写入pr的buffer中。


### 关键字
`PipedReader`中有几个关键字需要弄明白，
```java
boolean closedByWriter = false;     //标记PipedWriter是否关闭

boolean closedByReader = false;    //标记PipedReader是否关闭

boolean connected = false;         // 标记PipedWriter与标记PipedReader是否关闭的连接是否关闭

Thread readSide;    //拥有PipedReader的线程

Thread writeSide;   //拥有PipedWriter的线程

private static final int DEFAULT_PIPE_SIZE = 1024; //用于循环存放PipedWriter写入的字符数组的默认大小

char buffer[];      //用于循环存放PipedWriter写入的字符数组， 这个数组是一个循环buf。

int in = -1;   //buf中下一个存放PipedWriter调用此PipedReader的`receive(int c)`时、c在buf中存放的位置的下标。此为初始状态、即buf中没有字符

int out = 0;   //buf中下一个被读取的字符的下标
```

### 构造函数
重载了4个构造函数。
#### public PipedReader(PipedWriter src) throws IOException {...}
该方法接受一个`PipedWriter`的对象作为参数？诶，不是构造的是`PipedReader`，传`Writer`的对象做什么。上面管道的图我们可以看到，管道有两端，入端和初端，入端需要有一个输入流，而出端需要一个输出流。这个构造方法的定义如下：
```java
public PipedReader(PipedWriter src) throws IOException {
    this(src, DEFAULT_PIPE_SIZE);
}
```
指定了输出流以及管道的默认大小，即1024。该方法调用下面这个构造方法。

### public PipedReader(PipedWriter src, int pipeSize) throws IOException{...}
```java
public PipedReader(PipedWriter src, int pipeSize) throws IOException {
   initPipe(pipeSize);
   connect(src);
}
```
首先根据指定的管道大小进行初始化，调用的`initPipe`方法定义如下：
```java
private void initPipe(int pipeSize) {
    if (pipeSize <= 0) {
        throw new IllegalArgumentException("Pipe size <= 0");
    }
    buffer = new char[pipeSize];
}
```
其中`buffer`是一个字符数组，用来保存管道数据，所以虽然管道的概念很难理解，但是底层实现很简单。

接着连接输出流，`connect`方法定义如下：
```java
public void connect(PipedWriter src) throws IOException {
    src.connect(this);
}
```
可见`PipedReader`中的`connect`方法调用的是`PipedWriter`中的`connect`方法。这样的话，输入和输出流相互就建立了连接。

#### public PipedReader(){...}
无参数的构造方法，定义如下：
```java
public PipedReader() {
    initPipe(DEFAULT_PIPE_SIZE);
}
```
只是按照默认大小初始化了字符数组，但是没有指定与之相连接的`PipedWriter`，所以按照这个构造函数实例化得到的对象，在使用之前必须由`PipedWriter`的一个实例来建立连接。

> `PipedReader`的`connect`方法和`PipedWriter`的`connect`方法实现的是同一个效果，只用其中一方调用就可以建立双向的连接。


#### public PipedReader(int pipeSize){...}
指定了管道的大小，同样没有建立连接，需要由`PipedWriter`来建立连接，定义如下:
```java
public PipedReader(int pipeSize) {
    initPipe(pipeSize);
}
```

#### receive方法
`PipedWriter`对象调用此方法，向`PipedReader`中的字符数组中字符，线程安全。重载了2个方法。

#### synchronized void receive(int c) throws IOException{...}
这个方法每次都是写入一个字符。定义如下：
```java
synchronized void receive(int c) throws IOException {
     if (!connected) {
         throw new IOException("Pipe not connected");
     } else if (closedByWriter || closedByReader) {
         throw new IOException("Pipe closed");
     } else if (readSide != null && !readSide.isAlive()) {
         throw new IOException("Read end dead");
     }

     writeSide = Thread.currentThread();
     while (in == out) {
         if ((readSide != null) && !readSide.isAlive()) {
             throw new IOException("Pipe broken");
         }
         /* full: kick any waiting readers */
         notifyAll();
         try {
             wait(1000);
         } catch (InterruptedException ex) {
             throw new java.io.InterruptedIOException();
         }
     }
     if (in < 0) {
         in = 0;
         out = 0;
     }
     buffer[in++] = (char) c;
     if (in >= buffer.length) {
         in = 0;
     }
 }
```
当`in == out`的时候，表示buf中写入的数据已经被读取完了，此时唤醒所有在此对象监控的线程的其他方法，如果一秒钟之后还是满值，则再次唤醒其他方法，直到buf中被读取。当`in < 0 `，表示buf中存放第一个字符，此时将buf中存放位置的下标in初始化为0，读取下标也初始化为0，准备接受写入的洗衣歌字符。如果buf中放满了，则再从头开始。
##### synchronized void receive(char c[], int off, int len)  throws IOException{...}
这个方法向buf中写入字符数组c中的一部分值。定义如下：
```java
synchronized void receive(char c[], int off, int len)  throws IOException {
    while (--len >= 0) {
        receive(c[off++]);
    }
}
```
每次写入一个值，循环调用。


###  synchronized void receivedLast(){...}
提醒所有等待的线程、已经接收到了最后一个字符、`PipedWriter`已关闭。用于`PipedWriter`的`close()`方法。定义如下：
```java
synchronized void receivedLast() {
    closedByWriter = true;
    notifyAll();
}
```

### read方法
顾名思义，就是从buf中读取数据， 以整数的形式返回。重载了两个方法。
#### public synchronized int read()  throws IOException {...}
```java
public synchronized int read()  throws IOException {
    if (!connected) {
        throw new IOException("Pipe not connected");
    } else if (closedByReader) {
        throw new IOException("Pipe closed");
    } else if (writeSide != null && !writeSide.isAlive()
               && !closedByWriter && (in < 0)) {
        throw new IOException("Write end dead");
    }

    readSide = Thread.currentThread();
    int trials = 2;
    while (in < 0) {
        if (closedByWriter) {
            /* closed by writer, return EOF */
            return -1;
        }
        if ((writeSide != null) && (!writeSide.isAlive()) && (--trials < 0)) {
            throw new IOException("Pipe broken");
        }
        /* might be a writer waiting */
        notifyAll();
        try {
            wait(1000);
        } catch (InterruptedException ex) {
            throw new java.io.InterruptedIOException();
        }
    }
    int ret = buffer[out++];
    if (out >= buffer.length) {
        out = 0;
    }
    if (in == out) {
        /* now empty */
        in = -1;
    }
    return ret;
}
```
如果这个管道还没有数据写入过，在存在写入线程的情况下，等待两轮，如果还没有写入就抛出异常。在等待的过程中唤醒在此对象上监控的线程的其他方法，然后自己等待1秒再次尝试读。否则就读取值然后将in推进一个。如果buf已经满了，就把out值重置。如果buf是空的，就把in值置为-1。当写入流关闭的情况下返回-1。

#### public synchronized int read(char cbuf[], int off, int len)  throws IOException{...}
将buf中的值读取到字符数组cbuf中，返回的是实际读取的字符长度。
```java
public synchronized int read(char cbuf[], int off, int len)  throws IOException {
    if (!connected) {
        throw new IOException("Pipe not connected");
    } else if (closedByReader) {
        throw new IOException("Pipe closed");
    } else if (writeSide != null && !writeSide.isAlive()
               && !closedByWriter && (in < 0)) {
        throw new IOException("Write end dead");
    }

    if ((off < 0) || (off > cbuf.length) || (len < 0) ||
        ((off + len) > cbuf.length) || ((off + len) < 0)) {
        throw new IndexOutOfBoundsException();
    } else if (len == 0) {
        return 0;
    }

    /* possibly wait on the first character */
    int c = read();
    if (c < 0) {
        return -1;
    }
    cbuf[off] =  (char)c;
    int rlen = 1;
    while ((in >= 0) && (--len > 0)) {
        cbuf[off + rlen] = buffer[out++];
        rlen++;
        if (out >= buffer.length) {
            out = 0;
        }
        if (in == out) {
            /* now empty */
            in = -1;
        }
    }
    return rlen;
}
```
在读取第一个字符的时候可能存在等待状态，第一个读取成功后，后面的就顺利了，为什么？因为重入锁。


### public synchronized boolean ready() throws IOException{...}
查看此流是否可读、看各个线程是否关闭、以及buffer中是否有可供读取的字符。
```java
public synchronized boolean ready() throws IOException {
   if (!connected) {
       throw new IOException("Pipe not connected");
   } else if (closedByReader) {
       throw new IOException("Pipe closed");
   } else if (writeSide != null && !writeSide.isAlive()
              && !closedByWriter && (in < 0)) {
       throw new IOException("Write end dead");
   }
   if (in < 0) {
       return false;
   } else {
       return true;
   }
}
```
### public void close()  throws IOException{...}
释放资源。
```java
public void close()  throws IOException {
   in = -1;
   closedByReader = true;
}
```

## 测试程序
`PipedReader`和`PipedWriter`两者必须配合使用，下面是一个简单的测试程序；
```java
//用于接受字符的线程
@SuppressWarnings("all")
public class CharReceiveThread  implements Runnable{
    private PipedReader pr = new PipedReader();

    @Override
    public void run() {
        //receiveOnChar();
        //receiveShortMessage();
        receiveLongMessage();

    }

    public PipedReader getPr() {
        return pr;
    }


    private void receiveOnChar(){
        try {
            int n = pr.read();
            System.out.println(n);
            pr.close();
        }catch (IOException e){
            e.getStackTrace();
        }
    }

    private void receiveShortMessage(){
        try {

            char [] b = new char[1024];
            int n = pr.read(b);
            System.out.println(new String(b, 0, n));
            pr.close();

        }catch (IOException e){
            e.getStackTrace();
        }
    }

    private void receiveLongMessage(){
        try {
            char [] b = new char[2048];

            int count = 0;
            while (true){
                count = pr.read(b);
                for (int i = 0; i < count; i ++){
                    System.out.println(b[i]);
                }
                if (count == -1){
                    break;
                }
                pr.close();
            }
        }catch (IOException e){
            e.getStackTrace();
        }
    }
}
```

```java
//发送字符的线程
@SuppressWarnings("all")
public class CharSenderThread implements Runnable{
    @Override
    public void run() {
        //sendOneChart();
        //sendShortMessage();
        sendLongMessage();
    }

    private PipedWriter pw = new PipedWriter();

    public PipedWriter getPw() {
        return pw;
    }


    private void sendOneChart(){
        try {
            pw.write('a');
            pw.flush();
            pw.close();
        }catch (IOException e){
            e.getStackTrace();
        }

    }

    private void sendShortMessage(){
        try {
            pw.write("this is a short message from CharSenderThread !".toCharArray());
            pw.flush();
            pw.close();
        }catch (IOException e){
            e.getStackTrace();
        }

    }
    private void sendLongMessage(){
        char [] b = new  char[1028];

        //生成一个长度为1028的字符数组，前1020个是a， 后8个是b
        for (int i = 0; i < 1020; i ++){
            b[i] = 'a';
        }
        for (int i = 1020; i < 1028; i ++){
            b[i] = 'b';
        }
        try {
            pw.write(b);
            pw.flush();
            pw.close();
        }catch (IOException e){
            e.getStackTrace();
        }
    }
}
```
测试程序：
```java
public class PipedReaderAndPipedReaderTest {
    public static void main(String[] args) throws IOException{
        CharReceiveThread cst = new CharReceiveThread();
        CharSenderThread crt = new CharSenderThread();

        PipedReader pr = cst.getPr();
        PipedWriter pw = crt.getPw();

        pw.connect(pr);

        new Thread(cst).start();
        new Thread(crt).start();
    }
}
```

> 总结
>
> PipedReader、PipedWriter两者的结合如鸳鸯一般、离开哪一方都不能继续存在，同时又如连理枝一般。PipedWriter先通过connect(PipedReader sink)来确定关系、并初始化PipedReader状态、告诉PipedReader只能属于这个PipedWriter、connect =true、当想赠与PipedReader字符时、就直接调用receive(char c) 、receive(char[] b, int off, int len)来将字符或者字符数组放入pr的存折buffer中。站在PipedReader角度上、看上哪个PipedWriter时就暗示pw、将主动权交给pw、调用pw的connect将自己给他去登记。当想要花（将字符读取到程序中）字符了就从buffer中拿、但是自己又没有本事挣字符、所以当buffer中没有字符时、自己就等着、并且跟pw讲没有字符了、pw就会向存折（buffer）中存字符、当然、pw不会一直不断往里存、当存折是空的时候也不会主动存、怕花冒、就等着pr要、要才存。过到最后两个只通过buffer来知道对方的存在与否、每次从buffer中存或者取字符时都会看看对方是否安康、若安好则继续生活、若一方不在、则另一方也不愿独存！





参考
* [管道模式——pipeline与valve](http://blog.csdn.net/wangyangzhizhou/article/details/45441355)
* [Java流编程实例之四--管道流](http://blog.csdn.net/logicteamleader/article/details/52557261)
* [Java_io体系之PipedWriter、PipedReader简介、走进源码及示例](https://www.2cto.com/kf/201312/263319.html)
