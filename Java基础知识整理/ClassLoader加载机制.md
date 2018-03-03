# ClassLoader加载机制
<!-- toc -->
<!-- tocstop -->

从看`Class`开始，就感觉Java的学习境界更上一层楼了，`ClassLoader`应该比较接近 JVM了。虽然我不是什么框架开发者，一般来说用不到，但是理解classLoader的加载机制，也有利于我们编写出更高效的代码。尝试看看吧。

## 什么是ClassLoader
翻译过来就是"加载器"，加载什么呢?加载class文件。当我们编写的源文件经过编译后，产生字节码文件，即所谓的class文件，ClassLoader的作用就是讲class文件加载到JVM中，然后程序就可以正确运行了，但是JVM启动的时候，并不会一次加载所有的class文件【那当然了，那么多的class文件，都加载了内存岂不是要爆炸】，而是根据程序的需要，通过类加载机制（ClassLoader)来动态加载某个class文件到内存当中去，只要class文件被载入到了内存之后，才能被其他class所引用。

那么Java的类加载流程是怎样的呢？

在学习之前先来回顾一下Java环境变量。回忆下，在mac或linux环境下不用配置，但是在windows环境下是需要配置三个环境变量的：
1. `JAVA_HOME`： 指向JDK的安装位置，一般默认安装在C盘，比如`C:\Program Files\Java\jdk1.8.0_91`
2. `PATH`：将程序路径包含在PATH当中，在命令行窗口就可以直接键入它的名字而不需要键入全路径， 一般的 `PATH=%JAVA_HOME%\bin;%JAVA_HOME%\jre\bin;%PATH%;` 可以看到这里%JAVA_HOME%就是上面提到的JDK的安装位置，`%JAVA_HOME%\bin`只的是JDK目录下的bin目录，`%JAVA_HOME%\jre\bin`指的是JDK目录下的jre目录的bin。
3. `CLASSPATH`：`CLASSPATH=.;%JAVA_HOME%\lib;%JAVA_HOME%\lib\tools.jar`, 注意前面`.;`，`.`代表当前目录，可以看到CLASSPATH指向了jar包路径。


为什么要回顾环境变量呢，因为它和类加载流程关系密切。

Java语言系统自带了三个类加载器：
* `Bootstrap ClassLoader` ：称为启动类加载器，用C++语言写的，在JVM启动后初始化的，Java类加载层次中最顶层的类加载器，负责加载JDK中的核心类库，`%JRE_HOME%\jre\lib`下的`rt.jar`、`resources.jar`、`charsets.jar`和`class`等。另外需要注意的是可以通过启动JVM时指定`-Xbootclasspath`和路径来改变Bootstrap ClassLoader的加载目录。比如`java -Xbootclasspath/a:path`被指定的文件追加到默认的bootstrap路径中。
* `Extention ClassLoader`: 扩展的类加载器，Bootstrap ClassLoader加载Extention ClassLoader，并且将Extention ClassLoader的父加载器设置为Bootstrap ClassLoader。Extention ClassLoader是用Java写的，具体来说就是`sun.misc.Launcher$ExtClassLoader`，Extention ClassLoader主要加载`%JAVA_HOME%/jre/lib/ext`此路径下的所有classes目录以及`java.ext.dirs`系统变量指定的路径中类库。
* ` App ClassLoader`：Bootstrap ClassLoader加载完Extention ClassLoader后，就会加载App ClassLoader，并且将App ClassLoader的父加载器指定为 Extention ClassLoader。App ClassLoader也是用Java写成的，它的实现类是`sun.misc.Launcher$AppClassLoader`，另外我们知道ClassLoader中有个`getSystemClassLoade`r方法,此方法返回的正是`AppclassLoader.AppClassLoader`主要负责加载classpath所指定的位置的类或者是jar文档，它也是Java程序默认的类加载器。

> 这三个是默认的类加载器，当然我们可以根据自己的需要定义自己的ClassLoader，这些类（包括Extension ClassLoader和AppClassLoader都继承java.lang.ClassLoader，而Bootstrap ClassLoader不继承，因为它是用C++写的）。

所以，JVM启动时，三个类加载器组成的初始类加载器层次结构是这样的：

![classLoader_type](http://ovn0i3kdg.bkt.clouddn.com/ClassLoader_type.png)

为了更好理解我们进入`sun.misc.Launcher`类，这是Java虚拟机入口应用：
```java
public class Launcher {
    private static Launcher launcher = new Launcher();
    private static String bootClassPath =
        System.getProperty("sun.boot.class.path");

    public static Launcher getLauncher() {
        return launcher;
    }

    private ClassLoader loader;

    public Launcher() {
        // Create the extension class loader
        ClassLoader extcl;
        try {
            extcl = ExtClassLoader.getExtClassLoader();
        } catch (IOException e) {
            throw new InternalError(
                "Could not create extension class loader", e);
        }

        // Now create the class loader to use to launch the application
        try {
            loader = AppClassLoader.getAppClassLoader(extcl);
        } catch (IOException e) {
            throw new InternalError(
                "Could not create application class loader", e);
        }

        //设置AppClassLoader为线程上下文类加载器，这个文章后面部分讲解
        Thread.currentThread().setContextClassLoader(loader);
    }

    /*
     * Returns the class loader used to launch the main application.
     */
    public ClassLoader getClassLoader() {
        return loader;
    }
    /*
     * The class loader used for loading installed extensions.
     */
    static class ExtClassLoader extends URLClassLoader {}

/**
     * The class loader used for loading from java.class.path.
     * runs in a restricted security context.
     */
    static class AppClassLoader extends URLClassLoader {}

```
源码有精简，可看出个大概：

1. Launcher初始化了`ExtClassLoader`和`AppClassLoader`。

2. Launcher中并没有看见`BootstrapClassLoader`，但通过`System.getProperty("sun.boot.class.path")`得到了字符串`bootClassPath`,这个应该就是`BootstrapClassLoader`加载的jar包路径。

用代码检测这个路径是什么内容： `System.out.println(System.getProperty("sun.boot.class.path"));`，打印输出：
```java
/Library/Java/JavaVirtualMachines/jdk1.8.0_141.jdk/Contents/Home/jre/lib/resources.jar:
/Library/Java/JavaVirtualMachines/jdk1.8.0_141.jdk/Contents/Home/jre/lib/rt.jar:
/Library/Java/JavaVirtualMachines/jdk1.8.0_141.jdk/Contents/Home/jre/lib/sunrsasign.jar:
/Library/Java/JavaVirtualMachines/jdk1.8.0_141.jdk/Contents/Home/jre/lib/jsse.jar:
/Library/Java/JavaVirtualMachines/jdk1.8.0_141.jdk/Contents/Home/jre/lib/jce.jar:
/Library/Java/JavaVirtualMachines/jdk1.8.0_141.jdk/Contents/Home/jre/lib/charsets.jar:
/Library/Java/JavaVirtualMachines/jdk1.8.0_141.jdk/Contents/Home/jre/lib/jfr.jar:
/Library/Java/JavaVirtualMachines/jdk1.8.0_141.jdk/Contents/Home/jre/classes
```
这些都是jre目录下的jar包或者class文件。


对于`Extension classLoader`，我们之前说过可以通过`-D java.ext.dirs`来添加或改变加载路径，用`System.out.println(System.getProperty("java.ext.dirs"));`测试后打印输出如下内容：
```java
/Users/baoxiaofang/Library/Java/Extensions:/Library/Java/JavaVirtualMachines/jdk1.8.0_141.jdk/Contents/Home/jre/lib/ext:
/Library/Java/Extensions:/Network/Library/Java/Extensions:/System/Library/Java/Extensions:/usr/lib/java
```

对于`App ClassLoader`，加载路径是`java.class.path`，我们同样打印一下`System.out.println(System.getProperty("java.class.path"));`,输出：
```java
/Library/Java/JavaVirtualMachines/jdk1.8.0_141.jdk/Contents/Home/jre/lib/charsets.jar:
/Library/Java/JavaVirtualMachines/jdk1.8.0_141.jdk/Contents/Home/jre/lib/deploy.jar:
....
```
打印的实在是太多了就不粘贴过来了。

现在知道了，三个类加载器是通过查阅相应的环境属性：`sun.boot.class.path`、`java.ext.dirs`和`java.class.path`来加载资源文件的。

回想一下，在学习`Class`类的时，有一个`getClassLoader`方法用来获取该类的加载器，下面是测试程序
```java
public class ClassLoaderTest {
    public static void main(String[] args) {

        ClassLoader classLoader = ClassLoaderTest.class.getClassLoader();
        System.out.println(classLoader.toString()); //sun.misc.Launcher$AppClassLoader@18b4aac2

    }
}
```
可见，这个自己写的类的类加载器是AppClassLoader。那么是不是可以尝试打印int或者String这种类型的类加载器呢，进行下面的尝试：
```java
public class ClassLoaderTest {
    public static void main(String[] args) {
        ClassLoader clsl1 = int.class.getClassLoader();
        System.out.println(clsl1.toString());
    }
}
```
运行之后抛出`java.lang.NullPointerException`，用String类尝试也是抛出异常。难道这个类没有加载器，不可能啊，他们使用`BootstrapClassLoader`加载的。

要解决一个问题，首先要明确一个概念：**每个类加载器都有一个父加载器**，如下：
```java
public class ClassLoaderTest {
    public static void main(String[] args) {

        ClassLoader classLoader = ClassLoaderTest.class.getClassLoader();
        System.out.println("ClassLoaderTest类的类加载器是： " + classLoader.toString()); //sun.misc.Launcher$AppClassLoader@18b4aac2

        ClassLoader parentClassLoader = classLoader.getParent();
        System.out.println("ClassLoaderTest类的类加载器的父类加载器是 ： " +  parentClassLoader.toString());

        ClassLoader topParentClassLoader = parentClassLoader.getParent();
        System.out.println("在往上一层的父加载器是： " + topParentClassLoader.toString());

    }
}
```
打印结果是：
```java
Exception in thread "main" ClassLoaderTest类的类加载器是： sun.misc.Launcher$AppClassLoader@18b4aac2
ClassLoaderTest类的类加载器的父类加载器是sun.misc.Launcher$ExtClassLoader@610455d6
java.lang.NullPointerException
	at JavaLangJarTest.ClassLoaderTest.main(ClassLoaderTest.java:22)
```
可知`AppClassLoader`的父类加载器是`ExtClassLoader`，而当打印`ExtClassLoader`的父加载器的时候抛出了空指针异常。诶？不是说每个加载器都有一个父加载器么？

实际上，从源码看:
ClassLoader之间的继承关系是这样的：

![ClassLoader_hierarchy](http://ovn0i3kdg.bkt.clouddn.com/classloader_hierarchy.png?imageView/2/w/400/q/90)

`getParent()`这个方法定义在`ClassLoader`类中，是这样定义的：
```java
public final ClassLoader getParent() {
    if (parent == null)
        return null;
    return parent;
}
```
可以看到，返回的是一个ClassLoader对象parent，parent的赋值是在ClassLoader对象的构造方法中，它有两种情况
1. 由外部类创建ClassLoader时直接指定一个ClassLoader为parent。
2. 由`getSystemClassLoader()`方法生成，也就是在`sun.misc.Laucher`通过`getClassLoader()`获取，也就是AppClassLoader。直白的说，一个ClassLoader创建时如果没有指定parent，那么它的parent默认就是AppClassLoader。


我们所关注的`AppClassLoader`和`ExtClassLoader`都和`Launcher`类相关，之前贴过代码，需要注意的是下面这几句话：
```java
ClassLoader extcl;

extcl = ExtClassLoader.getExtClassLoader();

loader = AppClassLoader.getAppClassLoader(extcl);
```
这说明`AppClassLoader`的parent是`ExtClassLoader`。

`ExtClassLoader`并没有直接找到对parent的赋值。它调用了它的父类也就是`URLClassLoder`的构造方法并传递了3个参数：
```java
public ExtClassLoader(File[] dirs) throws IOException {
            super(getExtURLs(dirs), null, factory);   
}
```
对应到`URLClassLoader`中的代码是：
```java
public  URLClassLoader(URL[] urls, ClassLoader parent,
                          URLStreamHandlerFactory factory) {
     super(parent);
}
```
所以，`ExtClassLoader`的parent是null。

讲到这里明白了：`AppClassLoader`的父加载器是`ExtClassLoader`，而`ExtClassLoader`的父加载器是null。但是我们在开头加载器加载顺序的时候说过，`BootstrapClassLoader`可以作为`ExtClassLoader`的父加载器，这又是怎么回事。

之前说过`BootstrapClassLoader`是由C++写的，是JVM的一部分，但是不是java类，所以无法再java代码中取得它的引用。JVM启动是首先启动`BootstrapClassLoader`加载器，像int.class、String.class这种都是由它加载的，这也就是为什么之前用`getParent`对int.class取得父加载器抛出异常的原因。之后，JVM初始化`sun.misc.Launcher`并创建`Extension ClassLoader`和`AppClassLoader`实例，将`BootstrapClassLoader`设置为`Extension ClassLoader`的父加载器，将`Extension ClassLoader`设置为`AppClassLoader`的父加载器，这也就是为什么说`BootstrapClassLoader`是`ExtClassLoader`的父加载器的原因。

## 双亲委托机制

一个类加载器查找class和resource时候，采用双亲委托机制？什么意思？
>  首先判断这个class是不是已经加载成功，如果没有的话它并不是自己进行查找，而是先通过父加载器，然后递归下去，直到Bootstrap ClassLoader，如果Bootstrap classloader找到了，直接返回，如果没有找到，则一级一级返回，最后到达自身去查找这些对象。这种机制就叫做双亲委托。

参考下面这个图

![双亲委托](http://ovn0i3kdg.bkt.clouddn.com/%E5%8F%8C%E4%BA%B2%E5%A7%94%E6%89%98.png)

一步步讲：
1. 一个AppClassLoader查找资源时，先看看缓存是否有，缓存有从缓存中获取，否则委托给父加载器。
2. 递归，重复第1步的操作。
3. 如果ExtClassLoader也没有加载过，则由Bootstrap ClassLoader出面，它首先查找缓存，如果没有找到的话，就去找自己的规定的路径下，也就是`sun.mic.boot.class`下面的路径。找到就返回，没有找到，让子加载器自己去找。
4. Bootstrap ClassLoader如果没有查找成功，则ExtClassLoader自己在`java.ext.dirs`路径中去查找，查找成功就返回，查找不成功，再向下让子加载器找。
5. ExtClassLoader查找不成功，AppClassLoader就自己查找，在`java.class.path路径下查找`。找到就返回。如果没有找到就让子类找，如果没有子类会怎么样？抛出各种异常。

可以看出，委托和查询的方向是相反的。

> 这时候又有一个问题了，JVM是如何判断类相同？有两个标准，一个是**类的全称**（类加载器实例+包名+类名）;另一个是**类加载器**。

为什么是这样的顺序，需要从源码中找答案，`loadClass`是类加载时候执行的方法，这个方法通过指定的全限定类名加载class，定义如下：
```java
protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
{
    synchronized (getClassLoadingLock(name)) {
        // 首先，检测是否已经加载
        Class<?> c = findLoadedClass(name);
        if (c == null) {
            long t0 = System.nanoTime();
            try {
                if (parent != null) {
                    //父加载器不为空则调用父加载器的loadClass
                    c = parent.loadClass(name, false);
                } else {
                    //父加载器为空则调用Bootstrap Classloader
                    c = findBootstrapClassOrNull(name);
                }
            } catch (ClassNotFoundException e) {
                // ClassNotFoundException thrown if class not found
                // from the non-null parent class loader
            }

            if (c == null) {
                // If still not found, then invoke findClass in order
                // to find the class.
                long t1 = System.nanoTime();
                //父加载器没有找到，则调用findclass
                c = findClass(name);

                // this is the defining class loader; record the stats
                sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                sun.misc.PerfCounter.getFindClasses().increment();
            }
        }
        if (resolve) {
            //调用resolveClass()
            resolveClass(c);
        }
        return c;
    }
}

```
可以看到，在加载的时候先用`findLoadedClass()`查看了缓存，缓存没有的话如果父加载器来执行，否则的话就用JVM内置加载器 `BootstrapClassLoader`来执行，这也解释了ExtClassLoader的parent为null,但仍然说Bootstrap ClassLoader是它的父加载器。 如果向上委托父加载器没有加载成功，则通过`findClass(String)`查找。如果class在上面的步骤中找到了，参数resolve又是true的话，那么又会调用`resolveClass(Class)`这个方法来生成最终的Class对象。 我们可以从源代码看出这个步骤。

这段源码体现的就是“双亲委托”机制。


## 自定义ClassLoader

经过前面的学习发现系统默认的ClassLoader都是通过路径来获取资源，那么我能不能自定义资源路径，再写一个自定义的ClassLoader来获取这些自定义的资源呢。可以，按照下面的步骤。
1. 编写一个类继承自ClassLoader抽象类。
2. 复写它的findClass()方法。 这个方法在编写自定义classloader的时候非常重要，它能将class二进制内容转换成Class对象，如果不符合要求的会抛出各种异常。
3. 在findClass()方法中调用defineClass()。

> 别忘记了，一个ClassLoader创建时如果没有指定parent，那么它的parent默认就是AppClassLoader。这样就能够保证它能访问系统内置加载器加载成功的class文件。

先将下面的.java文件编译成.class，然后放到指定目录下，如`D:\lib`,
```java
package com.frank.test;

public class Test {

    public void say(){
        System.out.println("Say Hello");
    }

}
```
然后自定义`ClassLoader`的示例：
```java
public class DiskClassLoader extends ClassLoader {

    private String mLibPath;

    public DiskClassLoader(String path) {
        // TODO Auto-generated constructor stub
        mLibPath = path;
    }

    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        // TODO Auto-generated method stub

        String fileName = getFileName(name);

        File file = new File(mLibPath,fileName);

        try {
            FileInputStream is = new FileInputStream(file);

            ByteArrayOutputStream bos = new ByteArrayOutputStream();
            int len = 0;
            try {
                while ((len = is.read()) != -1) {
                    bos.write(len);
                }
            } catch (IOException e) {
                e.printStackTrace();
            }

            byte[] data = bos.toByteArray();
            is.close();
            bos.close();

            return defineClass(name,data,0,data.length);

        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }

        return super.findClass(name);
    }

    //获取要加载 的class文件名
    private String getFileName(String name) {
        // TODO Auto-generated method stub
        int index = name.lastIndexOf('.');
        if(index == -1){
            return name+".class";
        }else{
            return name.substring(index)+".class";
        }
    }

}
```
> 这里有个疑问，为什么父类这么多的方法，偏偏只用重写findClass方法呢？因为JDK已经在loadClass方法中帮我们实现了ClassLoader搜索类的算法。当在loadClass方法中所有到类的时候，就会调动findClass方法，所以我们只需要调用这个方法而已，如果没有特殊要求，一般不建议重写loadClass搜索类的方法。

我们在`findClass()`方法中定义了查找class的方法，然后数据通过`defineClass()`生成了Class对象。

写一个测试程序：
```java
public class ClassLoaderTest {

    public static void main(String[] args) {
        // TODO Auto-generated method stub

        //创建自定义classloader对象。
        DiskClassLoader diskLoader = new DiskClassLoader("D:\\lib");
        try {
            //加载class文件
            Class c = diskLoader.loadClass("com.frank.test.Test");

            if(c != null){
                try {
                    Object obj = c.newInstance();
                    Method method = c.getDeclaredMethod("say",null);
                    //通过反射调用Test类的say方法
                    method.invoke(obj, null);
                } catch (InstantiationException | IllegalAccessException
                        | NoSuchMethodException
                        | SecurityException |
                        IllegalArgumentException |
                        InvocationTargetException e) {
                    // TODO Auto-generated catch block
                    e.printStackTrace();
                }
            }
        } catch (ClassNotFoundException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
}

```
运行，会答应出`Say Hello`。

> 本文大量参考 http://blog.csdn.net/briblue/article/details/54973413 这篇博客真的写的很好

> 为什么要使用这种双亲委托模式呢？
>
> 因为这样可以避免重复加载，当父亲已经加载了该类的时候，就没有必要子ClassLoader再加载一次。
> 考虑到安全因素，我们试想一下，如果不使用这种委托模式，那我们就可以随时使用自定义的String来动态替代java核心api中定义类型，这样会存在非常大的安全隐患【试想如果一个人写了一个恶意的基础类（如java.lang.String）并加载到JVM将会引起严重的后果】，而双亲委托的方式，java.lang.String永远是由根装载器来装载，避免以上情况发生。因为String已经在启动时被加载，所以用户自定义类是无法加载一个自定义的ClassLoader。

> 思考：假如我们自己写了一个java.lang.String的类，我们是否可以替换调JDK本身的类？
> 答案是否定的。我们不能实现。为什么呢？我看很多网上解释是说双亲委托机制解决这个问题，其实不是非常的准确。因为双亲委托机制是可以打破的，你完全可以自己写一个classLoader来加载自己写的java.lang.String类，但是你会发现也不会加载成功，具体就是因为针对java.*开头的类，JVM的实现中已经保证了必须由bootstrap来加载。


一个完整的ClassLoader工作过程是：

1、装载：查找和导入Class文件

2、链接：其中解析步骤是可以选择的

  1. 检查：检查载入的class文件数据的正确性

  2. 准备：给类的静态变量分配存储空间

  3. 解析：将符号引用转成直接引用

3、初始化：对静态变量，静态代码块执行初始化工作

装载解析之后接下来怎么走？在JVM中都有一个对应的`java.lang.Class`对象，提供了类结构信息的描述。数组，枚举及基本数据类型，甚至void都拥有对应的Class对象。Class类没有public的构造方法，Class对象是在装载类时由JVM通过调用类装载器中的`defineClass()`方法自动构造的。

至于如何初始化，也是有顺序的，看下面一篇博文。


### 线程上下文类加载器
英文名叫`Context ClassLoader`，从 JDK 1.2 开始引入，并不是真实存在的，只是一个概念，出现在`java.lang.Thread`中，
```java
public class Thread implements Runnable {

/* The context ClassLoader for this thread */
   private ClassLoader contextClassLoader;

   public void setContextClassLoader(ClassLoader cl) {
       SecurityManager sm = System.getSecurityManager();
       if (sm != null) {
           sm.checkPermission(new RuntimePermission("setContextClassLoader"));
       }
       contextClassLoader = cl;
   }

   public ClassLoader getContextClassLoader() {
       if (contextClassLoader == null)
           return null;
       SecurityManager sm = System.getSecurityManager();
       if (sm != null) {
           ClassLoader.checkClassLoaderPermission(contextClassLoader,
                                                  Reflection.getCallerClass());
       }
       return contextClassLoader;
   }
}
```
发现`contextClassLoader`只是一个成员变量，通过`setContextClassLoader`方法设置，通过`getContextClassLoader`获得。

如果没有通过 setContextClassLoader(ClassLoader cl)方法进行设置的话，线程将继承其父线程的上下文类加载器。Java 应用运行的初始线程的上下文类加载器是系统类加载器。在线程中运行的代码可以通过此类加载器来加载类和资源。

　　前面提到的类加载器的代理模式并不能解决 Java 应用开发中会遇到的类加载器的全部问题。Java 提供了很多服务提供者接口（Service Provider Interface，SPI），允许第三方为这些接口提供实现。常见的 SPI 有 JDBC、JCE、JNDI、JAXP 和 JBI 等。这些 SPI 的接口由 Java 核心库来提供，如 JAXP 的 SPI 接口定义包含在`javax.xml.parsers包`中。这些 SPI 的实现代码很可能是作为 Java 应用所依赖的 jar 包被包含进来，可以通过类路径（CLASSPATH）来找到，如实现了 JAXP SPI 的 Apache Xerces所包含的 jar 包。SPI 接口中的代码经常需要加载具体的实现类。如 JAXP 中的 `javax.xml.parsers.DocumentBuilderFactory`类中的 `newInstance()`方法用来生成一个新的 `DocumentBuilderFactory`的实例。这里的实例的真正的类是继承自 `javax.xml.parsers.DocumentBuilderFactory`，由 SPI 的实现所提供的。如在 Apache Xerces 中，实现的类是 `org.apache.xerces.jaxp.DocumentBuilderFactoryImpl`。而问题在于，SPI 的接口是 Java 核心库的一部分，是由引导类加载器来加载的；SPI 实现的 Java 类一般是由系统类加载器来加载的。引导类加载器是无法找到 SPI 的实现类的，因为它只加载 Java 的核心库。它也不能代理给系统类加载器，因为它是系统类加载器的祖先类加载器。也就是说，类加载器的代理模式无法解决这个问题。

线程上下文类加载器正好解决了这个问题。如果不做任何的设置，Java 应用的线程的上下文类加载器默认就是系统上下文类加载器。在 SPI 接口的代码中使用线程上下文类加载器，就可以成功的加载到 SPI 实现的类。线程上下文类加载器在很多 SPI 的实现中都会用到。

> 参考 http://www.cnblogs.com/doit8791/p/5820037.html





参考：
1. [深入分析Java ClassLoader原理](http://www.importnew.com/15362.html)
2. [java classLoader体系结构使用详解](http://blog.csdn.net/yaerfeng/article/details/24960121)
3. [Java ClassLoader机制](http://www.cnblogs.com/yangy608/archive/2011/07/23/2114900.html)
