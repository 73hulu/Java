# Singleton
单例模式是最常用的设计模式，也是最简单的设计模式。它保证一个类只要一个实例，并且提供一个全局的访问点。

首先，如何保证只有一个？不能被随意创建吧，所以至少构造方法是私有的。OK，私有是可以接受的，但是你得提供访问方法啊？这里需要解决两个问题。一是对象只有一个，只能被创建一遍，如何保存这个实例呢，很简单，将其作为一个私有变量就行，注意是私有的。二是如何提供方法点呢，提供一个公有的方法，这个方法将这个私有变量返回出来就行。由于这个类不能被初始化（因为没有公有的构造函数），所以只能通过静态方法访问了，所以这个“全局访问点”应该是一个**public static**修饰的方法，根据“静态方法不能访问实例变量”这个特性，保存实例的那个私有变量应该也被"static"修饰。

所以，总结下来，Singleton实现有三个要点：
1. 需要有一个**私有静态变量**保存这个实例，使得这个类的所有对象都公用这个实例。
2. 类中还得有一个私有的构造函数，来主动创建这个私有静态变量
3. 类得提供一个**公有静态**方法来让别人获得这个实例。

![Singleton](http://ovn0i3kdg.bkt.clouddn.com/Singleton.png)

单例模式有很多实现方式。下面就是几种常见的实现：
## 懒汉模式
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

## 饿汉模式
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
## 双重锁模式
又想要不过早产生开销，又想要线程安全，所以对“懒汉模式”做出改变。实现如下：
```java
public class SyncSingleton {
    private static SyncSingleton instance;

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
双重锁写法是对懒汉模式的改进，采用类锁的方式。假如有两个线程同时调用`getInstance`方法，由于`synchronized`类锁的存在，只有一个线程能执行，另一个必须等待，等到第一个执行完了，第二个线程判断的时候已经没有`instance == null`的条件，所以两个线程返回的是同一个实例。为什么叫“双重锁”呢？哪双重？第一重是`instance == null`这个条件，第二个是`synchronized`这个条件，这两个锁保证了不过早产生开销，又保证了线程同步。

> 也有人说可以将`synchronized`加到`getInstance`方法上，让它成为一个线程同步方法，这种做法同样可以实现功能，但是同步方法被频繁调用，内存占用大，效率低。而采用同步块的方法，由于第一重锁的存在，同步块实际上只被执行了一次，效率高。所以双重锁模式是单例的最佳实现。



参考
* [[设计模式]单例模式](http://www.cnblogs.com/jingmoxukong/p/4015675.html)
