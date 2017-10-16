# @FunctionalInterface注解

标注为`FunctionalInterface`的接口被称为函数式接口，该接口中只能有一个自定义的方法。如果一个接口只有一个方法，则编译器会认为这是这是一个函数式接口。例如：
```java
public interface FunctionalInterface{
  void f1();
}
```
上面这样写没有错，如果在接口上面加入`@FunctionalInterface`注解，那么该接口就会被强制要求符合函数式接口的规范，比如写成这样：
```java
@FunctionalInterface
public interface FunctionalInterface{
  void f1();
}
```

也可以添加Object类中的方法：
```java
@FunctionalInterface
public interface FunctionInterface {
    void  f1();
    String toString();
}
```
这里有一点需要说明，之前在整理《类、抽象类、内部类、接口》一文的时候说道，接口并没有继承Object类，但是实现了和Object中一样的方法集合。所以这里的toString严格来说并不是继承自Object类，而是接口本身实现的一套机制。这并不属于“自定义”方法的范畴，所以可以被添加到上例中。但是如果是自定义的方法，就不可以了，比如：
```java
@FunctionInterface
public interface FunctionInterface{
  void f1();
  void f1();
}
```
会产生`Invalid '@FunctionalInterface' annotation; FunctionalInterfaceTest is not a functional interface`错误。

在JDK 8中，可以看到有一些类已经应用了这个注解，比如`Runnable`：
```java
@FunctionalInterface
public interface Runnable {
    /**
     * When an object implementing interface <code>Runnable</code> is used
     * to create a thread, starting the thread causes the object's
     * <code>run</code> method to be called in that separately executing
     * thread.
     * <p>
     * The general contract of the method <code>run</code> is that it may
     * take any action whatsoever.
     *
     * @see     java.lang.Thread#run()
     */
    public abstract void run();
}
```
