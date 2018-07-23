# Facade

## 门面模式
门面模式也叫做外观模式，是一种比较常见的封装模式。要求一个子系统的外部与其内部的通信必须通过一个统一的对象进行。是一个高层次的接口，使得子系统更易于使用。

![门面模式示意图](https://ws2.sinaimg.cn/large/006tKfTcgy1ftj3h9nxxcj30eo0fgmz0.jpg)

这里需要明确一下门面模式的角色
* Facade门面角色 ： 客户端可以调用这个角色的方法。此角色知晓子系统的所有功能和责任。会将所有从客户端发来的请求委派到相应的子系统去，也就是说该角色没有实际的业务逻辑，只是一个委托类。
* subsystem子系统角色： 可以同时有一个或者多个子系统，每个子系统都不是一个单独的类，而是一个类的集合。子系统并不知道门面的存在，对于子系统而言，门面仅仅是另外一个客户端而已。

> 动态地给一个对象添加一些额外的职责。就增加功能来说，Decorator 模式相比生成子类更为灵活。

子系统
```
public class ClassA{
  public void doSomethingA{
    // 业务逻辑
  }
}

public class ClassB{
  public void doSomethingB(){
    // 业务逻辑
  }
}

public class ClassC{
  public void doSomethingC{
    // 业务逻辑
  }
}
```
门面对象
```JAVA
public class Facade {
  // 被委托的对象
  private ClassA a = new ClassA();
  private ClassB b = new ClassB();
  private ClassC c = new ClassC();

  // 提供给外部访问的方法
  public void methodA(){
    this.a.doSomethingA()
  }

  public void methodB(){
    this.b.doSomethingB();
  }
  public void methodC(){
    this.c.doSomethingC();
  }
}
```
## 门面模式的优缺点
### 优点
1. 减少系统间的依赖
2. 提高灵活性：不管子系统内部如何变化，只要不影响到门面对象，任你自由活动。
3. 提高安全性

### 缺点
不符合开闭原则。一旦投产以后，如果发现错误，没有办法解决，继承、覆写都不管用，唯一能做的是修改门面角色的代码。

## 门面模式的使用场景
* 为复杂的模块或子系统提供一个供应外界访问的接口
* 子系统相对独立——外界对子系统的访问只需要黑箱操作即可
* 预防低水平人员带来的风险扩散



参考
* [JAVA 设计模式 装饰者模式](http://www.cnblogs.com/jingmoxukong/p/4226237.html)
