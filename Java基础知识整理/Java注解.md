# Java注解
Java注解是JDK5引入的功能。

### Java元注解
1. `@Target`：
2. `@Retention`：
3. `@Documented`：
4. `@Inherited`：

### Java内建注解
Java提供了三种内建注解。
1. `@Override`：当我们想要复写父类中的方法时，我们需要使用该注解去告知编译器我们想要复写这个方法。这样一来当父类中的方法移除或者发生更改时编译器将提示错误信息。源码如下：
```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {
}
```
2. `@Deprecated`：当我们希望编译器知道某一方法不建议使用时，我们应该使用这个注解。Java在javadoc中推荐使用该注解，我们应该提供为什么该方法不推荐使用以及替代的方法。源码如下：
```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(value={CONSTRUCTOR, FIELD, LOCAL_VARIABLE, METHOD, PACKAGE, PARAMETER, TYPE})
public @interface Deprecated {
}
```
3. `@SuppressWarnings`：这个仅仅是告诉编译器忽略特定的警告信息，例如在泛型中使用原生数据类型。它的保留策略是SOURCE（译者注：在源文件中有效）并且被编译器丢弃。源码如下：
```java
@Target({TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE})
@Retention(RetentionPolicy.SOURCE)
public @interface SuppressWarnings {
    String[] value();
}
```
内部有一个String数组，主要接受值有：
* `deprecation`：使用了不赞成使用的类或方法时的警告；
* `unchecked`：执行了未检查的转换时的警告，例如当使用集合时没有用泛型 (Generics) 来指定集合保存的类型;
* `fallthrough`：当 Switch 程序块直接通往下一种情况而没有 Break 时的警告;
* `path`：在类路径、源文件路径等中有不存在的路径时的警告;
* `serial`：当在可序列化的类上缺少 serialVersionUID 定义时的警告;
* `finally`：任何 finally 子句不能正常完成时的警告;
* `all`：关于以上所有情况的警告。

参考：
* [深入理解Java注解类型(@Annotation)](http://blog.csdn.net/javazejian/article/details/71860633?t=1495827334875)
* [Java注解教程及自定义注解](http://www.importnew.com/17413.html)
* [Java中三种简单注解介绍和代码实例](http://www.jb51.net/article/55370.htm)
