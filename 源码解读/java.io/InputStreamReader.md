# InputStreamReader
一个很好的辨认字符流和字节流的方法是：名字中带有`stream`的是字节流，带有`reader`或者`writer`的是字符流。那`InputStreamReader`是个什么？

这是一个转换流，即从从字节流转为为字符流的桥梁：它读取字节并将其转码为特定的字符集（这个要参考`java.nio.charset.Charset`）。这个字符集可以是系统默认的也可以指定的。

每一次触发该类的`read`方法，都会触发底层的字节流读取一个或多个字节。所以一般为了更好的效率，都会配合缓冲流一起使用。例如：

```java
BufferedReader in = new BufferedReader(new InputStreamReader(System.in));
```
`InputStreamReader`的结构如下：

![InputStreamReader](http://ovn0i3kdg.bkt.clouddn.com/InputStreamReader.png)


### public class InputStreamReader extends Reader
类声明，该类继承自`Reader`。

### 构造方法
重载了6种构造方法。

#### public InputStreamReader(InputStream in){...}
接受一个字节流作为输入流，使用默认的字符格式。
```java
public InputStreamReader(InputStream in) {
    super(in);
    try {
        sd = StreamDecoder.forInputStreamReader(in, this, (String)null); // ## check lock object
    } catch (UnsupportedEncodingException e) {
        // The default encoding should always be available
        throw new Error(e);
    }
}
```
其中`sd`的声明为`private final StreamDecoder sd;`，这是一个字节流的转换器。由于没有指定字符集，所以会使用系统默认的字符集UTF-8。

> 这个类的核心就是这个字节流的转换器。
> 关于字符集的更多内容可以看另一篇博客。


####  public InputStreamReader(InputStream in, String charsetName) throws UnsupportedEncodingEx{...}
多了一个字符串的参数，这个参数指定了字符集的名字。如果没有这个字符串对应的字符集的名字，就会抛出`UnsupportedEncodingException`异常。定义如下：
```java
public InputStreamReader(InputStream in, String charsetName)
        throws UnsupportedEncodingException
{
    super(in);
    if (charsetName == null)
        throw new NullPointerException("charsetName");
    sd = StreamDecoder.forInputStreamReader(in, this, charsetName);
}
```

####  public InputStreamReader(InputStream in, Charset cs){...}
与上一个构造方法相比，第二个参数是字符集的对象，这样就保证了一定存在这个字符集，不会抛出异常了。
```java
public InputStreamReader(InputStream in, Charset cs) {
   super(in);
   if (cs == null)
       throw new NullPointerException("charset");
   sd = StreamDecoder.forInputStreamReader(in, this, cs);
}
```

#### public InputStreamReader(InputStream in, CharsetDecoder dec){...}
直接指定了字符转换器。
```java
public InputStreamReader(InputStream in, CharsetDecoder dec) {
   super(in);
   if (dec == null)
       throw new NullPointerException("charset decoder");
   sd = StreamDecoder.forInputStreamReader(in, this, dec);
}
```

### public String getEncoding();
返回这个流使用的字符集。如果流已经关闭就会返回null。定义如下：
```java
public String getEncoding() {
    return sd.getEncoding();
}
```
下面是一个测试程序：
```java
public static void main(String[] args) {
   InputStreamReader reader = new InputStreamReader(System.in);

   String encodeingName = reader.getEncoding();

   System.out.println(encodeingName); //UTF8
   try {
       reader.close();
       encodeingName = reader.getEncoding();
       System.out.println(encodeingName); //null
   }catch (IOException e){
       e.getStackTrace();
   }
}
```
这里验证了系统默认的字符集是UTF-8，并且在流关闭的情况下，返回null。

### read方法
重载了两个方法，其实都借助于字节流转换器的read方法。
#### public int read() throws IOException{...}
读取单个字符。
```java
public int read() throws IOException {
    return sd.read();
}
```
#### public int read(char cbuf[], int offset, int length) throws IOException{...}
将字节流中的字节转换为字符，并存储在cbuf中，存储位置偏移是offset，要读取的字符长度是length。


### public boolean ready() throws IOException {...}
判断下一个字符是不是可读的。同样调用的是字节流转换器的ready方法。
```java
public boolean ready() throws IOException {
    return sd.ready();
}
```

#### public void close() throws IOException{...}
关闭流。
```java
public void close() throws IOException {
    sd.close();
}
```



参考
* [ Java字节流和字符流的转换器：StreamDecoder](http://blog.csdn.net/zhangzeyuaaa/article/details/17354751)
