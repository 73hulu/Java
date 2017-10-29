# System

java系统类`System`是一个不可实例化的类，提供了一些系统属性和方法。我们经常用到`System.out.println()`，那么其中的`out`是一个内部类么？看了这个类就能得到答案了。类的结构如下：

![System](http://ovn0i3kdg.bkt.clouddn.com/System.png)

内容还挺多的，但是我们只用记住几个常用的就可以了。

### 属性字段
```java
//标准输入流  
public final static InputStream in;  
//标准输出流  
public final static PrintStream out;  
//标准错误流
public final static PrintStream err;  
```
这三个就是我们常用都到的`System.in`、`System.out`、`System.err`，他们都不是内部类，而是字段变量。他们都是IO类，并且已经处于打开状态，等待着接收和输出内容。

###   public static native void arraycopy(Object src,  int  srcPos, Object dest, int destPos, int length);
这是经常被用到数组拷贝方法。数组拷贝有很多种方法，比如`String`类的`getChars(char dst[], int dstBegin)`、`Arrays`类的`copyOf(char[] origin, int len)`或`copyOfRange(char[] origin, int len)`，但是实际上三者都是直接调用了`System`类的`arraycopy`方法。

遇到过一道题目，问的是数组拷贝方法中哪种方法效率最高。候选的有for循环方法、`String`的`getChars`方法，`Arrays`的`copyOf`方法和`System`的`arraycopy`方法。在大规模复制的情况下，当然是`System`的`arraycopy`方法效率最高。为什么？且不说其他两个（除了for循环）是调用的`System.arraycopy`方法，它本身是一个native方法，是C++实现的，所以这个效率更高。


### public static native long currentTimeMillis();
该方法和`Date`类中`getTime`方法完全是一样的，如果只是需要毫秒数，这样的调用也是很方便的。但是需要注意的是`currentTimeMillis`并不是直接拿到了getTime的结果！`currentTimeMillis`是一个本地方法，返回的是**操作系统的时间**，由于有的操作系统时间的最小精确度是10毫秒所以这个方法可能会导致一些偏差。

### public static String getProperty(String key){...}
我们通过调用这个方法，在参数中输入键的字符串获取系统的属性。哪些系统属性呢？如下面。
<table summary="Shows property keys and associated values">
    <tr><th>Key</th>
        <th>Description of Associated Value</th></tr>
    <tr><td><code>java.version</code></td>
        <td>Java Runtime Environment version</td></tr>
    <tr><td><code>java.vendor</code></td>
        <td>Java Runtime Environment vendor</td></tr>
    <tr><td><code>java.vendor.url</code></td>
        <td>Java vendor URL</td></tr>
    <tr><td><code>java.home</code></td>
        <td>Java installation directory</td></tr>
    <tr><td><code>java.vm.specification.version</code></td>
        <td>Java Virtual Machine specification version</td></tr>
    <tr><td><code>java.vm.specification.vendor</code></td>
        <td>Java Virtual Machine specification vendor</td></tr>
    <tr><td><code>java.vm.specification.name</code></td>
        <td>Java Virtual Machine specification name</td></tr>
    <tr><td><code>java.vm.version</code></td>
        <td>Java Virtual Machine implementation version</td></tr>
    <tr><td><code>java.vm.vendor</code></td>
        <td>Java Virtual Machine implementation vendor</td></tr>
    <tr><td><code>java.vm.name</code></td>
        <td>Java Virtual Machine implementation name</td></tr>
    <tr><td><code>java.specification.version</code></td>
        <td>Java Runtime Environment specification  version</td></tr>
    <tr><td><code>java.specification.vendor</code></td>
        <td>Java Runtime Environment specification  vendor</td></tr>
    <tr><td><code>java.specification.name</code></td>
        <td>Java Runtime Environment specification  name</td></tr>
    <tr><td><code>java.class.version</code></td>
        <td>Java class format version number</td></tr>
    <tr><td><code>java.class.path</code></td>
        <td>Java class path</td></tr>
    <tr><td><code>java.library.path</code></td>
        <td>List of paths to search when loading libraries</td></tr>
    <tr><td><code>java.io.tmpdir</code></td>
        <td>Default temp file path</td></tr>
    <tr><td><code>java.compiler</code></td>
        <td>Name of JIT compiler to use</td></tr>
    <tr><td><code>java.ext.dirs</code></td>
        <td>Path of extension directory or directories
            <b>Deprecated.</b> <i>This property, and the mechanism
               which implements it, may be removed in a future
               release.</i> </td></tr>
    <tr><td><code>os.name</code></td>
        <td>Operating system name</td></tr>
    <tr><td><code>os.arch</code></td>
        <td>Operating system architecture</td></tr>
    <tr><td><code>os.version</code></td>
        <td>Operating system version</td></tr>
    <tr><td><code>file.separator</code></td>
        <td>File separator ("/" on UNIX)</td></tr>
    <tr><td><code>path.separator</code></td>
        <td>Path separator (":" on UNIX)</td></tr>
    <tr><td><code>line.separator</code></td>
        <td>Line separator ("\n" on UNIX)</td></tr>
    <tr><td><code>user.name</code></td>
        <td>User's account name</td></tr>
    <tr><td><code>user.home</code></td>
        <td>User's home directory</td></tr>
    <tr><td><code>user.dir</code></td>
        <td>User's current working directory</td></tr>
</table>

有些情况下会用到。比如我们在操作文件的时候可能需要用到我们的当前工作目录。可以使用这个方法来获取：
```java
public static void main(String[] args) {
    String dirPath = System.getProperty("user.dir");
    System.out.println(dirPath);
}
```

### public static void gc(){...}
调用 gc 方法暗示着 Java 虚拟机做了一些努力来回收未用对象或失去了所有引用的对象，以便能够快速地重用这些对象当前占用的内存。当控制权从方法调用中返回时，虚拟机已经尽最大努力从所有丢弃的对象中回收了空间。定义如下：
```java
public static void gc() {
    Runtime.getRuntime().gc();
}
```
需要注意的是，实际上我们并不一定需要调用`gc()`方法，让编译器自己去做就好了。另外需要注意人为调用`gc()`方法并不会立即触发垃圾回收，什么时候回收还是JVM说了算。

### public static void exit(int status){...}
退出虚拟机。定义如下：
```java
public static void exit(int status) {
    Runtime.getRuntime().exit(status);
}
```
`exit(int)`方法终止当前正在运行的 Java 虚拟机，参数解释为状态码。根据惯例，**非 0**的状态码表示异常终止。**0**表示正常中止。而且，该方法永远不会正常返回。 这是唯一一个能够退出程序并不执行finally的情况。例如：
```java
public static void main(String[] args) {  
    try {  
        System.out.println("this is try");  
        System.exit(0);  
    } catch (Exception e) {  
        // TODO Auto-generated catch block  
        e.printStackTrace();  
    } finally {  
        System.out.println("this is finally");  
    }  

}  
```
上面这段程序并不会指定finally中的语句。因为退出虚拟机会直接杀死整个程序，已经不是从代码层面来终止程序了，所以finally不会执行。

参考
* [java——深度解析System系统类](http://blog.csdn.net/quinnnorris/article/details/71077893?utm_source=gold_browser_extension)
