# IO

一直以来都搞不明白IO的framework，总觉得好多类啊好烦躁啊。现在来具体学习一下。

Java IO一般包含两个部分：
1. `java.io`中堵塞型IO。
2. `java.nio`中非堵塞型IO。

前者是基础，后者是扩展，通常称为"NEW IO"。到底new在什么地方呢？

操作系统的知识高速我们系统运行的瓶颈一般在于IO操作，一般打开某个IO通道需要大量的时间，同时端口中不一定就有足够的数据，这个read方法就一直等待获取此的端口的内容，从而浪费大量的系统资源。诶？不是有多线程技术。话是这么说，但是创建线程也要法费一定的时间和系统资源的，因此不一定可取。

Java NEW IO的非堵塞技术主要采取用了Observe模式，就是有一个具体的观察者来检测IO端口，如果有数据进入就会立即通知相应的应用程序。这样我们就避免建立多个线程，同时也避免了read等待的时间。


不过我们还是先学习下`java.io`。


## 概述

这个包中包含了非常非常多的IO类和接口（JDK1.6中有83个类或者接口，JDK1.8中没数反正很多）。这么多的类和接口，我们怎么入手呢？

IO可以进行简单的分类。IO主要包含三部分
1. 流式部分。
  这一部分是IO的主体部分。这部分可以概括为**两个对象一个桥梁**。 "两个对象"是指**字节流（Byte stream）** 和**字符流（Char Stream）** 的对应，“一个桥梁”是指从字节流到字符流的桥梁，对应于输入和输出为`InputStreamReader`和`OutputStreamWriter`。
2. 非流式部分。
  这一部分主要包含一些辅助流式部分的类，比如`File`类、`RandomAccessFile`类和`FileDescriptor`类。
3. 文件读取部分的与安全相关的类和以及与本地操作系统相关的文件系统的类。
  前者比如`SerializablePermission`类，后者比如`FileSystem`类和Win32`FileSystem`类和WinNT`FileSystem`类。

所以我们首先要把重点放在"IO"流上。

## IO中的流
流是什么？
> 流是一组有**顺序**的，** 有起点和终点**的字节集合，是对数据传输的总称或抽象。即数据在两设备间的传输称为流，流的本质是数据传输。

注意到上高亮的部分，可以知道流具有最基本的特点：“One dimension , one direction.” 即**流是一维的，同时流是单向**的。关于维和我们通常说的一维长度，二维平面，三维空间，四维时空……是同一个概念，流就是一维的。单向就是只可以一个方向（按顺序从头至尾依次）读取，不可以读到某个位置，再返回前面某个位置。

> 流的这种特性在JMS（Java Message Service）的API设计中得到了体现。JMS是J2EE平台下面向消息中间件的一个标准。（关于中间件技术有机会和大家探讨）JMS中有五种具体类型的消息，这些消息一般分为两类：①流式的消息――包含ByteMessage和StreamMessage；②非流式的消息――包含TextMessage、ObjectMessage和MapMessage。我们在明白IO中流的特点后，基本可以明白JMS API设计者的意图。

IO流的分类有很多的标准，比如按照处理数据的类型不同可以分为"字符流"和“字节流”，按照数据流向的不同可以分为"输入流"和“输出流”

那“字符流”和“字节流”、"输入流"和"输出流"各自有什么区别呢？

### 字符流（Byte Stream） 和 字节流（Char Stream）
因为数据编码的不同，从而有了对字符进行高效操作的流对象，其本质还是基于字节流读取的，去查了指定的码表而已。两者的区别如下表：


| |字节流     | 字符流     |
| :------------- | :------------- | : ---|
| 读写单位       | 以字节（8bit）为单位，一次读入或读出的是8位二进制      | 以字符为单位，根据码表映射字符，一次可能读多个字节，一次读入或读出是16为二进制 |
|处理对象   |  能处理所有类型的数据（如图片、avi等） |  只能处理字符类型的数据 |

设备上任何形式的资源都是以二进制存储的。二进制的最终都是以一个8位为数据单元进行体现的，所以计算机中最小数据单元就是字节。这就意味着，字节流能处理所有的数据。

上面的比较给我们就如何选择字符流还是字节流提供了启发：只要是处理纯文本数据，就优先考虑使用字符流。除此之外都使用字节流。

### 输出流和输入流
这两个概念就很好理解了，输出流就是往外写数据，而输入流就是从外面往里读数据。程序中往往根据传输数据的不同特性而使用不同的流，分类如下:

| |字符     | 字节     |
| :------------- | :------------- | : ---|
| 输入流       | Reader      | InputStream |
|输出流   |  Writer |  OutputStream |

下面是以数据处理类型为主要分类标准，以数据流向不同为次要分类标准进行的IO分类：

![IO](http://ovn0i3kdg.bkt.clouddn.com/java.io.jpeg)

这样的话，好像IO还是能学的:-D。

## IO目标媒介

上面我们提到了Java IO中的四个类：InputStream、OutputStream、Reader、Writer，而实际应用中，我们用到的是它们的子类。之所以设计那么多的子类，目的是让每一个类都应用于不同的媒介，实现不同的功能。这些用途总汇如下：

| 用途|  
| :------------- |
| 文件访问  (Files)    |
|网络访问  |
|内存缓存访问   |  
|线程内部通信（管道）（Pipes）   |  
|缓冲 (Buffering)  |   
|过滤(Filtering)   |   
|解析(Parsing)   |  
|读写文件  (Readers / Writers)  |  
|读写基本类型数据 (long, int etc.) |
|读写对象 (Object)   |  

上面各个类还可以按照媒介进行细分：

![IO媒介](http://ovn0i3kdg.bkt.clouddn.com/IO%E5%AA%92%E4%BB%8B)


事实上，IO的分类标准还有一种粗粒度的分类，可以将流分成“节点流”、“处理流”和"转换流"。
* `节点流`：直接与数据源相连，读入或读出。

  ![节点流](http://ovn0i3kdg.bkt.clouddn.com/%E8%8A%82%E7%82%B9%E6%B5%81.png)

  常用的节点流有以下几种
  * 父类 ：`InputStream` 、`OutputStream`、 `Reader`、 `Writer`
  * 文件：`FileInputStream` 、 `FileOutputStream` 、`FileReader` 、`FileWriter`文件进行处理的节点流
  * 数　组 ：`ByteArrayInputStream`、 `ByteArrayOutputStream`、 `CharArrayReader` 、`CharArrayWriter`对数组进行处理的节点流（对应的不再是文件，而是内存中的一个数组）
  * 字符串 ：`StringReader`、 `StringWriter`对字符串进行处理的节点流
  * 管　道 ：`PipedInputStream` 、`PipedOutputStream` 、`PipedReader` 、`PipedWriter`对管道进行处理的节点流
  直接使用节点流，读写不方便，为了更快地读写文件，才有处理流。

- 处理流：处理流和节点流一块使用，在节点流的基础上，再套接一层，套接在节点流上就是处理流。如`BufferedReader`处理流的构造方法总是要带有一个其他的流对象做参数。一个流对象经过其他流的多次包装，称为“流的链接”。

  ![处理流](http://ovn0i3kdg.bkt.clouddn.com/%E5%A4%84%E7%90%86%E6%B5%81.png)

  常用的处理流有以下几种：
  * 缓冲流：BufferedInputStrean 、BufferedOutputStream、 BufferedReader、 BufferedWriter 增加缓冲功能，避免频繁读写硬盘。
  * 转换流：InputStreamReader 、OutputStreamReader实现字节流和字符流之间的转换。
  * 数据流： DataInputStream 、DataOutputStream 等-提供将基础数据类型写入到文件中，或者读取出来。


## 常用的场景

由于I/O中的类是在太多了。到现在我们用到最多的场景就是读取文件，所以我们先了解一下最常见的用法吧。

### BufferedReader 用法
* 构造方法： BufferedReader br = new BufferReader(Reader in);
* int read();//读取单个字符。
* int read(char[] cbuf,int off,int len);//将字符读入到数组的某一部分。返回读取的字符数。达到尾部 ，返回-1。
* String readLine(); //读取一个文本行。
* void close(); //关闭该流。并释放与该流相关的所有资源。

使用示例：
```Java
package Buffered;  

import java.io.BufferedWriter;  
import java.io.FileWriter;  
import java.io.IOException;  

public class BufferedWriterDemo {  
    public static void main(String[] args) throws IOException {  
        FileWriter fw = new FileWriter("Buffered.txt");  
//      fw.write("ok168");  
//      fw.close();  
        /**
         * 为了提高写入的效率，使用了字符流的缓冲区。
         * 创建了一个字符写入流的缓冲区对象，并和指定要被缓冲的流对象相关联。
         */  
        BufferedWriter bufw = new BufferedWriter(fw);  

        //使用缓冲区中的方法将数据写入到缓冲区中。  
        bufw.write("hello world !");  
        bufw.newLine();  
        bufw.newLine();  
        bufw.write("!hello world !");  
        bufw.write("!hello world !");  
        //使用缓冲区中的方法，将数据刷新到目的地文件中去。  
        bufw.flush();  
        //关闭缓冲区,同时关闭了fw流对象  
        bufw.close();     
    }  
}
```
一个自定义的`BufferReader`类：
```Java
package Buffered;

import java.io.FileReader;
import java.io.IOException;

public class MyBufferedReader {

	private FileReader fr;
	private char []buf = new char[1024];
	private int count = 0;
	private int pos = 0;
	public MyBufferedReader(FileReader f){
		this.fr = f;		
	}
	public int myRead() throws IOException{
		if(count == 0){
			count = fr.read(buf);
			pos = 0;
		}
		if(count<0)
			return -1;
		int ch = buf[pos++];
		count--;
		return ch;
	}

	public String myReadLine() throws IOException{
		StringBuilder sb = new StringBuilder();
		int ch = 0;
		while ((ch = myRead()) != -1) {
			if (ch == '\r')
				continue;
			if (ch == '\n')
				return sb.toString();
			sb.append((char) ch);
			if(count == 0)
				return sb.toString();			
		}
		return null;
	}
	public void myClose() throws IOException {
		fr.close();
	}
}
```
### BufferedWriter 类
* 构造方法：bufferedWriter bf = new bufferedWriter(Writer out );
* void write(char ch);//写入单个字符。
* void write(char []cbuf,int off,int len)//写入字符数据的某一部分
* void write(String s,int off,int len)//写入字符串的某一部分。
* void newLine()//写入一个行分隔符。
* void flush();//刷新该流中的缓冲。将缓冲数据写到目的文件中去。
* void close();//关闭此流，再关闭前会先刷新他。

使用示例：
```Java
package Buffered;

import java.io.BufferedWriter;
import java.io.FileWriter;
import java.io.IOException;

public class BufferedWriterDemo {
	public static void main(String[] args) throws IOException {
		FileWriter fw = new FileWriter("Buffered.txt");
//		fw.write("ok168");
//		fw.close();
		/**
		 * 为了提高写入的效率，使用了字符流的缓冲区。
		 * 创建了一个字符写入流的缓冲区对象，并和指定要被缓冲的流对象相关联。
		 */
		BufferedWriter bufw = new BufferedWriter(fw);

		//使用缓冲区中的方法将数据写入到缓冲区中。
		bufw.write("hello world !");
		bufw.newLine();
		bufw.newLine();
		bufw.write("!hello world !");
		bufw.write("!hello world !");
		//使用缓冲区中的方法，将数据刷新到目的地文件中去。
		bufw.flush();
		//关闭缓冲区,同时关闭了fw流对象
		bufw.close();
	}
}
```

下面是一个使用`BufferReader`和`BufferedWriter`写的复制文本的小程序：
```Java
package IOtest;

import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.FileReader;
import java.io.FileWriter;
import java.io.IOException;

public class TextCopyByBuf {

	/**
	 * 首先创建读取字符数据流对象关联所要复制的文件。
	 * 创建缓冲区对象关联流对象。
	 * 从缓冲区中将字符创建并写入到要目的文件中。
	 * @throws IOException
	 */
	public static void main(String[] args) throws IOException {
		FileReader fr = new FileReader("C:\\demo.txt");
		FileWriter fw = new FileWriter("D:\\love.txt");
		BufferedReader bufr = new BufferedReader(fr);
		BufferedWriter bufw = new BufferedWriter(fw);
		//一行一行的寫。
		String line = null;
		while((line = bufr.readLine()) != null){
			bufw.write(line);
			bufw.newLine();
			bufw.flush();
		}
	/*	一个字节一个字节写。
	    int ch = 0;
		while((ch = bufr.read())!=-1){
			bufw.write(ch);
		}*/
		bufr.close();
		bufw.close();
	}
}
```

参考
* [Java IO流学习总结一：输入输出流](http://blog.csdn.net/zhaoyanjun6/article/details/54292148)
