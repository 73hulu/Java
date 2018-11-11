# Singleton

## 什么是单例？
单例模式是最常用的设计模式，也是最简单的设计模式。它保证一个类只要一个实例，并且提供一个全局的访问点。


## 单例模式的实现
首先，如何保证只有一个？不能被随意创建吧，所以至少构造方法是私有的。OK，私有是可以接受的，但是你得提供访问方法啊？这里需要解决两个问题。一是对象只有一个，只能被创建一遍，如何保存这个实例呢，很简单，将其作为一个私有变量就行，注意是私有的。二是如何提供方法点呢，提供一个公有的方法，这个方法将这个私有变量返回出来就行。由于这个类不能被初始化（因为没有公有的构造函数），所以只能通过静态方法访问了，所以这个“全局访问点”应该是一个**public static**修饰的方法，根据“静态方法不能访问实例变量”这个特性，保存实例的那个私有变量应该也被"static"修饰。

> 为什么不能是公有的instance?比如写成这样：
> ```java
> public Singleton{
  public static final Singleton = new Singleton()
  private Singleton(){}
}
> ```

所以，总结下来，Singleton实现有三个要点：
1. 需要有一个**私有静态变量**保存这个实例，使得这个类的所有对象都公用这个实例。
2. 类中还得有一个私有的构造函数，来主动创建这个私有静态变量
3. 类得提供一个**公有静态**方法来让别人获得这个实例。

![Singleton](https://ws1.sinaimg.cn/large/006tNbRwly1fx41enczb6j30mq09474p.jpg)


单例模式有很多实现方式。下面就是几种常见的实现：
### 懒汉模式
为什么起这个名儿呢？因为这个方法很懒，懒到什么程度？只有在被调用的时候才会实例化这唯一的一个实例，如果一直不被调用，那么一直不实例化。实现过程如下：
```java
public class LazySingleton {
    private static LazySingleton instance;

    private LazySingleton(){
    }
    public static LazySingleton getInstance(){
        if (instance == null){
            instance = new LazySingleton();
        }
        return instance;
    }
}
```
这种写法是线程非安全的。因为如果有两个线程调用`getInstance`方法，它们同时执行到`instance == null`，此时都会判断为true，所以都会分别创建一个实例，违背了单例模式的初衷。

### 饿汉模式
所谓的饿汉，就是等不及别人来调用，自己就先实例化自己，生怕自己饿着了。所以叫饿汉。实现如下：
```java
public class HugerSingleton {
    private static final HugerSingleton instance = new HugerSingleton();

    private HugerSingleton(){
    }
    public static HugerSingleton getInstance(){
        return instance;
    }
}
```
这种写法一定是线程安全的，因为`instance`是类变量，在类载入的时候就初始化好了。所以不管多少个线程调用，得到的都是同一个实例。
这个方法的缺点在不必要获取实例的时，已经产生了开销。
### 双重锁模式
又想要不过早产生开销，又想要线程安全，所以对“懒汉模式”做出改变。实现如下：
```java
public class SyncSingleton {
    private static SyncSingleton instance;

    private SyncSingleton(){}

    public static SyncSingleton getInstance(){
        if (instance == null){  // 这个条件的存在是为了提交性能，不让这个方法每调用一次就同步一次
            synchronized (SyncSingleton.class){ // 这是为了保证线程安全
                if (instance === null) { // 这是为了保证单例
                  instance = new SyncSingleton();                  
                }
            }
        }
        return instance;
    }
}
```
双重锁写法是对懒汉模式的改进，采用类锁的方式。假如有两个线程同时调用`getInstance`方法，由于`synchronized`类锁的存在，只有一个线程能执行，另一个必须等待，等到第一个执行完了，第二个线程判断的时候已经没有`instance == null`的条件，所以两个线程返回的是同一个实例。为什么叫“双重锁”呢？哪双重？第一重是`synchronized`这个条件，第二个是`instance === null`这个条件，这两个锁保证了不过早产生开销，又保证了线程同步。

> 为什么DoubleCheck需要两次判断和一次同步呢？试想有两个线程A和B，同时请求单例，显然他们都可以通过第一层的`instance == null`的判断，然后相互竞争，假设A获得了类锁，然后进入了临界区进行初始化，最后退出，这时候开始新一轮的锁的竞争，假设B得到的锁，进入临界区，如果此时没有第二重锁`instance == null`，那么B线程仍旧还会调用`instance == new Singleton()`方法，也就没有达到单例的效果了。


> 也有人说可以将`synchronized`加到`getInstance`方法上，让它成为一个线程同步方法，这种做法同样可以实现功能，但是同步方法被频繁调用，内存占用大，效率低。而采用同步块的方法，由于第一重锁的存在，同步块实际上只被执行了一次，效率高。所以双重锁模式是单例的最佳实现。

### 线程安全的双重锁模式
关于指令重排的内容参见"JVM-内存模型"的相关内容。这里解释一下为什么下原来双重锁的单例可能存在线程安全问题：

`instance = new SyncSingleton()`不是一个原子操作，它可以分解成以下三个原子操作：
```java
// 伪代码
memory = allocate(); // 1. 分配对象内存空间
instance (memory); // 2. 初始化对象
instance = memory; // 3. 设置instance指向刚才分配的内存地址, 此时 instance != null
```

由于存在指令重排，所以重排之后的指令可能是这样的：
```java
// 伪代码
memory = allocate(); // 1. 分配对象内存空间
instance = memory; // 3. 设置instance指向刚才分配的内存地址, 此时 instance != null
instance (memory); // 2. 初始化对象
```
这就有问题了！
试想，一开始有ABC三个线程，一同去请求这个单例，他们首先到达第一层的判空，发现对象为空，此时开始锁竞争，最终胜出，而BC在同步块外面等着。此线程进入了同步块，判空发现还是null。那么进行`instance = new DoubleCheckLock()`，就是上述三个子过程，如果发生了指令重排，当进行到`instance = memory`之后，此时`instance != null`已经是true了。此时线程D也来请求这个单例，它停在第一个判空操作那里，发现`instance != null`为`true`，所以就直接返回这个单例，但是！这个A线程还没有来得及初始化呢！所以D线程用着这个单例一定会出错！这就是为什么需要volatile来禁止指令重排的原因了。

所以线程安全又高效的单例应该这么写：

```java
public class SyncSingleton {
    private volatile  SyncSingleton instance;

    private SyncSingleton(){}

    public static SyncSingleton getInstance(){
        if (instance == null){
            synchronized (SyncSingleton.class){
                instance = new SyncSingleton();
            }
        }
        return instance;
    }
}
```

### 占位方式
```java
public class LazyInitHolderSingleton {  
        private LazyInitHolderSingleton() {}  

        private static class SingletonHolder {  
                private static final LazyInitHolderSingleton INSTANCE = new LazyInitHolderSingleton();  
        }  

        public static LazyInitHolderSingleton getInstance() {  
                return SingletonHolder.INSTANCE;  
        }  
}  
```
这里使用了内部类来保存一个单例，只有第一次使用的时候才初始化内部类`SingletonHolder`类，所以既解决了性能也处理了并发的问题。

### 枚举型
```java
public enum SingletonClass{
    INSTANCE;
}
```

## 单例模式的使用场景
单例模式的使用场景当然是那些要求一个类有且仅有一个对象的场景，比如：
* 要求生成唯一序列号的环境
* 把整个项目中需要一个共享访问点或共享数据，例如一个web页面上的计数器，可以不用每次把刷新都记录到数据库中，使用单例模式保持计数器的值，并确保是线程安全的。
* 如果创建一个对象需要消耗的资源过多，比如要访问IO和数据库等资源，此时就可以考虑使用单例。
* 需要定义大量的静态常量和静态方法（比如工具类）的华宁，可以采用单例（当然也可以直接声明为static）

## 扩展——有上限的多例模式
单例是只允许出现一个实例的类，那么只允许出现n个实例的类应该怎么写？

很简单，在单例中保存n个实例，然后随机返回就行了。
```java
public clss Nton (){
  // 定义最多能产生的实例个数
  private static int maxNumber = 2;

 // 保存实例
 private static ArrayList<Nton> list = new ArrayList<Nton>();

 //当前实例的序号
 private static countNum = 0;

 private Nton(){}

 static {
   for (int i = 0; i <maxNumber; i ++){
     list.add(new Nton());
   }
 }

 public statci Nton getInstance (){
   Random random = new Random();
   // 随机找到一个实例
   int i = random.nextInt(maxNumber);
   return list.get(i);
 }
}
```

实际上可以看出，上面这种写法用的就是类似于单例中的“占位”方法。

## 实践
* JDK中的单例模式：java.lang.Runtime#getRuntime()
* Spring 中每个默认单例的bean，这些bean是可以被Spring容器管理的，如果采用非单例模式（prototype类型）,则bean初始化之后的管理就交由J2EE容器，Spring 容器不再跟踪管理Bean的生命周期。交由Spring容器管理的好处就是不用担心JVM的垃圾回收机制。



参考
* [[设计模式]单例模式](http://www.cnblogs.com/jingmoxukong/p/4015675.html)
* [ 设计模式：单例模式（Singleton）](https://blog.csdn.net/u013256816/article/details/50966882)
