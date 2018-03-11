# Java中的引用类型
有四种引用，强引用、软引用、弱引用和虚引用。

这么多引用类型的目的，第一是可以让程序员通过代码的方式决定某些对象的生命周期；第二是有利于JVM进行垃圾回收。
## 强引用
是指创建一个对象并把这个对象赋给一个引用变量。这是我们代码中普遍存在的，比如
```Java
Object object =new Object();
String str ="hello";
```
强引用有引用变量指向时永远不会被垃圾回收，JVM宁愿抛出OutOfMemory错误也不会回收这种对象。
但是要注意对象的生存周期，比如：
```Java
public static void main(String[] args) {  
      new Main().fun1();  
  }  

  public void fun1() {  
      Object object = new Object();  
      Object[] objArr = new Object[1000];  
}
```
这段代码中，当`fun1`结束的时候，`object`和`objArr`都会被GC。

想要中断强引用，方法是将强引用赋值为null。这样一来的话，JVM就可以回收该对象。

## 软引用(SoftReference)
软引用对象回收与否完全要看内存的脸色。如果一个对象具有软引用，内存空间足够，垃圾回收器就不会回收它；如果内存空间不足了，就会回收这些对象的内存。只要垃圾回收器没有回收它，该对象就可以被程序使用。

软引用可用来实现内存敏感的高速缓存,比如网页缓存、图片缓存等。使用软引用能防止内存泄露，增强程序的健壮性。   

`SoftReference`的特点是它的一个实例保存对一个Java对象的软引用， 该软引用的存在不妨碍垃圾收集线程对该Java对象的回收。

也就是说，一旦`SoftReference`保存了对一个Java对象的软引用后，在垃圾线程对这个Java对象回收前，`SoftReference`类所提供的`get()`方法返回Java对象的强引用。另外，一旦垃圾线程回收该Java对象之 后，`get()`方法将返回null。
```java
MyObject aRef = new  MyObject();  
SoftReference aSoftRef=new SoftReference(aRef);  
```

此时，对于这个`MyObject`对象，有两个引用路径，一个是来自`SoftReference`对象的软引用，一个来自变量`aReference`的强引用，所以这个`MyObject`对象是强可及对象。

随即，我们可以结束`aReference`对这个`MyObject`实例的强引用:
```Java
aRef = null;
```
这个`MyObject`对象成为了软引用对象。如果垃圾收集线程进行内存垃圾收集，并不会因为有一个`SoftReference`对该对象的引用而始终保留该对象。

Java虚拟机的垃圾收集线程对软可及对象和其他一般Java对象进行了区别对待:软可及对象的清理是由垃圾收集线程根据其特定算法按照内存需求决定的。
也就是说，垃圾收集线程会在虚拟机抛出`OutOfMemoryError`之前回收软可及对象，而且虚拟机会尽可能优先回收长时间闲置不用的软可及对象，对那些刚刚构建的或刚刚使用过的“新”软可反对象会被虚拟机尽可能保留。在回收这些对象之前，我们可以通过:
```Java
MyObject anotherRef=(MyObject)aSoftRef.get();  
```



## 弱引用(WeakReference)
弱引用也是用来描述非必需对象的，当JVM进行垃圾回收时，无论内存是否充足，都会**回收**被弱引用关联的对象。在java中，用java.lang.ref.WeakReference类来表示。
```Java
public static void main(String[] args) {  
    WeakReference<People>reference=new WeakReference<People>(new People("zhouqian",20));  
    System.out.println(reference.get());  
    System.gc();//通知GVM回收资源  
    System.out.println(reference.get());  
}  
}  
class People{  
  public String name;  
  public int age;  
  public People(String name,int age) {  
      this.name=name;  
      this.age=age;  
  }  
  @Override  
  public String toString() {  
      return "[name:"+name+",age:"+age+"]";  
  }  
}
```
以上代码的输出结果是：
```Java
[name:zhouqian,age:20]
null
```
第二个输出结果是null，这说明只要JVM进行垃圾回收，被弱引用关联的对象必定会被回收掉。不过要注意的是，这里所说的被弱引用关联的对象是指只有弱引用与之关联，如果存在强引用同时与之关联，则进行垃圾回收时也不会回收该对象（软引用也是如此）。上面的代码做出一点变动：
```Java
public class test {  
    public static void main(String[] args) {  
        People people=new People("zhouqian",20);  
        WeakReference<People>reference=new WeakReference<People>(people);//<span style="color:#FF0000;">关联强引用</span>  
        System.out.println(reference.get());  
        System.gc();  
        System.out.println(reference.get());  
    }  
}  
class People{  
    public String name;  
    public int age;  
    public People(String name,int age) {  
        this.name=name;  
        this.age=age;  
    }  
    @Override  
    public String toString() {  
        return "[name:"+name+",age:"+age+"]";  
    }  
}//结果发生了很大的变化
```
运行结果将是：
```java
[name:zhouqian,age:20]  
[name:zhouqian,age:20]  
```

## 虚引用(PhantomReference)
虚引用和前面的软引用、弱引用不同，它并不影响对象的生命周期。在java中用java.lang.ref.PhantomReference类表示。如果一个对象与虚引用关联，则跟没有引用与之关联一样，在任何时候都可能被垃圾回收器回收。

要注意的是，虚引用必须和引用队列关联使用，当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会把这个虚引用加入到与之 关联的引用队列中。程序可以通过判断引用队列中是否已经加入了虚引用，来了解被引用的对象是否将要被垃圾回收。如果程序发现某个虚引用已经被加入到引用队列，那么就可以在所引用的对象的内存被回收之前采取必要的行动。
```Java
public class Main {  
    public static void main(String[] args) {  
        ReferenceQueue<String> queue = new ReferenceQueue<String>();  
        PhantomReference<String> pr = new PhantomReference<String>(new String("hello"), queue);  
        System.out.println(pr.get());  
    }  
}
```


参考
* [Java对象的四种引用类型](http://blog.csdn.net/gs12software/article/details/51051813)
* [Java的四种引用方式](https://www.cnblogs.com/huajiezh/p/5835618.html)
