# Error
结构如下：

![Error](http://ovn0i3kdg.bkt.clouddn.com/Error.png)

和隔壁`Exception`一样，方法只定义了四种构造方法

### public class Error extends Throwable

类声明， `Error`类直接继承了`Throwable`类。所以在异常机制中，`Error`和`Exception`是同等地位的，两者不同在哪里？注意到，源码中对`Error`有一段注释解释了`Error`类的用途：
```java
/**
 * An {@code Error} is a subclass of {@code Throwable}
 * that indicates serious problems that a reasonable application
 * should not try to catch. Most such errors are abnormal conditions.
 * The {@code ThreadDeath} error, though a "normal" condition,
 * is also a subclass of {@code Error} because most applications
 * should not try to catch it.
 * <p>
 * A method is not required to declare in its {@code throws}
 * clause any subclasses of {@code Error} that might be thrown
 * during the execution of the method but not caught, since these
 * errors are abnormal conditions that should never occur.
 *
 * That is, {@code Error} and its subclasses are regarded as unchecked
 * exceptions for the purposes of compile-time checking of exceptions.
 *
 * @author  Frank Yellin
 * @see     java.lang.ThreadDeath
 * @jls 11.2 Compile-Time Checking of Exceptions
 * @since   JDK1.0
 */
```
第一段话的意思是：`Error`用于指示合理的应用程序不应该试图捕获的严重问题。大多数这样的错误都是异常条件。虽然`ThreadDeath`错误是一个“正规”的条件，但它也是`Error`的子类，因为大多数应用程序都不应该试图捕获它。在执行该方法期间，无需在其 throws 子句中声明可能抛出但是未能捕获的 Error 的任何子类，因为这些错误可能是再也不会发生的异常条件。

大概的意思就是，`Error`总是不可控的（unchecked），经常用来用于表示系统错误或低层资源的错误，如何可能的话，应该在系统级被捕捉。


很少用到`Error`类及其子类，在看源码的过程中有遇到`AssertionError`。
