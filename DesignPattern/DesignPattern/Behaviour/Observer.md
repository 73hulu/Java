# Observer

观察者模式。定义了一种**一对多**的依赖关系，让多个观察者对象同时监听某一个主题对象。

这个主题对象在状态发生变化时，会通知所有观察者对象，使它们能够自动更新自己。


![观察者模式](https://ws4.sinaimg.cn/large/006tNbRwly1fx27dy5l2vj30hq0a9gnd.jpg)

观察者模式又叫做“发布订阅模式”。有以下角色：
* Subject被观察者：定义被观察者的职责，必须能够动态增加、取消观察者。一般是抽象类或者是实现类。仅仅完成作为被观察者必须实现的职责；管理观察者并通知观察者。
* Observer观察者：观察者接收到消息后，立即进行update，对接受到的消息记性处理。
* ConcreteSubject具体的被观察者：定义被观察者自己的业务逻辑，同时定义哪些事件进行通知、
* ConcreteObserver具体的观察者：每个观察在接受到消息后处理的反应不尽相同。

```java
// 被观察者
public abstract class Subject{
  // 定义一个观察者数组
  private List<Observes> observers = new ArrayList<Observer>;

  // 增加一个观察者
  public void addObserver(Observer o){
    this.observers.add(o);
  }
  // 删除一个观察者
  public void delObserver(Observer o){
    this.observers.remove();
  }
  // 通知所有观察者
  public void nortify(){
    for(Observer o : this.observers){
      o.update();
    }
  }
}

// 具体的被观察者
public class ConcreteSubject extends Subject{
  // 具体的业务
  public void doSomething(){
    super.notifyObservers();
  }
}
// 观察者
public interface Observer{
  // 更新方法
  public void udpdate();
}
// 具体的观察者
public class ConcreteObserver implements Observer{
  // 实现更新方法
  public void update(){
    System.out.println("接受到信息，并进行处理！");
  }
}
// 场景类
public class Client{
  public static void main(String[] args){
    // 创建一个被观察者
    ConcreteSubject subject = new ConcreteSubject();
    // 定义一个观察者
    Observer obs = new ConcreteObserver();
    // 观察者观察被观察者
    subject.addObserver(obs);
    // 观察者开始活动
    subject.doSomething();
  }
}
```

可以看到，观察者模式建立了一套触发机制，这种触发机制继而扩展，可以形成一个触发链。缺点在于Java中的消息通知默认是顺序执行的，一个观察者卡壳，会影响到整体的执行效率。这种情况下，一般考虑采取异步的方式（观察者采用多线程技术或者缓存技术来进行响应）。

观察者模式适用于
* 关联行为场景。注意关联行为是可拆分的而不是“组合”关系
* 事件多级触发场景
* 跨系统的消息交换场景。如消息队列的处理机制。

在Java中，`java.util.Observable`提供了暴露信息的能力，**被观察者应该实现该接口**。该接口结构如下：
![java.util.Observable](https://ws2.sinaimg.cn/large/006tNbRwly1fx2x8orixyj30fq0fe74e.jpg)

可以看到，这个接口和上文中我们自定义的`Subject`中的`nortify`、`addObserver`、`delObserver`的功能相同。对于观察者来说，Java提供了`java.util.Observer`，重写其中的`update`方法来实现观察功能。两者的具体内容可以参考【源码解读】部分相关博文。
