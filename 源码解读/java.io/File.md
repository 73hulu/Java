# File

流的本质是对文件的处理，所以在学习具体的流之前，先要学习`File`类。

![File](http://ovn0i3kdg.bkt.clouddn.com/File_1.png)
![File](http://ovn0i3kdg.bkt.clouddn.com/File_2.png)

## public class File implements Serializable, Comparable<File>
类声明，可以看到`File`实现了`Serializable`接口和`Comparable`接口，所以这个类中一定会定义一个`serialVersionUID`。

## 静态构造块
这个类中有一个静态构造块，定义如下：
```java
static {
     try {
         sun.misc.Unsafe unsafe = sun.misc.Unsafe.getUnsafe();
         PATH_OFFSET = unsafe.objectFieldOffset(
                 File.class.getDeclaredField("path"));
         PREFIX_LENGTH_OFFSET = unsafe.objectFieldOffset(
                 File.class.getDeclaredField("prefixLength"));
         UNSAFE = unsafe;
     } catch (ReflectiveOperationException e) {
         throw new Error(e);
     }
 }

```
其中`PATH_OFFSET`，`PREFIX_LENGTH_OFFSET`，`UNSAFE`是声明在静态代码块之前的类变量，定义如下：
```java
private static final long PATH_OFFSET;
private static final long PREFIX_LENGTH_OFFSET;
private static final sun.misc.Unsafe UNSAFE;
```
这段代码其实没能看懂具体有什么作用o(╯□╰)o

## 构造方法
`File`类定义了6种构造方法，其中有两种是私有的。

## public File(String pathname){...}
这是一个最常用的公共构造方法。用指定的路径`pathname`创建文件对象。定义如下：
```java
public File(String pathname) {
    if (pathname == null) {
        throw new NullPointerException();
    }
    this.path = fs.normalize(pathname);
    this.prefixLength = fs.prefixLength(this.path);
}
```
当参数为null时会抛出空指针异常。否则将文件路径转为抽象路径，什么是抽象路径？
输入参数pathname是文件路径及文件名的字符串表达形式，不同的操作系统有不同的表达形式。这个类提供了一个“抽象路径”，这个pathname会转化为abstract pathname，即与系统无关的文件名表达形式，并用`path`、`prefixLength`两个变量存储输入参数的`pathname`转化后的变量。下面就是这两个变量的声明。

```java
private final String path;
private final transient int prefixLength;
```
一个抽象路径包含两个部分：
1. 一个可选的依赖系统的前缀字符串，例如一个用于UNIX根目录的磁盘驱动器说明符“/”，或者一个Microsoft Windows UNC路径名的“\\\”
2. 零个或多个字符串名称的序列

抽象路径名中的第一个名称可能是目录名称，或者在Microsoft Windows UNC路径名的情况下是主机名。 抽象路径名中的每个后续名称表示一个目录; 最后一个名称可以表示目录或文件。 空的抽象路径名没有前缀和空名称序列。

字符串文件路径是如何转化为抽象路径的呢？其过程是方法`normalize`的具体实现。`normalize`是一个抽象方法。在mac os x系统性下，在dubug模式下可以发现最后执行的是`FileSystem`的实现类`UnixFileSystem`重写的`normalize`方法。可见，将路径名字符串转换为抽象路径名或从抽象路径名转换本质上是系统依赖的。它会自动转化为某个操作系统的路径以及去除重复和多余的分隔符等。 下面是`UnixFileSystem`类中的`normalize`方法的定义：
```java
public String normalize(String pathname) {
    int n = pathname.length();
    char prevChar = 0;
    for (int i = 0; i < n; i++) {
        char c = pathname.charAt(i);
        if ((prevChar == '/') && (c == '/'))
            return normalize(pathname, n, i - 1);
        prevChar = c;
    }
    if (prevChar == '/') return normalize(pathname, n, n - 1);
    return pathname;
}
```
这个方法只会执行一次，为什么？因为`path`被`final`修饰，所以`File`类的实例是不可变的，也就是说一旦创建，由`File`对象表示的抽象路径永远不会改变。

> 当抽象路径名转换为路径名字符串时，每个名称将与默认分隔符字符的单个副本分隔开。 默认名称分隔符由系统属性`file.separator`定义，并在该类的公共静态字段`separator`和`separatorChar`中可用。
> 当路径名字符串转换为抽象路径名时，其中的名称可能由默认名称分隔符或由底层系统支持的任何其他名称 - 分隔符分隔。

路径名，无论是抽象路径还是字符串形式，都可以是绝对路径或相对路径。
>  关于路径的具体解释可以看另一篇博客。

由于存在平台的差异，所以一般我们不把字符串直接写成`c:\\zuidaima\\1.txt`这种形式，而是采用`separator`跨平台分隔符，例如：
```java
File f3 =new File("c:"+File.separator+"abc");
```


### public File(String parent, String child){...}
该方法用于从父路径名字符串和子路径名字符串创建新的File实例。
```java
public File(String parent, String child) {
    if (child == null) {
        throw new NullPointerException();
    }
    if (parent != null) {
        if (parent.equals("")) {
            this.path = fs.resolve(fs.getDefaultParent(),
                                   fs.normalize(child));
        } else {
            this.path = fs.resolve(fs.normalize(parent),
                                   fs.normalize(child));
        }
    } else {
        this.path = fs.normalize(child);
    }
    this.prefixLength = fs.prefixLength(this.path);
}
```
如果parent为null，那么将创建新的File实例，就像在给定子路径名字符串中调用单参数File构造函数一样。否则，将使用父路径名字符串表示目录，并将子路径名字符串用于表示目录或文件。 如果子路径名字符串为绝对值，则以系统相关方式将其转换为相对路径名。 如果parent是空字符串，则通过将子代码转换为抽象路径名并根据系统相关的默认目录解析结果来创建新的File实例。 否则，每个路径名字符串将转换为抽象路径名，并针对父对象解析子抽象路径名。

讲了这么多，实际上就是将一个完整的路径分开写了而已。比如调用上一个构造函数创建文件`File f1 =new File("c:\\zuidaima\\1.txt");`如果想要在同一个目录下创建另一个txt文件，那么就可以用`File f2 =new File("c:\\zuidaima","2.txt"); `这种方式。


### public File(File parent, String child){...}
和上面一个构造函数好像，注意这个方法的第一个参数是`File`类型。
```java
public File(File parent, String child) {
    if (child == null) {
        throw new NullPointerException();
    }
    if (parent != null) {
        if (parent.path.equals("")) {
            this.path = fs.resolve(fs.getDefaultParent(),
                                   fs.normalize(child));
        } else {
            this.path = fs.resolve(parent.path,
                                   fs.normalize(child));
        }
    } else {
        this.path = fs.normalize(child);
    }
    this.prefixLength = fs.prefixLength(this.path);
}
```
下面是一个使用这个构造函数的例子：
```java
File f3 =new File("c:"+File.separator+"abc");//separator 跨平台分隔符  
File f4 =new File(f3,"3.txt");  
```

### public File(URI uri){...}
将URI 转换为一个抽象路径名来创建一个新的 File 实例。URI 的具体形式与系统有关，因此，由此构造方法执行的转换也与系统有关。定义如下：
```java
public File(URI uri) {
    // Check our many preconditions
    if (!uri.isAbsolute())
        throw new IllegalArgumentException("URI is not absolute");
    if (uri.isOpaque())
        throw new IllegalArgumentException("URI is not hierarchical");
    String scheme = uri.getScheme();
    if ((scheme == null) || !scheme.equalsIgnoreCase("file"))
        throw new IllegalArgumentException("URI scheme is not \"file\"");
    if (uri.getAuthority() != null)
        throw new IllegalArgumentException("URI has an authority component");
    if (uri.getFragment() != null)
        throw new IllegalArgumentException("URI has a fragment component");
    if (uri.getQuery() != null)
        throw new IllegalArgumentException("URI has a query component");
    String p = uri.getPath();
    if (p.equals(""))
        throw new IllegalArgumentException("URI path component is empty");

    // Okay, now initialize
    p = fs.fromURIPath(p);
    if (File.separatorChar != '/')
        p = p.replace('/', File.separatorChar);
    this.path = fs.normalize(p);
    this.prefixLength = fs.prefixLength(this.path);
}
```
这个方法到现在都没有用到过，所以暂时不知道是什么意思，先不看了。


### private File(String pathname, int prefixLength){...}
这是一个内部的构造方法，用来已经规范化的路径名字符串的内部构造函数。定义如下：
```java
private File(String pathname, int prefixLength) {
   this.path = pathname;
   this.prefixLength = prefixLength;
}
```

### private File(String child, File parent){...}
另一个内部的构造方法，定义如下：
```java
private File(String child, File parent) {
   assert parent.path != null;
   assert (!parent.path.equals(""));
   this.path = fs.resolve(parent.path, child);
   this.prefixLength = parent.prefixLength;
}
```


其他方法都是一些路径访问、文件属性方法，不用太深究了，下面是API的总结，会用就行：

| 方法 | 描述 |
| :------------- | :------------- |
| public String getName() | 返回由此抽象路径名表示的文件或目录的名称 |
|public String getParent()    | 返回此抽象路径名的父路径名字符串，如果此路径名没有指定父目录，则返回null  |
|public File getParentFile()   |返回此抽象路径名的父路径名的抽象路径名，如果此路名没有指定父目录，则返回null   |
|**public String getPath()  ** | 将次抽象路径名转化为一个路径名字符串  |
|public boolean isAbsolute()   | 测试此抽象路径名是否为绝对路径名 |
|**public String getAbsolutePath() ** | 返回抽象路径名的绝对路径名字符串  |
|**public String getCanonicalPath() throws IOException**  |  返回抽象路径名的规范路径名字符串  |
|public boolean canExecute() |  判断文件是否可执行 |
|public boolean canRead()   |   测试应用程序是否可以读取此抽象路径名表示的的文件|
| public boolean canWrite() | 测试应用程序是否可以修改此抽象路径名表示的文件  |
|**public boolean exists()  ** | 测试此抽象路径名表示的文件或目录是否存在  |
|public boolean isDirectory()   |  测试此抽象路径名表示的文件是否是一个目录 |
|public boolean isFile()   |测试此抽象路径名表示的文件是否是一个标准文件   |
|public long lastModify()   |  返回此抽象路径名表示的文件最后一次被需改的时间 |
|public long length()   |  返回由此抽象路径名表示的文件的长度 |
|**public boolean createNewFile() throws IOException  ** |  当且仅当不存在具有此抽象路径名指定的文件时，原子地创建由此抽象路径名执行的一个新的空文件，注意这个方法的返回值是boolean值 |
|public boolean delete()   | 删除此抽象路径名表示的文件或目录  |
|public void deleteOnExit()   |   在虚拟机终止时，请求删除此抽象路径名表示的文件或目录|
|public String[] list()   |返回由此抽象路径名表示的目录中的文件和目录的名称锁组成的字符串数组   |
|public String[] list(FilenameFilter filter)   |  返回由包含在目录中的文件和目录的名称所组成的字符串数组，这一目录是通过满足指定过滤器的抽象路径名来表示的。|
| public File[] listFiles()  |  返回表示此抽象路径名所表示目录中的文件和目录的抽象路径名数组，这些路径名满足特定过滤器。 |
|public boolean mkdir()   | 创建此抽象路径名指定的目录。只能创建多级目录  |
|  public boolean mkdirs() |  创建此抽象路径名指定的目录，包括创建必需但不存在的父目录。创建多级目录 |
|	public boolean renameTo(File dest） |    重新命名此抽象路径名表示的文件。|
|public boolean setLastModified(long time)   |  设置由此抽象路径名所指定的文件或目录的最后一次修改时间。 |
|public boolean setReadOnly()   |  标记此抽象路径名指定的文件或目录，以便只可对其进行读操作。 |
|public static File createTempFile(String prefix, String suffix, File directory) throws IOException   | 在指定目录中创建一个新的空文件，使用给定的前缀和后缀字符串生成其名称。  |
|  public static File createTempFile(String prefix, String suffix) throws IOException |   在默认临时文件目录中创建一个空文件，使用给定前缀和后缀生成其名称。|
|public int compareTo(File pathname)   |  按字母顺序比较两个抽象路径名。 |
|public int compareTo(Object o)   |   按字母顺序比较抽象路径名与给定对象。|
|public boolean equals(Object obj)   |  测试此抽象路径名与给定对象是否相等。 |
|public String toString()   |  返回此抽象路径名的路径名字符串。  |

> 上面简单罗列了一下常见的用法，一些常用的操作例子可以参考：[Java File操作汇总](http://blog.csdn.net/qingdujun/article/details/41223841)


其中有些方法需要明确概念、适用场景和雷区。

### `getPath`、`getAbsolutePath`和`getCanonicalPath`的区别
这三个方法都不管文件是不是真是存在，只要有文件引用就可以了。前两个的区别很好解释：`getPath`输出的是创建的时候用的路径字符串，`getAbsolutePath`输出的是绝对路径。`getCanonicalPath`方法不但输出全路径，而且把一些`..`、`.`这样的符号解析出来了。看下面的例子。
```Java
File file = new File("..\\src\\test1.txt");
System.out.println(file.getPath());
System.out.println(file.getAbsolutePath());
try {
    System.out.println(file.getCanonicalPath());

}catch (Exception e){
    e.getStackTrace();
}
```
输出的结果是：
```Java
..\\src\\test1.txt
D:\workspace\test\..\src\test1.txt
D:\workspace\src\test1.txt
```
输出结果可以看到三者明显的不同。另外这篇博文http://www.blogjava.net/dreamstone/archive/2007/08/08/134968.html 中有写一个由windows和linux大小写敏感问题造成结果差异，可以看一下。


### `renameTo`方法的应用
这个方法名字上看就是"重命名"的意思，但是这个方法经常用来拷贝文件。仔细看方法签名，`public boolean renameTo(File dest）`，参数是目标文件，难道重命名参数不应该是String么？而且返回值是`boolean`，所以还存在重命名不成功的时候。看下面这个测试：
```Java
public static void main(String[] args) {
    File source = new File("/Users/baoxiaofang/Documents/aaa.txt");
    if (!source.exists()){
        try {
            source.createNewFile();

        }catch (Exception e){
            e.getStackTrace();
        }
    }
    File dest1 = new File("/Users/baoxiaofang/Documents/bbb.txt");
    File dest2 = new File("/Users/baoxiaofang/Documents/ccc.txt");

    System.out.println("dest1 is exist? "  + dest1.exists());
    System.out.println("dest2 is exist? " + dest2.exists());

    System.out.println("source rename to dest1 " + source.renameTo(dest1));
    System.out.println("source rename to dest2 " + source.renameTo(dest2));
}
```
执行结果是：
```Java
dest1 is exist? false
dest2 is exist? true
source rename to dest1 true
source rename to dest2 false
```
该程序中首先保证了源文件source一定存在，然后分给renameTo到一个不存在的文件bbb.txt，和一个已存在的文件ccc.txt，结果发现，前者成功而后者不成功，并且最后看文件夹发现有一个bbb.txt和ccc.txt，而原来的aaa.txt不见了。所以可以得出结论：使用renameTo重命名文件，首先要保证源文件存在（这个可以做实验验证一下确实是这样），不管这个源文件是一个“文件”还是一个“目录”。最重要的一点是，要保证参数dest是一个不存在的对象。满足以上两点才能重命名成功。这个方法和linux下的`mv`命令不是一样。

但是这个方法有人做过测试，很不靠谱，参考http://www.iteye.com/topic/149328， 所以拷贝文件还是用`FileUtilsc.copyFileToDirectory(File,File)`比较好。
