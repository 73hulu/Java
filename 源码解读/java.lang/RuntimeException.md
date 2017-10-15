# RuntimeException
结构如下：

![RuntimeException](http://ovn0i3kdg.bkt.clouddn.com/RuntimeException.png)

### public class RuntimeException extends Exception
类声明， `RuntimeException`继承了`Exception`类，四种构造方法全部调用了父类`Exception`的四种构造方法。

源码对`RuntimeException`的解释如下：
> /**
> * {@code RuntimeException} is the superclass of those
> * exceptions that can be thrown during the normal operation of the
> * Java Virtual Machine.
> *
> * <p>{@code RuntimeException} and its subclasses are <em>unchecked
> * exceptions</em>.  Unchecked exceptions do <em>not</em> need to be
> * declared in a method or constructor's {@code throws} clause if they
> * can be thrown by the execution of the method or constructor and
> * propagate outside the method or constructor boundary.
> *
> * @author  Frank Yellin
> * @jls 11.2 Compile-Time Checking of Exceptions
> * @since   JDK1.0
> */

可以看到`RuntimeException`的子在JVM运行的时候会被抛出，它们属于`unchecked exceptions`，这类异常需要在子句`throws`中声明。
