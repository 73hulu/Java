# Object
从JDK1.0起，Object是所有类的父类，每个类都使用它作为超类。注意，所有类，包括arrays。

Object类的结果如下：

![Object_classStructure](http://ovn0i3kdg.bkt.clouddn.com/Object_classStructure.png)


可以看到，Object类没有任何的属性，只有12方法，而且这些方法都被其子类继承覆盖，非常重要。

> 严格来说，Object有13个方法，没有在structure中显示出来的是默认构造方法`public Object()`。这是因为，Java中规定：在类定义过程中，对于未定义构造函数的类，默认会生成一个无参数的构造函数。Object作为所有类的基类，自然要反映出此特性，在源码中虽然没有给出定义，但是这个方法实际上是存在的。

> 当然，并不是所有的类都是以这种方法去构建的，也并不是所有的类构造函数都是public的。


### private static native void registerNatives()；

  `registerNatives`含义是本地注册的意思，方法被`static`和`navtive`两个关键词修饰。被`static`修饰的方法是静态方法。`native`关键词修饰的函数表明该方法的实现并不是在Java中完成，而是由C/C++去完成，并被编译了`.dll`，由Java调用。方法的具体实现体在`.dll`文件中，对于不同的平台，其具体的使用应该有所不同。用`native`修饰，即表示操作系统，需要提此方法，Java本身需使用。具体到`registerNatives()`方法本身，其主要作用是将C/C++中的方法映射到Java中的native方法，实现方法命名的解耦。

  > 通常情况下，为了使JVM发现您的本机功能，他们被一定的方式命名。对于java.lang.Object,registerNatives，对应的C函数命名为Java_java_lang_Object_registerNatives，通过使用registerNatives（或者更确切地说，JNI函数RegisterNatives），您可以命名任何你想要你的C函数。相关的C代码可以参考http://www.linuxidc.com/Linux/2015-06/118676.htm

  注意到，`registerNatives`的修饰符为private，而且无方法体，那么怎么达到作用呢？在源码中，该方法后面跟了紧接着一段静态代码块：
  ```java
  private static native void registerNatives();
  static {
     registerNatives();
  }

  ```
  根据Java初始化的加载顺序：静态代码块在类**被加载**的时候执行且仅执行一次，一般用来初始化静态变量和调用静态方法。在这里，Object在被构造之前先执行了静态代码块，其中调用的registerNatives方法。

### public final native Class<?> getClass()；

  `getClass`也是一个由 `native`修饰的方法，由`final`修饰表示不能被继承。返回的是此Object对象的类对象/运行时类对象Class<?>，效果与Object.class相同。

  上面这段话中，注意到一个词：“类对象”，那么什么是”类对象”呢？在Java中，**类**是对具有一组相同特征或行为的实例的抽象并进行描述，**对象**是此类所描述的特征或行为的具体实例。而作为概念层次的“类”，其本省也具有某些共同的特性，比如都具有类名称，由类加载器去加载，都具有包，具有父类、属性和方法等。所以，Java中有专门定义的一个类——**Class**，去描述其他类所具有的这些特征。因此，从这个角度看，类本身也是属于Class类的对象。为了与经常意义上的对象相区别，在此称为"类对象"。

  这个方法常常与“反射”联系到一起，至于反射的相关知识点会有一个专题去学习，再次不做记录。

### public boolean equals(Object obj){...}

   用来判断两个对象是否相等，需要注意的是，这个方法的参数是`Object`类的对象，在其子类重写该方法时候，别忘记了这个点！有面试的时候让手写过。

   说到`equals`，不得不提的一个运算符就是`==`。`==`表示的是变量值完全相同（对于基础类型，地址中存储的是值，引用类型则存储指向实际对象的地址），`equals`表示的是对象的内容完全相同，此处多指对象的特征/属性。

   在Object中，`equals()`方法定义如下：
   ```java
   public boolean equals(Object obj) {
        return (this == obj);
    }
  ```
  可见，在Object中，判断是否相等的“标尺”仅仅是"=="，但是在更复杂类中，判断的标尺没有这么简单了，需要按照实际的需求对标尺的含义重新定义。如在String类中，可以将字符串的类型作为标尺。如果子类没有重新定义`equals()`方法，那么它会继承其父类的的`equals()`方法，直到`Object`。

  提到`equals()`方法，有一句话需要谨记：**重新equals()方法必须重写hashCode()方法！！！**。

### public native int hashCode();
`hashCode()`返回一个整型数值，表示该对象的哈希码值。源码中对该方法做了如下约定：
 1. 在Java应用程序执行期间，对于同一个对象多次调用`hashCode()`方法，其返回值是相同的，前提是将对象进行`equals`比较时候所用的标尺信息未做修改。在Java应用程序的一次执行到另一次执行，同一个对象的`hashCode()`返回的哈希码无须保持一致。
 2. 如果两个对象相等（依据：调用equals()方法），那么这两个对象调用`hashCode()`返回的哈希码也必须相等。
 3. 反之，如果两个对象调用的`hashCode()`不相等，那么他们一定不相等。

 由此可见， `hasCode()`、`equals()`方法和"两个对象相等"常常联系到一起，他们之间严格的数学逻辑是：
  ````
  两个对象相等 <=>  equals()相等  => hashCode()相等。

  ````
因此，**重写equlas()方法必须重写hashCode()方法**，以保证此逻辑严格成立，同时可以推理出：
````
hasCode()不相等 => equals（）不相等 <=> 两个对象不相等。
````

可以看出，“两个对象是否相等”等价于"equals()"是否相等，那么`hasCode()`是否还有存在的必要呢？

`hashCode()`的存在主要是增强哈希表的性能。举个例子，以集合类中，以Set为例，当新加一个对象时，需要判断现有集合中是否已经存在与此对象相等的对象，如果没有hashCode()方法，需要将Set进行一次遍历，并逐一用equals()方法判断两个对象是否相等，此种算法时间复杂度为o(n)。通过借助于hasCode方法，先计算出即将新加入对象的哈希码，然后<u>根据哈希算法计算出此对象的位置</u>，直接判断此位置上是否已有对象即可。（注：Set的底层用的是Map的原理实现）。

这里有一个常见错误：
**对象的hashCode()返回的不是对象所在的物理内存地址。甚至也不一定是对象的逻辑地址，hashCode()相同的两个对象，不一定相等，换言之，不相等的两个对象，hashCode()返回的哈希码可能相同。**

上面说到，**重写equals方法时一定要重写hashCode方法**，下面是一个例子：
```java
public class User {
    private int uuid;
    private String name;
    private int age;

    public User() {
    }

    public User(int uuid, String name, int age) {
        this.uuid = uuid;
        this.name = name;
        this.age = age;
    }

    public int getUuid() {
        return uuid;
    }

    public void setUuid(int uuid) {
        this.uuid = uuid;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public boolean equals(Object obj){
        if (obj == null || !(obj instanceof User)){
            return false;
        }
        if (((User) obj). getUuid() == this.getUuid()){
            return true;
        }
        return false;
    }

    @Override
    public int hashCode(){
        int result = 17;
         result = 31 * result + this.getUuid();
         return result;
    }

    public static void main(String[] args) {
        User user = new User(1, "baoxiaofang", 18);
        User user1 = user;

        System.out.println(user.hashCode());
        System.out.println(user1.hashCode());
        System.out.println(user.equals(user1));
    }
}
```
注：上述hashCode()的重写中出现了`result * 31`，是因为`result * 31 = (result<<5) - result`。之所以选择31，是因为**左移运算和减运算计算效率远大于乘法运算**。当然，也可以选择其他数字。

### protected native Object clone() throws CloneNotSupportedException;

`clone()`方法返回一个引用，指向的是新的clone出来的对象，此对象和原来对象占据了不同的堆空间。既然这样，我就可以写出下面这段程序了:
```java
public class ObjectCloneTest {
    public static void main(String[] args) {
        Object object = new Object();
        Object object1 = object.clone();
    }
}
```
实际上，如果在IDE中写这段程序，发现写最后一条语句时，自动提示中并没有clone()方法，当然这个程序编译通过，会提示你“Error:(14, 33) java: clone()可以在java.lang.Object中访问protected”。好，问题知道了，`Object`中`clone()`方法用`protected`修饰，即同一个包内或者不同包内的子类可以访问，`ObjectCloneTest`和`Object`不在一个包内，但是前者继承了后者，为什么不能访问？

关键在于对"不同包中的子类可以访问"没有正确理解！

> "不同包中的子类可以访问"，是指当两个类不在同一个包中的时候，继承自父类的子类内部且主调（调用者）为子类的引用时才能访问父类用protected修饰的成员（属性/方法）。 在子类内部，主调为父类的引用时并不能访问此protected修饰的成员。！（super关键字除外）

改成下面这样就可以正常编译了：
```java
public static void main(String[] args) {

       ObjectCloneTest objectCloneTest = new ObjectCloneTest();

       try {
           ObjectCloneTest objectCloneTest1 = (ObjectCloneTest)objectCloneTest.clone();
       }catch (Exception e){
           e.printStackTrace();
       }
   }

```
因为此时clone方法主调已经是子类的引用了。这里又有一个知识点了，`clone()`方法返回值是Object类型，需要强制转化。

但是运行该main方法抛出了异常，如下:
```java
java.lang.CloneNotSupportedException: JavaLangJarTest.ObjectCloneTest
	at java.lang.Object.clone(Native Method)
	at JavaLangJarTest.ObjectCloneTest.main(ObjectCloneTest.java:17)
```
问题在于，Java语法规定：
**clone()正确调用需要实现Cloneable接口**，如果没有实现，并且子类直接调用**Object类**的clone()方法，就会抛出`CloneNotSupportedException`异常。
所以代码改成下面这种就可以完美执行啦：
```java
public class ObjectCloneTest  implements Cloneable{
    public static void main(String[] args) {

        ObjectCloneTest objectCloneTest = new ObjectCloneTest();

        try {
            ObjectCloneTest objectCloneTest1 = (ObjectCloneTest)objectCloneTest.clone();
            System.out.println("o1: = " +  objectCloneTest1);
        }catch (Exception e){
            e.printStackTrace();
        }
    }
}
```

### public String toString(){...}
Object中的`toString()`方法定义如下：
```java
public String toString() {
    return getClass().getName() + "@" + Integer.toHexString(hashCode());
}
```
返回了一个字符串，构成是“对象所属类名@哈希值的十六进制”。 `toString()`经常被用到，但是很多时候不是显式调用，比如`System.out.println(obj)`时，其内部也是通过toString()来实现的。

### protected void finalize() throws Throwable{}

`finalize()`方法主要和垃圾回收机制有段，在Object类中该方法的定义如下：
```java
protected void finalize() throws Throwable { }
```
方法体为空，为什么为空？

finalize具体的调用时机在：JVM准备对此对象所占用的内容空间进行垃圾回收之前，该方法被调用。所以一般来说，这个方法不是我们主动去调用的。当然你可以主动调用，这个时候，重新的这个方法就与其他自定义犯法没有什么区别，如下面这个例子：
```java
public class ObjectFinalizeTest {
    @Override
    public void finalize(){
        System.out.println("This is from finalize method");
    }
    public static void main(String[] args) {
        ObjectFinalizeTest obj = new ObjectFinalizeTest();
        obj = null;

        System.gc();

    }
}
```
上面这个例子重写了`finalize()`方法，之后在main方法中，先得到一个实例，在将null赋值给它，目的是为了让他在下一轮的垃圾回收过程中被回收，`System.gc()`主动触发垃圾回收机制【注意，触发之后并不是立即执行垃圾回收，什么时候回收是由JVM决定的】。程序运行之后将会打印出"This is from finalize method"。

###  wait | notify | notifyAll    
这三种方法用于Java多线程之间的协作。Object中一共重载了三种wait方法，一种notify方法，一种notifyAll方法。各自的定义和含义如下：

* public final void wait() throws InterruptedException

 调用此方法的当前线程等待，直到在其他线程上调用此方法的主调（某一对象）的notify()/notifyAll()方法。
  ```java
  public final void wait() throws InterruptedException {
      wait(0);
  }
  ```
  方法体中调用了有一个参数的wait方法。
* wait(long timeout)/wait(long timeout, int nanos)

  调用此方法所在的当前线程等待，直到其他线程上调用此方法的主调（某一对象）的notify()/notifyAll方法，或者超过指定的超时时间量。
  ```java
  public final native void wait(long timeout) throws InterruptedException;
  ```

  ```java
  public final void wait(long timeout, int nanos) throws InterruptedException {
      if (timeout < 0) {
          throw new IllegalArgumentException("timeout value is negative");
      }

      if (nanos < 0 || nanos > 999999) {
          throw new IllegalArgumentException(
                              "nanosecond timeout value out of range");
      }

      if (nanos > 0) {
          timeout++;
      }

      wait(timeout);
  }
  ```
  注意两个参数的这个`wait`方法，第一个参数timeout的单位是毫秒，参数nanos的单位是纳秒， 1毫秒 = 1000 微秒 = 1000 000 纳秒， 处理时候，由于纳秒时间太短（猜测），所以对参nanos采取了近似处理。这个方法在JDK1.6中的定义与上面的定义不同，1.6中的定义方法如下：
  ```java
    public final void wait(long timeout, int nanos) throws nterruptedException {
        if (timeout < 0) {
            throw new IllegalArgumentException("timeout value is negative");
        }
    //nanos 单位为纳秒,  1毫秒 = 1000 微秒 = 1000 000 纳秒
        if (nanos < 0 || nanos > 999999) {
            throw new IllegalArgumentException(
                                "nanosecond timeout value out of range");
        }

       //  nanos 大于 500000 即半毫秒  就timout 加1毫秒
       //  特殊情况下: 如果timeout为0且nanos大于0,则timout加1毫秒
        if (nanos >= 500000 || (nanos != 0 && timeout == 0)) {
            timeout++;
        }

        wait(timeout);
    }
    ```
这对于半毫秒的加1毫秒，小于1毫秒的舍弃（特殊情况下，参数timeout为0时，参数nanos大于0时，也算1毫秒），器租用应该在能更精确控制等待时间（尤其在高并发时，毫秒的时间节省也是值得的）。


*  public final native void notify();

  作用是唤醒在此对象监视器上等待的单个线程。

* public final native void notifyAll();

  作用是唤醒在此对象监视器上等待的所有线程。

  以上三种方法都是配套使用的，下面是一个简单的例子：
  ```java
    public class ThreadTest {

        /**
         * @param args
         */
        public static void main(String[] args) {
            // TODO Auto-generated method stub
            MyRunnable r = new MyRunnable();
            Thread t = new Thread(r);
            t.start();
            synchronized (r) {
                try {
                    System.out.println("main thread 等待t线程执行完");
                    r.wait();
                    System.out.println("被notity唤醒，得以继续执行");
                } catch (InterruptedException e) {
                    // TODO Auto-generated catch block
                    e.printStackTrace();
                    System.out.println("main thread 本想等待，但被意外打断了");
                }
                System.out.println("线程t执行相加结果" + r.getTotal());
            }
        }
    }

    class MyRunnable implements Runnable {
        private int total;

        @Override
        public void run() {
            // TODO Auto-generated method stub
            synchronized (this) {
                System.out.println("Thread name is:" + Thread.currentThread().getName());
                for (int i = 0; i < 10; i++) {
                    total += i;
                }
                notify();
                System.out.println("执行notif后同步代码块中依然可以继续执行直至完毕");
            }
            System.out.println("执行notif后且同步代码块外的代码执行时机取决于线程调度");
        }

        public int getTotal() {
            return total;
        }
    }

  ```
  执行结果为：
  ```java
  main thread 等待t线程执行完
  Thread name is:Thread-0
  执行notif后同步代码块中依然可以继续执行直至完毕
  执行notif后且同步代码块外的代码执行时机取决于线程调度  //此行输出位置有具体的JVM线程调度决定，有可能最后执行
  被notity唤醒，得以继续执行
  线程t执行相加结果45

  ```

  既然是作用于多线程中，为什么却是Object这个基类所具有的方法？原因在于理论上任何对象都可以视为线程同步中的监听器，且wait(...)/notify()|notifyAll()方法只能在同步代码块中才能使用。
  从上述例子的输出结果中可以得出如下结论：
  1. `wait(...)`方法调用后当前线程将立即阻塞，且适当其所持有的同步代码块中的锁，直到被唤醒或超时或打断后且重新获取到锁后才能继续执行；
  2. `notify()/notifyAll()`方法调用后，其所在线程不会立即释放所持有的锁，直到其所在同步代码块中的代码执行完毕，此时释放锁，因此，如果其同步代码块后还有代码，其执行则依赖于JVM的线程调度。


  多线程是一块很难啃的骨头，具体的还是在以后的专题中整理吧。


  ##### 参考：
  * [Java总结篇系列：java.lang.Object](http://www.cnblogs.com/lwbqqyumidi/p/3693015.html)
