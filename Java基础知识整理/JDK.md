
# JDK
Java 体系结构如下图：

![Java System](http://ovn0i3kdg.bkt.clouddn.com/Java_System.png)

### JDK、JRE和JVM
* JDK（Java development Kit）组成：
Java程序设计语言 + Java虚拟机 + Java API类库

* JRE(Java Runtime Environment)组成：
Java SE API子集 + Java 虚拟机，与JDK相比，它不包含开发工具——编译器、调试器和其他工具

* JVM(Java Virtual Mechinal)：
虚拟出来的机器，有自己完善的硬件架构，如处理器、堆栈、寄存器和相应的指令系统。Java能跨平台运行，其实就是不同的操作系统，使用不同的JVM映射规则，让其与操作系统无关，完成跨平台性。

JVM主要工作是解释自己的指令集（即字节码）并映射到本地的CPU的指令集或OS的系统调用。


根据Java技术关注的终点业务领域来划分，Java技术体系可分为4个平台：
1. `Java Card`： 支持一些Java小程序（Applets）运行在小内存设备（如智能卡）上的平台。
2. `Java ME(Micro Edition)`：支持Java程序运行在移动终端上的平台，
3. `Java SE(Standard Edition)`：支持面向桌面级应用的平台，提供完整的核心Java核心API，这个版本以前称为J2SE.
4. `Java EE(Enterprise Edition)`：支持使用多层架构的企业应用的平台，除了提供Java SE API之外，还对其做了大量的扩充，并提供了相应的部署支持。这个版本以前称为J2EE.

### C++和Java的异同
相同：OOP
不同：
1. Java是解释型语言（.java -(编译)-> .class -(JVM解释)->执行），C++是编译型语言。所以JAVA比C++要慢一点
2. Java跨平台，C++不跨平台
3. Java纯面向对象，不存在全局变量或全局函数，C++具有面向过程和面向对象，可以定义全局变量和全局函数
4. Java没有指针更加安全
5. Java不支持多重继承，但是有接口继承，C++可以有多重继承
6. Java提供垃圾回收器来实现垃圾的自动回收，C++需要开发人员自己管理内存
7. Java不支持运算符重载，C++支持
8. Java没有预处理器（但是提供import与预处理器相似），C++有预处理器
9. Java不支持goto，但是`goto`是保留字，C++支持goto。
10. Java不支持自动类型转换，必须由开发人员进行显示强制类型转换
11. Java没有结构和联合，C++支持，C++会导致安全问题
12. Java支持文档内建，C++不支持
13. Java提供标准库，如Servlet、JSP提供对web的支持，Socket、RMI可以用来开发分布式应用程序。C++依靠一些非标准、由其他厂商提供的库

### java开发工具

| 命令 | 作用 |
| :------------- | :------------- |
|javac   |  讲一个.java文件编译成.class文件 |
| java | 用来运行一个.class文件 |
|  javadoc |用来生成api文档   |
|jar   |用来生成jar包   |
|jdb   | 调试工具  |
|javaprof   | 剖析工具  |
|javah   |把java代码声明的JNI方法转化成C\C++头文件，就是那些native方法  |

问： Java开发工具是Java语言写的？
对，但是其底层是C++写的。

命令可以通过-`help`来查看用法，如`javac -help`:
![javac](http://ovn0i3kdg.bkt.clouddn.com/javac.png)

## 【JDK 1.6 & 1.7】String.intern()方法
`String.intern()`是一个Native方法，其作用是：如果字符串常量池中已经包含一个等于此String对象的字符串，则返回代表池中这个字符串的String对象；否则，将此String对象包含的字符串添加到常量池中，并且返回此String对象的引用。

不同JDK对字符串常量池的实现不同，分界点是JDK1.7。
例如下面这段代码：
```Java
public static void main(String[] args) {
    String str1 = new StringBuilder("计算机").append("软件").toString();
    System.out.println(str1.intern() == str1);

    String str2 = new StringBuilder("ja").append("va").toString();
    System.out.println(str2.intern() == str2);
}
```
以上代码如果在JDK1.6中运行，会得到两个false，但是在JDK1.7之后，会得到一个true和false。

差异的原因在于：
在JDK1.6中，`intern()`方法会将**首次**遇到的字符实例复制到永久代中，返回的也是永久代中这个字符串实例的引用，而用`StringBuilder`创建的字符串实例在Java堆上，所以必然不是同一个引用，将返回false。

在JDK1.7+中，`intern()`不再复制实例，只是在常量池中记录首次出现的实例引用，因此`intern()`返回的引用和由`StringBuilder`创建的那个字符串将是同一个。对于str2比较返回false是因为“java”这个字符串在执行`StringBuilder.toString()`之前已经出现过，字符串常量中已经有其引用，不符合"**首次**出现"的原则，而“计算机软件”这个字符串则是首次出现的，因此返回true。



### JDK版本

#### 集合框架 @since 1.2
#### NIO @since 1.4
#### Lock @since 1.5
#### 自动装箱 @since 1.5
#### Enum @since 1.5
#### Objects @since 1.7
#### Predicate @since 1.8
#### Iterable @since 1.5
