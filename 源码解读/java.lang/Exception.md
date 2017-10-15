# Exception

我们经常自定义的类一般都是直接继承这个类，该类结果如下：

![Exception](http://ovn0i3kdg.bkt.clouddn.com/Exception.png)

结构简单哭了，只有四种构造方法。

### public class Exception extends Throwable
类声明，可以看到Exception类直接继承了Throwable类。并且`Exception`定义的四种构造方法实际上是直接调用了父类的四种构造方法。

源码中有段注释解释了这个类的用途：
```java
/**
 * The class {@code Exception} and its subclasses are a form of
 * {@code Throwable} that indicates conditions that a reasonable
 * application might want to catch.
 *
 * <p>The class {@code Exception} and any subclasses that are not also
 * subclasses of {@link RuntimeException} are <em>checked
 * exceptions</em>.  Checked exceptions need to be declared in a
 * method or constructor's {@code throws} clause if they can be thrown
 * by the execution of the method or constructor and propagate outside
 * the method or constructor boundary.
 *
 * @author  Frank Yellin
 * @see     java.lang.Error
 * @jls 11.2 Compile-Time Checking of Exceptions
 * @since   JDK1.0
 */

```
首先第一段里说了`Exception`是合理的应用程序想要捕获的条件。

第二段里说了，`Exception`子类中，`RuntimeException`是那些可能在 Java 虚拟机正常运行期间抛出的异常的超类，`RuntimeException`的子类需要被写到`throws`子句中，而除此之外的`Exception`子类，他们是`checked exceptions`。
