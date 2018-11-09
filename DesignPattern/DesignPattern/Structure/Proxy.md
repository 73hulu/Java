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

> * 动态代理是结构型数据模式的核心，AOP就是采用了动态代理机制。
> * 动态代理是在实现阶段不关心代理谁，而在运行阶段才指定哪一个对象（如何实现？显然是反射）。
> * 相对来说，自己写的代理类的方法就是静态代理。
> * 动态代理的首要条件是：被代理类必须要实现一个接口。当然也有很多技术如CGLIB可以实现不需要接口也可以实现动态代理的方式。

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
        Object result = method.invoke(this.object, args);
        // 这里可以获取到method的信息，来进行相应的操作，比如如果是“login”方法则发送一条信息
        if(method.getName().equalsIgnoreCase("login")){
          System.out.println("login !");
        }
        return result;
    }
}
```


第四步，创建动态代理类：
```Java
Subject realSubject = new RealSubject();
InvocationHandler handler = new DynamicProxy(realSubject);
ClassLoader classLoader = realSubject.getClass().getClassLoader();
Subject subject = (Subject) Proxy.newProxyInstance(classLoader, new  Class[]{Subject.class}, handler);
subject.visit();
```

创建动态代理的对象，需要借助`Proxy.newProxyInstance`。该方法的三个参数分别是：
```Java
ClassLoader loader表示当前使用到的appClassloader。
Class<?>[] interfaces表示目标对象实现的一组接口。
InvocationHandler h表示当前的InvocationHandler实现实例对象。
```



## 为什么要用代理
前面说到，代理模式的目的是为了控制访问。为什么控制？原因可以有很多，屏蔽实现、安全...

比如上面的动态代理中，如果我们对`DynamicProxy`在做一种改变，在构造该对象的时候传入名称，然后在该类内部实例化该名称即可。这样对于调用者来说，只用根据名称就能得到代理类的对象，“只知代理的存在，而不知道具体操作对象的存在”，这就是对外屏蔽了实现细节。

还有一种情况是强制代理。什么意思呢？就是说无论是自己new一个`RealSubject`还是创建一个`DynamicProxy`，最后返回的都是由`RealSubject`指定的特定的代理，否则办不成事。比如下面是一个游戏账号和代理游戏账号的类定义：
```java
// 强制代理的接口类
public interface IGamePlayer{
  // 登录游戏
  public void login(String username, String password);
  // 杀怪，这是网络游戏的主要特点
  public void killBoss();
  // 升级
  public void upgrade();
  // 每个人都可以找一下自己的代理
  public IGamePlayer getProxy();
}
// 强制代理的真实角色
public class GamePlayer implements IGamePlayer{
  private String name;
  // 我的代理
  private IGamePlayer proxy = null;

  public GamePlayer(String _name){
    this.name = _name;
  }
  // 找到自己的代理
  public IGamePlayer getProxy(){
    this.proxy = new GamePlayerProxy(this.name);
    return this.proxy;
  }

  public void login(String username, String password){
    if(this.isProxy()){
      System.out.println(this.name);
    }else {
      System.out.println('请使用指定的代理');
    }
  }

  // 打怪
  public void killBoss(){
    if(this.isProxy){
      System.out.println(this.name);
    }else {
      System.out.println('请使用指定的代理');
    }
  }

  // upgrade同理
  // 检查是否有代理
  private boolean isProxy(){
    return this.proxy != null
  }  
}

// 强制代理的代理类
public class GamePlayerProxy implements IGamePlayer{
  private IGamePlayer gamePlayer = null;

  public GamePlayerProxy(IGamePlayer _gameplayer){
    this.gamePlayer = _gameplayer;
  }

  // 代练杀怪
  public void killBoss(){
    this.gamePlayer.killBoss();
  }

  public void login(String user, String password){
    this.gamePlayer.login(user, name);
  }
  // 其他类似
  public IGamePlayer getProxy(){
    return this;
  }
}
```
这时候，直接访问真实角色：
```java
public static void main(String[] main){
  // 定义一个游戏角色
  IGamePlayer player = new GamePlayer('张三');
  // 开始打游戏
  player.login('张三', '123');
  //开始杀怪
  player.killBoss();
  // 升级
  player.upgrade();
}
```
以上代码从登陆开始就会提示“指定代理”，其原因在于内部`proxy`内有实例。行，那就创建一个代理实例：
```java
public static void main(String[] args){
  // 定义一个游戏角色
  IGamePlayer player = new GamePlayer('张三');
  // 定义一个代练者
  GamePlayerProxy proxy = new GamePlayerProxy(player);
  // 开始打游戏
  // 开始打游戏
  proxy.login('张三', '123');
  //开始杀怪
  proxy.killBoss();
  // 升级
  proxy.upgrade();
}
```
会发现，上面这段代码还是提示“指定代理”。诶？我不是创建了代理了么？创建了一个实例没错，问题在于，创建的这个代理并非真实类**指定的代理**，随便一个代理实例是没有用的。所以这段代码改成下面这样就可以了：
```java
public static void main(String[] args){
  // 定义一个游戏角色
  IGamePlayer player = new GamePlayer('张三');
  // 获得指定的代理
  GamePlayerProxy proxy = player.getProxy();
  // 开始打游戏
  // 开始打游戏
  proxy.login('张三', '123');
  //开始杀怪
  proxy.killBoss();
  // 升级
  proxy.upgrade();
}
```
这就OK了。**强制代理的概念就是从真实角色查到代理角色，不允许访问真实角色。**

上一个例子中的代理似乎只是将真实类进行了简单的包装，实际上，代理类可以为真是角色进行预处理消息、过滤消息、消息转发、事后消息处理等功能。例如增加一个`IProxy`接口实现计费功能：
```java
public interface IProxy{
  void count();
}
```
代理类叠加实现游戏练级和计费的功能：
```java
public class GamePlayerProxy implements IGamePlayer, IProxy{
  // 其他不变

  public void upgrade(){
    this.gamePlayer.upgrade();
    this.count();
  }

  public void count(){
    System.out.println('升级总费用是：150元');
  }
}
```
这样在使用代理实例进行升级的时候，既能完成账号升级也能计费。

> “代理模式”这部分内容强烈建议阅读《设计模式之禅》这部分的相关部分，写的非常有启发性。


参考
* [Java设计模式之代理模式](https://www.cnblogs.com/qifengshi/p/6566752.html)
