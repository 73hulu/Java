# FileReader

之前学到`InputStreamReader`，大部分情况下不会直接用`InputStreamReader`而是用它的子类`FileReader`。这是一个读取字符文件的方便类。这个类的构造函数默认字符编码和字节缓冲区大小都是适当的。如果想要自己制定这些，那就不要用这个类，而是在`FileInputStream`上构造一个`InputStreamReader`。`FileReader`要用于读取字符流。如果要读取原始字节流，请考虑使用`FileInputStream`。

结构如下：

![FileReader](http://ovn0i3kdg.bkt.clouddn.com/FileReader.png)

只有三个构造方法而已。


### public class FileReader extends InputStreamReader
类声明。该类继承自`InputStreamReader`。

### 构造方法
三个构造方法，其实本质都是使用了文件字节输入流`FileInputStream`.
#### public FileReader(String fileName) throws FileNotFoundException{...}
指定文件名字来构造文件读取。所以要承担文件不存在的后果。定义如下：
```java
public FileReader(String fileName) throws FileNotFoundException {
   super(new FileInputStream(fileName));
}
```
#### public FileReader(File file) throws FileNotFoundException{...}
参数是`File`对象。定义如下
```java
public FileReader(File file) throws FileNotFoundException {
     super(new FileInputStream(file));
 }
```
这个构造函数是最常用的，配合`BufferedReader`，最常见的写法是：
```java
BufferedReader in = new BufferedReader(new FileReader('foo.txt'));
```


#### public FileReader(FileDescriptor fd){...}
参数是`FileDescriptor`对象，`FileDescriptor`是一个辅助类，很少用到，所以暂时先不深究了。
```java
public FileReader(FileDescriptor fd) {
    super(new FileInputStream(fd));
}
```
