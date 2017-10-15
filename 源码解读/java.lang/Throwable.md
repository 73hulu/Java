# Throwable
Throwable是Java中异常类的顶级父类，有必要看看源码实现。类的结构如下：
![Throwable](http://ovn0i3kdg.bkt.clouddn.com/Throwable.png)

### public class Throwable implements Serializable
首先看到类的声明，实现了`Serializable`，表示可以序列化，实现这个接口的类通常还会有一个常量`serialVersionUID`。

### private transient Object backtrace;
这个变量上面有一行注释`Native code saves some indication of the stack backtrace in this slot.`，表明本地代码在这个变量中保存了堆栈回溯的一些指示。注意到的是这个变量被关键词`transient`修饰，表示在序列化的时候序列该变量。`transient`一般用来修饰一些敏感的变量，比如用户的密码，内存堆栈信息等。

### private String detailMessage;
这个变量描述此异常的信息，比如如果抛出`FileNotFoundException`，那么这个变量就会是未找到的文件的名字。

### private Throwable cause = this;
表示当前异常由那个Throwable引起，在初始化的时候指向了自己本身。如果为null表示此异常不是由其他Throwable引起的或者原因未知，如果此对象与自己相同,表明此异常的起因对象还没有被初始化

### private static final StackTraceElement[] UNASSIGNED_STACK = new StackTraceElement[0];
对这个变量的解释是:空堆栈的共享值。而对于`StackTraceElement`这个类，其结构如下：

![StackTraceElement](http://ovn0i3kdg.bkt.clouddn.com/StackTraceElement.png)

可以看到其定义的成员变量：`declaringClass`、`methodName`、`fileName`、`lineNumber`共同定义堆栈轨迹。

### private StackTraceElement[] stackTrace = UNASSIGNED_STACK;
这是真正保存栈轨迹的地方，初始化成上面提到的`UNASSIGNED_STACK`，可以使用实例`getStackTrace()`获取该数组，使用实例方法`etStackTrace`或者`fillInStackTrace`来填充数组。

### 私有静态内部类SentinelHolder
`Throwable`中定义了一个内部类，如下：
```Java
private static class SentinelHolder {
    /**
     * {@linkplain #setStackTrace(StackTraceElement[]) Setting the
     * stack trace} to a one-element array containing this sentinel
     * value indicates future attempts to set the stack trace will be
     * ignored.  The sentinal is equal to the result of calling:<br>
     * {@code new StackTraceElement("", "", null, Integer.MIN_VALUE)}
     */
    public static final StackTraceElement STACK_TRACE_ELEMENT_SENTINEL =
        new StackTraceElement("", "", null, Integer.MIN_VALUE);

    /**
     * Sentinel value used in the serial form to indicate an immutable
     * stack trace.
     */
    public static final StackTraceElement[] STACK_TRACE_SENTINEL =
        new StackTraceElement[] {STACK_TRACE_ELEMENT_SENTINEL};
}
```
这个类暂时还没有搞清楚是怎么回事。

###  private static final List<Throwable> SUPPRESSED_SENTINEL = Collections.unmodifiableList(new ArrayList<Throwable>(0));

###  private List<Throwable> suppressedExceptions = SUPPRESSED_SENTINEL;
这个变量是从JDK1.7开始加入的。

### 一些静态常量
```java
/** Message for trying to suppress a null exception. */
private static final String NULL_CAUSE_MESSAGE = "Cannot suppress a null exception.";

/** Message for trying to suppress oneself. */
private static final String SELF_SUPPRESSION_MESSAGE = "Self-suppression not permitted";

/** Caption  for labeling causative exception stack traces */
private static final String CAUSE_CAPTION = "Caused by: ";

/** Caption for labeling suppressed exception stack traces */
private static final String SUPPRESSED_CAPTION = "Suppressed: ";
```

### 构造函数
`Throwable`有四种构造函数，
#### public Throwable(){..}
无参构造函数，定义如下：
```Java
public Throwable() {
    fillInStackTrace();
}
```
发现直接调用了成员方法`fillInStackTrace`，其定义如下：
```java
public synchronized Throwable fillInStackTrace() {
    if (stackTrace != null ||
        backtrace != null /* Out of protocol state */ ) {
        fillInStackTrace(0);
        stackTrace = UNASSIGNED_STACK;
    }
    return this;
}
```
方法用`synchronized`修饰，所以是线程同步的。方法中首先判断stackTrace和backtrace其中有没有不是null的，任何一个方法非null表明什么呢？说明栈中已经有内容，然后调用具有参数的`fillInStackTrace`方法，定义如下：
```java
private native Throwable fillInStackTrace(int dummy);
```
这是一个native方法，dummy这个参数的意思目前还不了解。总之是调用了吧，然后给stackTrace赋值，最后返回的是对象本身，但是在调用者那里，并没有使用到这个返回值。

#### public Throwable(String message){...}

```java
public Throwable(String message) {
    fillInStackTrace();
    detailMessage = message;
}
```
多了一个给`detailMessage`赋值的过程。

#### public Throwable(String message, Throwable cause){...}
```java
public Throwable(String message, Throwable cause) {
    fillInStackTrace();
    detailMessage = message;
    this.cause = cause;
}
```

#### public Throwable(Throwable cause) {...
定义如下自己看：
```java
public Throwable(Throwable cause) {
    fillInStackTrace();
    detailMessage = (cause==null ? null : cause.toString());
    this.cause = cause;
}
```
其中`toString`的定义如下：
```java
public String toString() {
    String s = getClass().getName();
    String message = getLocalizedMessage();
    return (message != null) ? (s + ": " + message) : s;
}
```
其中`getLocalizedMessage`方法定义如下：
```java
public String getLocalizedMessage() {
    return getMessage();
}
```
而`getMessage`实际上直接返回成员变量`detailMessage`：
```java
public String getMessage() {
    return detailMessage;
}
```

#### protected Throwable(String message, Throwable cause, boolean enableSuppression, boolean writableStackTrace) {...}
JDK 1.7 中引入的构造方法，
```java
protected Throwable(String message, Throwable cause,
                       boolean enableSuppression,
                       boolean writableStackTrace) {
     if (writableStackTrace) {
         fillInStackTrace();
     } else {
         stackTrace = null;
     }
     detailMessage = message;
     this.cause = cause;
     if (!enableSuppression)
         suppressedExceptions = null;
 }

```

### public synchronized Throwable getCause() {...}

该方法是一个同步方法，用来获取造成`Throwable`的cause，定义如下：
```java
public synchronized Throwable getCause() {
    return (cause==this ? null : cause);
}
```
方法会返回造成`Throwable`的cause或者null，后者是因为原因未知或者不存在。cause什么时候被赋值呢？这个变量是私有成员变量，在初始化的时候赋值为this，即指向自己本身。后面可以通过构造方法或者`initCause()`方法进行赋值。所以如果`cause`仍旧等于自己，说明变量这个引起异常的对象并没有被初始化，这个时候返回null。


### public synchronized Throwable initCause(Throwable cause){...}
这就是上面`getCause`方法中提到的`initCause`方法，定义如下：
```java
public synchronized Throwable initCause(Throwable cause) {
   if (this.cause != this)
       throw new IllegalStateException("Can't overwrite cause with " +
                                       Objects.toString(cause, "a null"), this);
   if (cause == this)
       throw new IllegalArgumentException("Self-causation not permitted", this);
   this.cause = cause;
   return this;
}
```
提到当一个`Throwable`对象初始化的时候，cause是被赋值为this的，并且后面只能再被赋值一次，这个方法就解释为什么只能赋值一次：`this.cause != this`就会抛出异常，此时说明cause已经被赋值过了，必能再赋值进行覆盖。并且这次赋值不能再把自身作为参数。所以，这个方法最多只能被调用一次，通常在构造函数中被调用，如果调用的是`Throwable(Throwable)`或者`Throwable(String,Throwable)`这种构造方法，由于在构造方法中已经对cause进行赋值，所以这个方法不会再被调用。这个方法有到底怎么用呢？可以看下面这个例子：
首先定义三个异常类
```java

public class ExceptionA extends Exception {
    public ExceptionA(String str) {
        super();
    }
}

public class ExceptionB extends ExceptionA {

    public ExceptionB(String str) {
        super(str);
    }
}

public class ExceptionC extends ExceptionA {
    public ExceptionC(String str) {
        super(str);
    }
}
```
执行下面这个程序：
```java
public class NeverCaught {
    static void f() throws ExceptionB{
        throw new ExceptionB("exception b");
    }

    static void g() throws ExceptionC {
        try {
            f();
        } catch (ExceptionB e) {
            ExceptionC c = new ExceptionC("exception a");
            throw c;
        }
    }

    public static void main(String[] args) {
            try {
                g();
            } catch (ExceptionC e) {
                e.printStackTrace();
            }
    }

}
```
结果是：
```java
/*
exception.ExceptionC
at exception.NeverCaught.g(NeverCaught.java:12)
at exception.NeverCaught.main(NeverCaught.java:19)
*/
```
发现只打印了`ExceptionC`而没有打印出`ExceptionB`呢？原因是在实例化`ExceptionC`的时候调用的方法是`Exceotion(String)`方法。这种情况叫做“异常链”丢失，少了一个异常，在我们的排错过程中非常不利。这时候就用到了`initCause`方法：保存异常信息，在抛出另一个异常的同时不丢失原来的异常：
```java

public class NeverCaught {
    static void f() throws ExceptionB{
        throw new ExceptionB("exception b");
    }

    static void g() throws ExceptionC {
        try {
            f();
        } catch (ExceptionB e) {
            ExceptionC c = new ExceptionC("exception a");
            //异常链
            c.initCause(e);
            throw c;
        }
    }

    public static void main(String[] args) {
            try {
                g();
            } catch (ExceptionC e) {
                e.printStackTrace();
            }
    }

}
```
其中`c.initCause`保持了异常链的完整性，当然其实可以采取`ExceptionC c = new ExceptionC("exception a", e)`这种构造方法来使异常链完整的。注意 `Throwable`中cause只能被赋值一次。
### printStackTrace方法
定义了三种printStackTrace方法，第一种无参数：
```java
public void printStackTrace() {
    printStackTrace(System.err);
}
```
其中调用了以`PrintStream`对象作为参数的`printStackTrace`方法，定义如下：
```java
public void printStackTrace(PrintStream s) {
    printStackTrace(new WrappedPrintStream(s));
}
```
其中调用了以`WrappedPrintStream`对象作为参数的`printStackTrace`方法，定义如下：
```java
private void printStackTrace(PrintStreamOrWriter s) {
    // Guard against malicious overrides of Throwable.equals by
    // using a Set with identity equality semantics.
    Set<Throwable> dejaVu =
        Collections.newSetFromMap(new IdentityHashMap<Throwable, Boolean>());
    dejaVu.add(this);

    synchronized (s.lock()) {
        // Print our stack trace
        s.println(this);
        StackTraceElement[] trace = getOurStackTrace();
        for (StackTraceElement traceElement : trace)
            s.println("\tat " + traceElement);

        // Print suppressed exceptions, if any
        for (Throwable se : getSuppressed())
            se.printEnclosedStackTrace(s, trace, SUPPRESSED_CAPTION, "\t", dejaVu);

        // Print cause, if any
        Throwable ourCause = getCause();
        if (ourCause != null)
            ourCause.printEnclosedStackTrace(s, trace, CAUSE_CAPTION, "", dejaVu);
    }
}
```
输出到`System.err`，首先第一行调用`toString`方法，输出异常的类名和message(如果有的话)。接着调用`getOurStackTrace`，这个方法是做什么的？
```java

private synchronized StackTraceElement[] getOurStackTrace() {
    // Initialize stack trace field with information from
    // backtrace if this is the first call to this method
    if (stackTrace == UNASSIGNED_STACK ||
        (stackTrace == null && backtrace != null) /* Out of protocol state */) {
        int depth = getStackTraceDepth();
        stackTrace = new StackTraceElement[depth];
        for (int i=0; i < depth; i++)
            stackTrace[i] = getStackTraceElement(i);
    } else if (stackTrace == null) {
        return UNASSIGNED_STACK;
    }
    return stackTrace;
}
```
其中`getStackTraceDepth`方法是一个native方法，可以看到实际上它是拷贝了一份`StackTraceElement`的内容，之后循环打印出内容然后调用了`getSuppressed`方法，定义如下：
```java
public final synchronized Throwable[] getSuppressed() {
    if (suppressedExceptions == SUPPRESSED_SENTINEL ||
        suppressedExceptions == null)
        return EMPTY_THROWABLE_ARRAY;
    else
        return suppressedExceptions.toArray(EMPTY_THROWABLE_ARRAY);
}
```
这里将会返回一个包含所有被抑制的异常数组，如果没有被一直的异常或者这个异常本身由`Throwable(String, Throwable, boolean, boolean)`方法创建，倒数第二个参数是false，也就是被抑制的异常不显示出来，此时返回的是一个空数组。

这之后的`printEnclosedStackTrace`又是怎么回事，
```java
private void printEnclosedStackTrace(PrintStreamOrWriter s,
                                        StackTraceElement[] enclosingTrace,
                                        String caption,
                                        String prefix,
                                        Set<Throwable> dejaVu) {
   assert Thread.holdsLock(s.lock());
   if (dejaVu.contains(this)) {
       s.println("\t[CIRCULAR REFERENCE:" + this + "]");
   } else {
       dejaVu.add(this);
       // Compute number of frames in common between this and enclosing trace
       StackTraceElement[] trace = getOurStackTrace();
       int m = trace.length - 1;
       int n = enclosingTrace.length - 1;
       while (m >= 0 && n >=0 && trace[m].equals(enclosingTrace[n])) {
           m--; n--;
       }
       int framesInCommon = trace.length - 1 - m;

       // Print our stack trace
       s.println(prefix + caption + this);
       for (int i = 0; i <= m; i++)
           s.println(prefix + "\tat " + trace[i]);
       if (framesInCommon != 0)
           s.println(prefix + "\t... " + framesInCommon + " more");

       // Print suppressed exceptions, if any
       for (Throwable se : getSuppressed())
           se.printEnclosedStackTrace(s, trace, SUPPRESSED_CAPTION,
                                      prefix +"\t", dejaVu);

       // Print cause, if any
       Throwable ourCause = getCause();
       if (ourCause != null)
           ourCause.printEnclosedStackTrace(s, trace, CAUSE_CAPTION, prefix, dejaVu);
   }
}
```
过程好复杂看不懂，反正就是打印了呗。

最后打印cause，同样是一个复杂的过程，不看了。

可以看到除了第一行，后面的打印形式是依据具体的实现的，下面有几种典型的情况：
```Java
class MyClass {
    public static void main(String[] args) {
       crunch(null);
    }
   static void crunch(int[] a) {
      mash(a);
   }
   static void mash(int[] b) {
      System.out.println(b[0]);
   }
}
```
执行程序打印出以后的结果：
```Java
java.lang.NullPointerException
      at MyClass.mash(MyClass.java:9)
      at MyClass.crunch(MyClass.java:6)
      at MyClass.main(MyClass.java:3)
```
可以看到这里抛出了空指针异常，后面打印了`StackTraceElement`的内容。

再看下面这个例子：
```java
public class Junk {
    public static void main(String args[]) {
          try {
                    a();
              } catch(HighLevelException e) {
                  e.printStackTrace();
              }
          }
         static void a() throws HighLevelException {
            try {
                b();
            } catch(MidLevelException e) {
                 throw new HighLevelException(e);
             }
         }
         static void b() throws MidLevelException {
             c();
         }
         static void c() throws MidLevelException {
             try {
                 d();
             } catch(LowLevelException e) {
                throw new MidLevelException(e);
             }
         }
         static void d() throws LowLevelException {
            e();
         }
         static void e() throws LowLevelException {
             throw new LowLevelException();
         }
     }

     class HighLevelException extends Exception {
         HighLevelException(Throwable cause) { super(cause); }
     }

     class MidLevelException extends Exception {
         MidLevelException(Throwable cause)  { super(cause); }
     }

     class LowLevelException extends Exception {
     }
```
执行main方法将出现以下结果：
```Java

HighLevelException: MidLevelException: LowLevelException
      at Junk.a(Junk.java:13)
      at Junk.main(Junk.java:4)
Caused by: MidLevelException: LowLevelException
      at Junk.c(Junk.java:23)
      at Junk.b(Junk.java:17)
      at Junk.a(Junk.java:11)
  ... 1 more
Caused by: LowLevelException
      at Junk.e(Junk.java:30)
      at Junk.d(Junk.java:27)
      at Junk.c(Junk.java:21)
  ... 3 more
```
可以看到，main方法最终会执行到`e()`该方法会抛出`LowLevelException`异常，这个类调用的是父类无参构造方法，也就是其创建的时候没有给cause赋值。而向上抛出异常的过程中，都调用的是以cause为参数的构造方法，将上一个以上当做cause,所以可以看到`MidLevelException`和`HighLevelException`的`toString`方法不单单打印了类名，还有`getMessage`的值。

> 注意上面代码中包含`...`的行，这些表示堆栈跟踪的其余部分。异常匹配是从底部指定的帧数

这个例子打印的是cause数组的值，那么“被抑制的异常”是什么意思呢？`suppressedExceptions`是Java 7的新特性，在`try-with-resources`语法中调用`close()`方法的时候，如果此时出现异常，那么值之前出现的异常被叫做`Suppressed Exceptions`。`Throwable`类中有一个`addSuppressed`方法可以将它保存起来，定义如下：
```java
public final synchronized void addSuppressed(Throwable exception) {
      if (exception == this)
          throw new IllegalArgumentException(SELF_SUPPRESSION_MESSAGE, exception);

      if (exception == null)
          throw new NullPointerException(NULL_CAUSE_MESSAGE);

      if (suppressedExceptions == null) // Suppressed exceptions not recorded
          return;

      if (suppressedExceptions == SUPPRESSED_SENTINEL)
          suppressedExceptions = new ArrayList<>(1);

      suppressedExceptions.add(exception);
}
```
当用户捕捉到`close`里抛出的异常时，就可以调用`Throwable.getSuppressed`函数来取出close之前的异常了，这个方法定义如下：
```java
public final synchronized Throwable[] getSuppressed() {
    if (suppressedExceptions == SUPPRESSED_SENTINEL ||
        suppressedExceptions == null)
        return EMPTY_THROWABLE_ARRAY;
    else
        return suppressedExceptions.toArray(EMPTY_THROWABLE_ARRAY);
}
```

一个典型的例子如下：
```java

Exception in thread "main" java.lang.Exception: Something happened
       at Foo.bar(Foo.java:10)
       at Foo.main(Foo.java:5)
       Suppressed: Resource$CloseFailException: Resource ID = 0
               at Resource.close(Resource.java:26)
               at Foo.bar(Foo.java:9)
               ... 1 more
```
如果一个异常兼有一个`cause`和多个`suppressedExceptions`，那么它的打印结果可能是这样的：
```java
Exception in thread "main" java.lang.Exception: Main block
       at Foo3.main(Foo3.java:7)
       Suppressed: Resource$CloseFailException: Resource ID = 2
               at Resource.close(Resource.java:26)
               at Foo3.main(Foo3.java:5)
       Suppressed: Resource$CloseFailException: Resource ID = 1
               at Resource.close(Resource.java:26)
               at Foo3.main(Foo3.java:5)
      Caused by: java.lang.Exception: I did it
       at Foo3.main(Foo3.java:8)
```
同样的,`suppressedExceptions`也可能含有cause，
```java
Exception in thread "main" java.lang.Exception: Main block
       at Foo4.main(Foo4.java:6)
       Suppressed: Resource2$CloseFailException: Resource ID = 1
               at Resource2.close(Resource2.java:20)
               at Foo4.main(Foo4.java:5)
       Caused by: java.lang.Exception: Rats, you caught me
               at Resource2$CloseFailException.&lt;init&gt;(Resource2.java:45)
               ... 2 more
```

### private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException{...}
```java
private void readObject(ObjectInputStream s)
        throws IOException, ClassNotFoundException {
    s.defaultReadObject();     // read in all fields
    if (suppressedExceptions != null) {
        List<Throwable> suppressed = null;
        if (suppressedExceptions.isEmpty()) {
            // Use the sentinel for a zero-length list
            suppressed = SUPPRESSED_SENTINEL;
        } else { // Copy Throwables to new list
            suppressed = new ArrayList<>(1);
            for (Throwable t : suppressedExceptions) {
                // Enforce constraints on suppressed exceptions in
                // case of corrupt or malicious stream.
                if (t == null)
                    throw new NullPointerException(NULL_CAUSE_MESSAGE);
                if (t == this)
                    throw new IllegalArgumentException(SELF_SUPPRESSION_MESSAGE);
                suppressed.add(t);
            }
        }
        suppressedExceptions = suppressed;
    } // else a null suppressedExceptions field remains null

    /*
     * For zero-length stack traces, use a clone of
     * UNASSIGNED_STACK rather than UNASSIGNED_STACK itself to
     * allow identity comparison against UNASSIGNED_STACK in
     * getOurStackTrace.  The identity of UNASSIGNED_STACK in
     * stackTrace indicates to the getOurStackTrace method that
     * the stackTrace needs to be constructed from the information
     * in backtrace.
     */
    if (stackTrace != null) {
        if (stackTrace.length == 0) {
            stackTrace = UNASSIGNED_STACK.clone();
        }  else if (stackTrace.length == 1 &&
                    // Check for the marker of an immutable stack trace
                    SentinelHolder.STACK_TRACE_ELEMENT_SENTINEL.equals(stackTrace[0])) {
            stackTrace = null;
        } else { // Verify stack trace elements are non-null.
            for(StackTraceElement ste : stackTrace) {
                if (ste == null)
                    throw new NullPointerException("null StackTraceElement in serial stream. ");
            }
        }
    } else {
        // A null stackTrace field in the serial form can result
        // from an exception serialized without that field in
        // older JDK releases; treat such exceptions as having
        // empty stack traces.
        stackTrace = UNASSIGNED_STACK.clone();
    }
}
```

### private synchronized void writeObject(ObjectOutputStream s) throws IOException{...}
```java
private synchronized void writeObject(ObjectOutputStream s)
       throws IOException {
   // Ensure that the stackTrace field is initialized to a
   // non-null value, if appropriate.  As of JDK 7, a null stack
   // trace field is a valid value indicating the stack trace
   // should not be set.
   getOurStackTrace();

   StackTraceElement[] oldStackTrace = stackTrace;
   try {
       if (stackTrace == null)
           stackTrace = SentinelHolder.STACK_TRACE_SENTINEL;
       s.defaultWriteObject();
   } finally {
       stackTrace = oldStackTrace;
   }
}
```





接下来所有行输出了由`fillInStackTrace`方法执行的结果。


参考：
* [Java异常处理机制](http://blog.sina.com.cn/s/blog_6f6c0f3501019mdo.html)
* [Java7里try-with-resources分析](http://blog.csdn.net/hengyunabc/article/details/18459463)
