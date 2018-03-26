# ByteBuffer

NIO中重要的组成部分缓冲区，主要以字符的形式读取内容。类的结构如下：

![ByteBuffer](http://ovn0i3kdg.bkt.clouddn.com/ByteBuffer_1.png?imageView/2/w/400)
![ByteBuffer](http://ovn0i3kdg.bkt.clouddn.com/ByteBuffer_2.png?imageView/2/w/400)

## public abstract class ByteBuffer extends Buffer implements Comparable<ByteBuffer>

类声明。继承自`Buffer`抽象类。这是一个抽象类，它的一些具体的实现类如下：
![ByteBuffer的实现类](http://ovn0i3kdg.bkt.clouddn.com/ByteBuffer%E7%9A%84%E5%AE%9E%E7%8E%B0%E7%B1%BB.png?imageView/2/w/400)


在`ByteBuffer`中提供了四个静态工厂类来创建不同的`ByteBuffer`实例：

|方法名|描述|
|:---   |  :--- |
|allocate(int capacity)	|从堆空间中分配一个容量大小为capacity的byte数组作为缓冲区的byte数据存储器， 分配的`ByteBuffer`的类型是`HeapByteBuffer`|
|allocateDirect(int capacity)	|是不使用JVM堆栈而是通过操作系统来创建内存块用作缓冲区，它与当前操作系统能够更好的耦合，因此能进一步提高I/O操作速度。但是分配直接缓冲区的系统开销很大，因此只有在缓冲区较大并长期存在，或者需要经常重用时，才使用这种缓冲区, 分配的类型是`DirectByteBuffer`|
|wrap(byte[] array)	|这个缓冲区的数据会存放在byte数组中，bytes数组或buff缓冲区任何一方中数据的改动都会影响另一方。其实ByteBuffer底层本来就有一个bytes数组负责来保存buffer缓冲区中的数据，通过allocate方法系统会帮你构造一个byte数组|
|wrap(byte[] array, int offset, int length)	|在上一个方法的基础上可以指定偏移量和长度，这个offset也就是包装后byteBuffer的position，而length呢就是limit-position的大小，从而我们可以得到limit的位置为length+position(offset)|

## 构造函数
定义了两个构造函数，
```Java
// Creates a new buffer with the given mark, position, limit, capacity,
// backing array, and array offset
//
ByteBuffer(int mark, int pos, int lim, int cap,   // package-private
             byte[] hb, int offset)
{
    super(mark, pos, lim, cap);
    this.hb = hb;
    this.offset = offset;
}

// Creates a new buffer with the given mark, position, limit, and capacity
//
ByteBuffer(int mark, int pos, int lim, int cap) { // package-private
    this(mark, pos, lim, cap, null, 0);
}
```
特别注意到这里的参数变量，其中`mark`、`pos`、`lim`和`cap`是每个容器共有的属性，它们都定义在父类`Buffer`中：
```Java

// Invariants: mark <= position <= limit <= capacity


//标记，调用mark()来设置mark=position，再调用reset()可以让position恢复到标记的位置
private int mark = -1;
//位置，下一个要被读或写的元素的索引，每次读写缓冲区数据时都会改变改值，为下次读写作准备
private int position = 0;
//表示缓冲区的当前终点，不能对缓冲区超过极限的位置进行读写操作。且极限是可以修改的
private int limit;
//容量，即可以容纳的最大数据量；在缓冲区创建时被设定并且不能改变
private int capacity;
```
注意到这几个变量之间的大小关系`mark <= position <= limit <= capacity`


## 四个静态工厂方法
之前提到，`ByteBuffer`是抽象类，不能实例化，提供了静态工厂方法，用来创建不同类型的`ByteBuffer`，具体解释参考上面的表格
```Java
//直接内存存取快， 不在堆栈区分配空间
public static ByteBuffer allocateDirect(int capacity) {
    return new DirectByteBuffer(capacity);
}

//分配堆区的空间
public static ByteBuffer allocate(int capacity) {
    if (capacity < 0)
        throw new IllegalArgumentException();
    return new HeapByteBuffer(capacity, capacity);
}

// warp最终分配的还是堆区的空间，不同allocate方法的是，这个方法可以指定偏移和长度
public static ByteBuffer wrap(byte[] array) {
    return wrap(array, 0, array.length);
}

public static ByteBuffer wrap(byte[] array,
                                    int offset, int length)
{
    try {
        return new HeapByteBuffer(array, offset, length);
    } catch (IllegalArgumentException x) {
        throw new IndexOutOfBoundsException();
    }
}
```

对于这四个不同类型的`ByteBuffer`，下面有一个测试程序帮助理解：
```Java
public static void main(String[] args) {
    System.out.println("----------Test allocate--------");
    System.out.println("before alocate:"
            + Runtime.getRuntime().freeMemory());

    // 如果分配的内存过小，调用Runtime.getRuntime().freeMemory()大小不会变化？
    // 要超过多少内存大小JVM才能感觉到？
    ByteBuffer buffer = ByteBuffer.allocate(102400);
    System.out.println("buffer = " + buffer);

    System.out.println("after alocate:"
            + Runtime.getRuntime().freeMemory());

    // 这部分直接用的系统内存，所以对JVM的内存没有影响
    ByteBuffer directBuffer = ByteBuffer.allocateDirect(102400);
    System.out.println("directBuffer = " + directBuffer);
    System.out.println("after direct alocate:"
            + Runtime.getRuntime().freeMemory());

    System.out.println("----------Test wrap--------");
    byte[] bytes = new byte[32];
    buffer = ByteBuffer.wrap(bytes);
    System.out.println(buffer);

    buffer = ByteBuffer.wrap(bytes, 10, 10);
    System.out.println(buffer);
}
```
程序的输出结果是：
```
----------Test allocate--------
before alocate:126930104
buffer = java.nio.HeapByteBuffer[pos=0 lim=102400 cap=102400]
after alocate:126930104
directBuffer = java.nio.DirectByteBuffer[pos=0 lim=102400 cap=102400]
after direct alocate:126930104
----------Test wrap--------
java.nio.HeapByteBuffer[pos=0 lim=32 cap=32]
java.nio.HeapByteBuffer[pos=10 lim=20 cap=32]
```

## 常用方法
### limit方法
继承自父类`Buffer`，有点类似于jquery的val方法，有参数的时候是set方法，无参的时候是get方法。注意set的时候，可能会导致positon和limit值的改变：
```Java
public final int limit() {
    return limit;
}

public final Buffer limit(int newLimit) {
    if ((newLimit > capacity) || (newLimit < 0))
        throw new IllegalArgumentException();
    limit = newLimit;
    if (position > limit) position = limit;
    if (mark > limit) mark = -1;
    return this;
}
```

### mark方法
将mark位置设置成当前position。
```Java
public final Buffer mark() {
    mark = position;
    return this;
}
```

### reset方法
把position设置成mark的值，相当于之前做过一个标记，现在要退回到之前标记的地方
```Java
public final Buffer reset() {
    int m = mark;
    if (m < 0)
        throw new InvalidMarkException();
    position = m;
    return this;
}
```



### flip()方法
limit = position;position = 0;mark = -1;  翻转，也就是让flip之后的position到limit这块区域变成之前的0到position这块，翻转就是**将一个处于存数据状态的缓冲区变为一个处于准备取数据的状态。**
```Java
public final Buffer flip() {
    limit = position;
    position = 0;
    mark = -1;
    return this;
}
```

### rewind()方法
把position设为0，mark设为-1，不改变limit的值，
```Java
public final Buffer rewind() {
    position = 0;
    mark = -1;
    return this;
}
```

### remaining()
返回limit和position之间相对位置差。这表示的是剩余的容量。
```Java
public final int remaining() {
    return limit - position;
}
```

### hasRemaining()方法
也表示是否已经写满，返回的是布尔值。
```Java
public final boolean hasRemaining() {
    return position < limit;
}
```

### compact()方法
把从position到limit中的内容移到0到limit-position的区域内，position和limit的取值也分别变成limit-position、capacity。如果先将positon设置到limit，再compact，那么相当于clear()。

与compact不同的是，它会将剩余的元素搬运到写区域。但是clear就不会。

### get方法
重载了多个get方法。
* get()： 相对读，从position位置读取一个byte，并将position+1，为下次读写作准备，具体的实现在具体的实现类中。
* get(int index)：绝对读，读取byteBuffer底层的bytes中下标为index的byte，不改变position。
* get(byte[] dst, int offset, int length)：从position位置开始相对读，读length个byte，并写入dst下标从offset到offset+length的区域。


### put方法
* put(byte b)：相对写，向position的位置写入一个byte，并将postion+1，为下次读写作准备。
* put(int index, byte b)： 绝对写，向byteBuffer底层的bytes中下标为index的位置插入byte b，不改变position
* put(ByteBuffer src)：用相对写，把src中可读的部分（也就是position到limit）写入此byteBuffer
* put(byte[] src, int offset, int length)：从src数组中的offset到offset+length区域读取数据并使用相对写写入此byteBuffer


下面是这些常用方法的测试程序:
```Java
ByteBuffer buffer = ByteBuffer.allocate(102400);

 System.out.println("--------Test reset----------");
 buffer.clear();
 buffer.position(5);
 buffer.mark();
 buffer.position(10);
 System.out.println("before reset:" + buffer);
 buffer.reset();
 System.out.println("after reset:" + buffer);

 System.out.println("--------Test rewind--------");
 buffer.clear();
 buffer.position(10);
 buffer.limit(15);
 System.out.println("before rewind:" + buffer);
 buffer.rewind();
 System.out.println("before rewind:" + buffer);

 System.out.println("--------Test compact--------");
 buffer.clear();
 buffer.put("abcd".getBytes());
 System.out.println("before compact:" + buffer);
 System.out.println(new String(buffer.array()));
 buffer.flip();
 System.out.println("after flip:" + buffer);
 System.out.println((char) buffer.get());
 System.out.println((char) buffer.get());
 System.out.println((char) buffer.get());
 System.out.println("after three gets:" + buffer);
 System.out.println("\t" + new String(buffer.array()));
 buffer.compact();
 System.out.println("after compact:" + buffer);
 System.out.println("\t" + new String(buffer.array()));

 System.out.println("------Test get-------------");
 buffer = ByteBuffer.allocate(32);
 buffer.put((byte) 'a').put((byte) 'b').put((byte) 'c').put((byte) 'd')
         .put((byte) 'e').put((byte) 'f');
 System.out.println("before flip()" + buffer);
 // 转换为读取模式
 buffer.flip();
 System.out.println("before get():" + buffer);
 System.out.println((char) buffer.get());
 System.out.println("after get():" + buffer);
 // get(index)不影响position的值
 System.out.println((char) buffer.get(2));
 System.out.println("after get(index):" + buffer);
 byte[] dst = new byte[10];
 buffer.get(dst, 0, 2);
 System.out.println("after get(dst, 0, 2):" + buffer);
 System.out.println("\t dst:" + new String(dst));
 System.out.println("buffer now is:" + buffer);
 System.out.println("\t" + new String(buffer.array()));

 System.out.println("--------Test put-------");
 ByteBuffer bb = ByteBuffer.allocate(32);
 System.out.println("before put(byte):" + bb);
 System.out.println("after put(byte):" + bb.put((byte) 'z'));
 System.out.println("\t" + bb.put(2, (byte) 'c'));
 // put(2,(byte) 'c')不改变position的位置
 System.out.println("after put(2,(byte) 'c'):" + bb);
 System.out.println("\t" + new String(bb.array()));
 // 这里的buffer是 abcdef[pos=3 lim=6 cap=32]
 bb.put(buffer);
 System.out.println("after put(buffer):" + bb);
 System.out.println("\t" + new String(bb.array()));
```

参考
* [ByteBuffer常用方法详解](https://blog.csdn.net/u012345283/article/details/38357851?utm_source=tuicool&utm_medium=referral)
* [ByteBuffer类的常用方法](https://www.cnblogs.com/jiduoduo/p/6397454.html)
* !!! [攻破JAVA NIO技术壁垒](http://www.importnew.com/19816.html)
