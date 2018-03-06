# java.io

`java.io`是一套Java用来读写数据（输入和输出）的API。这个包并没有涵盖所有的输入输出类型，例如，并不包括GUI或者网页上的输入输出，这些输入输出在其他地方都涉及，比如Swing工程中JFC(Java Foundation Classes)类，或者J2EE里的Servlet和HTTP包。`java.io`包主要涉及文件，网络数据流，内存缓冲等的输入输出。

这里的类主要可以分为三类
1. 流式部分。这是java.io的主体，概括起来是“两个对象和一个桥梁”，两个对象是指“字符流char stream”和“字节流byte stream”，一个桥梁是指**字节流**到**字符流**的桥梁，对应于输入和输出是`InputStreamReader`和`OutputStreamWriter`。
2. 辅助类，只要是文件`FIle`。
3. 与文件读取相关的安全类与本地操作相关的文件系统的类。

我们还是将重点放到流式部分。

分类如下:

![IO](http://ovn0i3kdg.bkt.clouddn.com/java.io.jpeg)


包主要包含的类如下：

| 类名 | 功能 |
| :------------- | :------------- |
| **File**	|  封装了对文件系统进行操作的功能      |
| **Reader**	   | 用于读取字符流的抽象类  |
|** BufferedReader **|   从字符输入流读取文本，缓冲字符，以提供字符，数组和行的有效读取|
|**InputStreamReader	**   |  InputStreamReader是从字节流到字符流的桥梁：它读取字节，并使用指定的字符集将其解码为字符|
| **FileReader**  |  用于读取字符文件的方便的流|
|StringReader	   | 用于读取字符串的流 |
|PipedReader   |   |
|CharArrayReader   |   |
|FilterReader   |   |
|PushbackReader   |   |
|**Writer	 **  |  用于写入字符流的抽象类 |
|**BufferedWriter	 **  | 将文本写入字符输出流，缓冲字符，以提供单个字符，数组和字符串的高效写入。  |
|**OutputStreamWriter**	   |   |
| **FileWriter ** |   |
|StringWriter   |   |
| PipedWriter |   |
|CharArrayWriter   |   |
|FilterReader   |   |
|**InputStream **  |   |
|**FileInputStream**   |   |
|FilterInputStream   |   |
|**BufferedInputStream **  |   |
|DataInputStream |   |
|PushbackInputStream   |   |
|  ObjectInputStream |   |
|PipedInputStream   |   |
|SequenceInputStream   |   |
|StringBufferInputStream   |   |
|ByteArrayInputStream   |   |
|**OutputStream**   |   |
| **FileOutputStream ** |   |
|FilterOutputStream   |   |
|**BufferedOutputStream **  |   |
|  PrintStream |   |
|ObjectOutputStream   |   |
|  PipedOutputStream |   |
|ByteArrayOutputStream   |   |


哎看到这个真的是要长叹一声啊，太多了。其实我们常用的就是加粗的几个类。



参考：
* [Java IO教程](http://ifeve.com/java-io/)
* [Java IO最详解](http://blog.csdn.net/yczz/article/details/38761237)
* [Java IO](http://blog.csdn.net/suifeng3051/article/details/48344587)
* [Java IO流学习总结](http://www.cnblogs.com/oubo/archive/2012/01/06/2394638.html)
* [Java IO流学习总结一：输入输出流](http://blog.csdn.net/zhaoyanjun6/article/details/54292148)
