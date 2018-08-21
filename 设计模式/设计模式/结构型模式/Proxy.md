# Proxy

代理模式也叫做为委托模式，为其他对象提供一种**代理**以控制对这个对象的**访问**。

![代理模式](http://ww1.sinaimg.cn/large/006rMFVegy1fdpnfxbh3oj30j60ayglq.jpg)


## 静态代理模式
```Java
//Subject接口的实现
public interface Subject {
    void visit();
}
```
有两个实现了Subject的实现类：
```Java
// RealSubject
public class RealSubject implements Subject {

    private String name = "byhieg";
    @Override
    public void visit() {
        System.out.println(name);
    }
}
```
```Java
// ProxySubject
public class ProxySubject implements Subject{

    private Subject subject;

    public ProxySubject(Subject subject) {
        this.subject = subject;
    }

    @Override
    public void visit() {
        subject.visit();
    }
}
```

具体的调用如下：
```Java
public class Client {

    public static void main(String[] args) {
        ProxySubject subject = new ProxySubject(new RealSubject());
        subject.visit();
    }
}
```
> 为什么和适配器那么像？但是有一点不一样，适配器是将一个接口A转化成另一个接口B，两者的身份并没有重叠。而代理模式中，代理对象和被代理对象都是同一个接口的实例，只是在代理者的handler中调用被的代理这的handler。

通过上面的代理代码，我们可以看出代理模式的特点，代理类接受一个Subject接口的对象，任何实现该接口的对象，都可以通过代理类进行代理，增加了通用性。但是也有缺点，每一个代理类都必须实现一遍委托类（也就是realsubject）的接口，如果接口增加方法，则代理类也必须跟着修改。其次，代理类每一个接口对象对应一个委托对象，如果委托对象非常多，则静态代理类就非常臃肿，难以胜任。

注意还可以定义强制代理。


### 动态代理

> 动态代理是结构型数据模式的核心，AOP就是采用了动态代理机制。

动态代理有别于静态代理，是根据代理的对象，动态创建代理类。这样，就可以避免静态代理中代理类接口过多的问题。动态代理是实现方式，是通过反射来实现的，借助Java自带的`java.lang.reflect.Proxy`，通过固定的规则生成。

其步骤如下：

1. 编写一个委托类的接口，即静态代理的（Subject接口）
2. 实现一个真正的委托类，即静态代理的（RealSubject类）
3. 创建一个动态代理类，实现`InvocationHandler`接口，并重写该`invoke`方法
4. 在测试类中，生成动态代理的对象。


第一二步骤，和静态代理一样，不过说了。
第三步，代码如下：
```java
public class DynamicProxy implements InvocationHandler {
    private Object object;
    public DynamicProxy(Object object) {
        this.object = object;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        Object result = method.invoke(object, args);
        return result;
    }
}
```


第四步，创建动态代理类：
```Java
Subject realSubject = new RealSubject();
DynamicProxy proxy = new DynamicProxy(realSubject);
ClassLoader classLoader = realSubject.getClass().getClassLoader();
Subject subject = (Subject) Proxy.newProxyInstance(classLoader, new  Class[]{Subject.class}, proxy);
subject.visit();
```

创建动态代理的对象，需要借助`Proxy.newProxyInstance`。该方法的三个参数分别是：
```Java
ClassLoader loader表示当前使用到的appClassloader。
Class<?>[] interfaces表示目标对象实现的一组接口。
InvocationHandler h表示当前的InvocationHandler实现实例对象。
```
关于动态代理的使用，我们就介绍到这里。


参考
* [Java设计模式之代理模式](https://www.cnblogs.com/qifengshi/p/6566752.html)
