# Chain of Responsibility

责任链模式。使多个对象都有机会处理请求，从而避免请求的发送者和接收者之间的耦合关系。将这个对象连成一条链，并沿着这条链传递该请求，直到有一个对象处理它为止。


![责任链模式](https://ws1.sinaimg.cn/large/006tNbRwly1fx1wrk2fkbj30qi0c6mxp.jpg)


责任链的重点在于“链”，由一条链去处理相似的请求在链中决定谁来处理这个请求，并返回相应的结果。这条链由多个处理者`ConcreteHandler`组成：
```java
// 抽象处理类
public abstract class Handler{
  private Handler successor;

  // 每个处理者都必须对请求做出处理，这注意final关键字
  public final Response handleMessage(Request request){
    Reponse response = null;
    // 判断是否是自己的处理级别
    if(this.getHandlerLevel().equals(request.getRequestsLevel())){
      response = this.echo(request);
    }else {
      // 判读是否有下一个处理这
      if(this.nextHandler ! = null){
        response = this.nextHandler.handleMessage(request);
      }else {
        // 没有适当的处理者，业务自行处理
      }
    }
    return response;
  }
  // 设置下一个处理者是谁
  public void setNext(Handler _handler){
    this.successor = _handler;
  }
  // 每个处理者都有一个处理级别
  public asbtract Level getHandlerLevel();
  // 每个吹着都必须实现处理任务
  public asbtract Response echo(Request request);
}

// 三个具体的处理者，构成一条链
public class ConcreteHandler1 extends Handler{
  // 定义自己的处理逻辑
  protected Response echo(Request request){
    // 完成处理逻辑
    return null;
  }
  // 设置自己的处理级别
  protected Level getHandlerLevel(){
    return null;
  }
}
public class ConcreteHandler2 extends Handler{
  // 定义自己的处理逻辑
  protected Response echo(Request request){
    // 完成处理逻辑
    return null;
  }
  // 设置自己的处理级别
  protected Level getHandlerLevel(){
    return null;
  }
}

public class ConcreteHandler3 extends Handler{
  // 定义自己的处理逻辑
  protected Response echo(Request request){
    // 完成处理逻辑
    return null;
  }
  // 设置自己的处理级别
  protected Level getHandlerLevel(){
    return null;
  }
}
// 模式中相关的框架代码
public class Level{
  // 定义一个请求和处理等级
}
// 定义请求
public class Request{
  // 请求的等级
  public Level getRequestsLevel(){
    return null;
  }
}
// 定义响应
public class Response{
  // 处理者返回的数据
}
```
场景类：
```java
public class Client{
  public static void main(String[] args){
    // 声明所有处理节点
    Handler handler1 = new ConcreteHandler1();
    Handler handler2 = new ConcreteHandler2();
    Handler handler3 = new ConcreteHandler3();

    // 设置链中的阶段熟悉怒 1->2 -> 3
    handler1.setNext(handler2);
    Handler2.setNext(handler3);
    //提交请求，返回结果
    Response response = handler1.handleMessage(new Request());
  }
}
```

责任链的好处不言而喻，请求和处理分开，请求者可以不用知道是谁处理的，矗立着也不需要知道处理的全貌。

这种方法有点类似于“击鼓传花”。每一个处理者在无法处理的时候将问题抛给下一个人，直到有一人解决这个问题。不同在于，游戏“击鼓传花”的参与者在鼓声停下来的时候需要得出结果，由解决问题的人（最后手握花球的人）需要将答案报给问题的提出者（击鼓者），而责任链模式中，问题的提出者只需要和链中第一个人有交流即可。

责任链模式的缺点一在于性能（当链非常长的时候性能越差），而在于调试困难（因为采取了类似于递归的方式）。


参考
* [JAVA 设计模式 职责链模式](http://www.cnblogs.com/jingmoxukong/p/4241496.html)
