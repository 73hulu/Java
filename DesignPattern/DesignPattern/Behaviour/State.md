# State

状态模式。当一个对象内在状态改变允许其改变行为，这个对象看起来像改变了其类。

状态模式的核心是封装，状态的变更引起了行为的变更，从外部看起来就好像这个对象对应的类好像发生了改变一样。

![状态模式](https://ws4.sinaimg.cn/large/006tNbRwly1fx2zk6w79bj30ok0jm75r.jpg)

有以下角色：
* State（抽象状态角色）：接口或抽象类，负责对象状态定义，并且封装环境角色以实现状态切换。
* ConcreteState（具体状态角色）：每一个具体状态必须完成两种职责：本状态的行为管理以及趋向状态处理，通俗地说，就是本状态下要做的事情，以及本状态如何过渡到其他状态。
* Context（环境角色）：定义客户端需要的接口，并且负责具体的状态切换。

```java
// 抽象环境角色
public abstract class State{
  // 定义一个环境角色，提供子类访问
  // 注意用protected来修饰
  protected Context Context;
  // 设计环境角色
  public void setContext(Context _context){
    this.context = _context;
  }
  // 行为1
  public abstract void handle1();
  // 行为2
  public abstract void handle2();
}
// 环境角色
public class ConcreteState1 extends State{
  @Override
  public void handle1(){
    // 本状态下必须处理的逻辑
  }
  @Override
  public void handle2(){
    // 设计当前状态为state2
    super.context.setCurrentState(Context.STATE2);
    // 过度到state2状态，由Context实现
    super.context.handle2();
  }
}

public class ConcreteState2 extends State{
  @Override
  public void handle2(){
    // 本状态下必须处理的逻辑
  }
  @Override
  public void handle1(){
    // 设计当前状态为state1
    super.context.setCurrentState(Context.STATE1);
    // 过度到state2状态，由Context实现
    super.context.handle1();
  }
}
// 具体环境角色
public class Context{
  // 定义状态
  public final static State STATE1 = new ConcreteState1();
  public final static State STATE2 = new ConcreteState2();
  // 当前状态
  private State currentState;

  // 获取当前状态
  public State getCurrentState(){
    return this.currentState;
  }

  // 设置当前状态
  public void setCurrentState(State currentState){
    this.currentState = currentState;
    // 切换状态
    this.currentState.setContext(this);
  }
  // 行为委托
  public void handle1(){
    this.currentState.handle1();
  }
  public void handle2(){
    this.currentState.handle2();
  }
}
//具体的环境角色
public class Client{
  public static void main(String[] args){
    // 定义环境角色
    Context context = new Context();
    // 初始化状态
    context.setCurrentState(new ConcreteState1());
    //行为执行
    context.handle1();
    context.handle2();
  }
}
```
注意这里的环境角色有两个不成文的约束：
1. 把状态独享声明为静态常量，有几个状态对象就声明几个静态常量。
2. 环境角色具有状态抽象橘色定义的所有行为，具体执行是由委托方式。

> 注意看上面的代码，状态对象和环境对象互相支持有对方。


状态模式就是一个一个“自动状态机”的实现。
