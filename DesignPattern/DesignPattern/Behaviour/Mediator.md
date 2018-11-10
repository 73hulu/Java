# Mediator

中介者模式。用一个中介对象封装一些列对象的交互，中介者使各对象不要要显示地相互作用，从而使其耦合松散，而且可以独立地改变它们之间的交互。

![中介者模式](https://ws2.sinaimg.cn/large/006tNbRwly1fx31g702b6j30fg094t8v.jpg)


有以下角色：
* Mediator（抽象中介者）：定义统一的接口，用于各同事角色之间的通信。
* ConcreteMeditor（具体中介者角色）：通过协调各同时角色实现协作行为，因此它必须依赖于各同时角色。
* Colleague（同事角色）：每个同事角色都知道中介者角色，而且与其他同时角色通信的时候，一定要通过中介者角色协作。每个同时类的行为分为两类：①同事本身行为，比如改变自身状态，处理自己的行为等，即自发行为，与其他的同事类或中介者没有任何的依赖。②依赖方法，必须依赖中介者才能完成的行为。

```java
// 抽象中介者
public abstract class Mediator{
  // 定义同事类 注意定义了具体的同事而不是抽象的
  protected ConcreteColleague1 c1;
  protected ConcreteColleague2 c2;
  // 通过getter或setter将其注入进来 略
  // 中介者模式的业务逻辑
  public abstract void doSomething();
  public abstract void doSomething();
}
// 通用中介者
public class ConctreteMediator extends Mediator{
  @Override
  public void doSomething1(){
    // 调用同事类的方法，只要是public方法都可以调用
    super.c1.selfMethod1();
    super.c2.selfMethod2();
  }

  @Override
  public void doSomething2(){
    super.c1.selfMethod1();
    super.c2.selfMethod2();
  }
}

// 抽象同事类
public abstract class Colleague{
  protected Mediator meditor;
  public Colleague(Mediator _mediator){
    this.mediator = _mediator;
  }
}
// 具体同时类
public class ConctreteColleague1 extends Colleague{
  // 通过构造函数传递终中介者
  public ConctreteColleague1(Mediator _mediator){
    super(_mediator);
  }
  // 自有方法self-method
  public void selfMethod1(){
      // 处理自己的业务逻辑
  }
  // 依赖方法 dep-method
  public void depMethod2(){
    // 处理自己的逻辑
    // 自己不能处理的逻辑，委托给终结者处理
    super.meditor.doSomething1();
  }
}

public class ConctreteColleague2 extends Colleague{
  // 通过构造函数传递终中介者
  public ConctreteColleague2(Mediator _mediator){
    super(_mediator);
  }
  // 自有方法self-method
  public void selfMethod2(){
      // 处理自己的业务逻辑
  }
  // 依赖方法 dep-method
  public void depMethod2(){
    // 处理自己的逻辑
    // 自己不能处理的逻辑，委托给终结者处理
    super.meditor.doSomething2();
  }
}
```
中介者模式将互相依赖的对象构成的网状结构简化成了星形拓扑，起到中场调度的作用，把原有一对一的依赖变成了一对一的依赖。


中介者模式在开发中常常用到，只是我们没有意识到。比如MVC框架中，前端控制器即“C”实际上就是M和V的一个中介者。另外媒体网关进行消息转发也是一个中介者。

但是注意不要滥用，在依赖关系过于复杂以及过于行为不确定的时候再用这种策略。
