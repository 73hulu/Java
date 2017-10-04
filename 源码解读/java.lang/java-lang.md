# java.lang

Java语言包（java.lang）是Java语言体系中其他所有类库的基础（没有它什么都干不了），已经**内嵌**到Java虚拟机中，而且以**对象的形式建好了**。所以在使用的时候不需要再使用`import`将其导入，可以直接使用java.lang包中所有的类及直接引用某个类中的常量、变量和操作方法。

其中包含的类及相应的功能说明如下：

| 类名 | 功能 |
| :------------- | :------------- |
| Boolean |   	封装了boolean类型的值以及一些操作该类型的方法     |
|Byte   |  	封装了byte类型的值以及一些操作该类型的方法 |
|Character   |  	封装了char类型的值以及一些操作该类型的方法 |
|Double   |封装了double类型的值以及一些操作该类型的方法   |
|Float	|封装了float类型的值以及一些操作该类型的方法   |  
|Integer   |  	封装了int类型的值以及一些操作该类型的方法 |
|Long   |	封装了long类型的值以及一些操作该类型的方法   |
|Short   |封装了short类型的值以及一些操作该类型的方法   |
|String   |  	封装了与字符串类型相关的操作方法|
|Void   |	表示对Java中的void关键字的声明，这个类不可以实例   |
|Class| 用于描述正在运行的Java应用程序中的类和接口的状态  |
|ClassLoader | 用于加载类的对象|
|Enum   | 用于定义枚举类型   |
|Math   |  实现基础的数学计算 |
|Object   |  是所有Java类的根类 |
|Package   |  	封装了有关Java包的实现和规范的版本信息 |
|Runtime   |  Runtime类对象使Java应用程序与其运行环境相连接 |
|  StrictMath |   用于实现基本的数学运算|
|StringBuffer   | 用于可变字符串的操作  |
|StringBuilder   |  创建可变的字符串对象 |
|System   |封装了一些与Java虚拟机系统相关的方法|
|Thread   |  	创建和控制线程 |
|ThreadGroup   |  	创建和控制线程组 |
|Throwable   |  定义了Java中的所有错误或者异常的父类 |
|Process   |  	定义一个进程process对象，通过Runtime类中的exec方法启动该进程对象|

其中包含的主要接口和相应的功能说明如下：

| 接口 | 功能 |
| :------------- | :------------- |
| Appendable       |用于追加字符串      |
|Cloneable   |用于复制类对象   |
|Runnable   |  用于实现类对象具有线程功能 |
|Comparable   |用于类对象的排序   |
