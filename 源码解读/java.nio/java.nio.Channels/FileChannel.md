# FileChannel

![FileChannel](http://ovn0i3kdg.bkt.clouddn.com/FileChannel.png?imageView/2/w/400)


## public abstract class FileChannel extends AbstractInterruptibleChannel implements SeekableByteChannel,GatheringByteChannel, ScatteringByteChannel

类定义。继承了`AbstractInterruptibleChannel`，实现了很多接口。注意到这个类是一个抽象类，不能实例化。


## 构造方法
`FileChannel`有一个构造方法，但是是受保护的，所以外部不能实例化，当然不能实例化，因为本身就是一个抽象类。
```Java
protected FileChannel() { }
```

那么我们如何取得实例呢，那只能靠这个抽象类的实现类了。比如`FileSystemImpl`，我们可以利用`RandomAccessFile`或者`FileInputStream`的`getChannel()`方法来取得一个该抽象类的实例。

## open方法
重载了两个open方法。
```Java
public static FileChannel open(Path path,
                                   Set<? extends OpenOption> options,
                                   FileAttribute<?>... attrs)
        throws IOException
{
    FileSystemProvider provider = path.getFileSystem().provider();
    return provider.newFileChannel(path, options, attrs);
}

public static FileChannel open(Path path, OpenOption... options)
        throws IOException
{
    Set<OpenOption> set = new HashSet<OpenOption>(options.length);
    Collections.addAll(set, options);
    return open(path, set, NO_ATTRIBUTES);
}
```
Emmmm 具体不知道干什么用，先放着。

## read方法
重载了三个read的抽象方法，具体的实现参见子类。这个类的作用就是将数据从channel中读到ByteBuffer中，注意这个方法的参数中的`ByteBuffer`类实例。

```Java
public abstract int read(ByteBuffer dst) throws IOException;

public abstract long read(ByteBuffer[] dsts, int offset, int length) throws IOException;

public final long read(ByteBuffer[] dsts) throws IOException {
      return read(dsts, 0, dsts.length);
}
```

## write方法
`Channel`具有双向读和写的功能，这一点和BIO中的各种stream是不一样，的stream只有一个方法的功能，要么读，要么写。

同样重载了三个方法，都是抽象方法，需要子类重写。

```Java
public abstract int write(ByteBuffer src) throws IOException;

public abstract long write(ByteBuffer[] srcs, int offset, int length) throws IOException;

public final long write(ByteBuffer[] srcs) throws IOException {
    return write(srcs, 0, srcs.length);
}
```


剩下的方法用的场景很少，所以暂且先不看了。

## NIO中FileChannel和BIO中FileInputStream对比

我们通过一个例子来了解一下BIO中FileInputStream和NIO中FileChannel的使用对比。例子的功能是——从文件中读取内容并显示：

使用FileInputStream的例子：
```java
public static void readFileWithBIO(){
  BufferedInputStream in = null;
  try{
    in = new BufferedInputStrean(new FileInputStream("../BIO.txt"));
    byte[] buf = new byte[1024];
    int readLength = in.read(buf);
    while(readLength != -1){
      for (int i = 0; i < readLength; i ++){
        System.out.print((char)buf[i] + " ");
      }
      readLength = in.read(buf);
    }
  }catch (IOException e){
    e.printStackTrace();
  }fianlly{
    in.close();
  }
}
```

使用`FileChannel`的例子：
```Java
public static void method2(){
  RandomAccessFile aFile = null;
  try{
    aFile = new RandomAccessFile("../NIO.txt");

    FileChannel filechannel = aFile.getChannel();
    ByteBuffer buf = ByteBuffer.allocation(1024);

    int byteRead = filechannel.read(buf);
    while(byteRead != -1){
      buf.flip();
      while(buf.hasReamining()){
        System.out.print((char)buf.get() + " ");

        buf.compact();
        byteRead = filechannel.read(buf);
      }
    }
  }catch (IOException e){
    e.printStackTrace();
  }finally{
    try{
      if (aFile != null) {
        aFile.close();
      }
    }catch (IOException e){
      e.printStackTrace();
    }
  }
}
```

再来看一个复制文件的例子:
```java
public class CopyFileTest {

    /**
     * 使用BIO中的FileInputStream和FileOutInput来复制文件
     * @param fromPath 源文件地址
     * @param toPath 目标文件地址
     */
    public static void fileCopyWithBIO(String fromPath, String toPath) throws IOException{

        File formFile = new File(fromPath);
        if (!formFile.exist()) throw new IOException("origin file is not exist");
        File toFile = new File(toPath);
        if (!toFile.exists()){
            toFile.createNewFile();
        }

        try (InputStream in = new BufferedInputStream(new FileInputStream(fromPath));
        OutputStream out = new BufferedOutputStream(new FileOutputStream(toFile))){

            byte[] buf = new byte[1024];
            int byteRead;
            while ((byteRead = in.read(buf)) != -1){
                out.write(buf, 0, byteRead);
            }
        }catch (IOException e){
            e.printStackTrace();
        }

    }
    /**
     * 利用NIO中的FileChannle来复制文件
     * @param fromPath
     * @param toPath
     */
    public static void fileCopyWithFileChannel(String fromPath, String toPath) throws IOException{
        File fromFile = new File(fromPath);
        if (fromFile == null) throw new IOException("origin file is not exist");
        File toFile = new File(toPath);
        if (!toFile.exists()){
            toFile.createNewFile();
        }

        try (FileInputStream in = new FileInputStream(fromFile);
        FileOutputStream out = new FileOutputStream(toFile);
             FileChannel fileChannelInput = in.getChannel();
             FileChannel fileChannelOutput = out.getChannel()){

            // 两个channel之间通信
            fileChannelInput.transferTo(0, fileChannelInput.size(), fileChannelOutput);

        }catch (IOException e){
            e.printStackTrace();
        }
    }
}
```
分别使用了BIO和NIO中来实现文件的拷贝。
