# Command

即命令模式， 将一个请求封装为一个对象，从而使你可以用不同的请求对客户进行参数化；对请求排队或请求日志，以及支持可撤销的操作

不少Command模式的代码都是针对图形界面的，它实际就是菜单命令，我们在一个下拉菜单选择一个命令时，然后会执行一些动作。将这些命令封装成在一个类中，然后用户(调用者)再对这个类进行操作，这就是Command模式，换句话说，本来用户(调用者)是直接调用这些命令的，如菜单上打开文档(调用者)，就直接指向打开文档的代码，使用Command模式，就是在这两者之间增加一个中间者，将这种直接关系拗断，同时两者之间都隔离,基本没有关系了。

显然这样做的好处是符合封装的特性，降低耦合度，Command是将对行为进行封装的典型模式，Factory是将创建进行封装的模式。


![命令模式](http://img.blog.csdn.net/20170621121426157?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2p5dHRrbA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

在类图中可以看到三个角色：
* Receiver接收者：就是干活的角色，命令传递到这里是应该被执行的。
* Command命令角色：需要执行的所有命令都在这里声明。
* Invoker调用者角色：接收到命令并执行命令。
```java
// 通用的Receiver类
public abstract class Receiver{
  // 抽象接收者，定义每个接收者都必须完成的业务
  public abstract void doSomething();
}

// 具体的Receiver类 这种接收者可以很多，主要依赖业务的具体定义。
public class ConcreteReceiver1 extends Receiver{
  public void doSomething(){
  }
}

public class ConcreteReceiver2 extends Receiver{
  public void doSomething(){
  }
}

// 抽象的Command类。 这是命令模式的核心
public asbtract class Command{
  public abstract void execute();
}

// 根据环境的需求，具体的命令也可以由很多
public class ConcreteCommand1 extends Command{
  // 对哪个Receiver类进行命令处理
  private Receiver receiver;
  // 构造函数传递接收者
  public ConcreteCommand(Receiver _receiver){
    this.receiver = _receiver;
  }
  public void execute(){
    // 业务处理
    this.receiver.doSomething();
  }
}

// 调用者Invoker
public class Invoker(){
  private Command command;
  // 受气包，接受命令
  public Invoker(Command _command){
    this.command = _command;
  }
  public void action(){
    this.command.execute();
  }
}
```
以下是测试代码：
```java
public class Client{
  public static void main(String[] args){
    // 首先声明Invoker
    Invoker invoker = new Invoker();
    // 定义接收者
    Receiver receiver = new ConcreteReceiver1();
    // 定义一个发送给接收者的命令
    Command command = new ConcreteCommand(receiver);
    // 把命令交给调用者去执行
    invoker.setCommand(command);
    invoker.action();
  }
}

```

可以看出，命令模式中调用者角色（即上例中的Client）和接收者角色（即上例中的Receiver）之间没有任何依赖关系，调用者实现功能时只需要调用Command抽象类的execute方法即可，不需要了解到底是哪个执行的。并且Command的子类非常好扩展，和责任链模式相结合，实现命令簇解析任务，结合模板方法模式，就可以减少Comand子类的膨胀问题。

> 上例中Client中仍旧依赖了Receiver，那是因为在Command在构造方法中依赖了Receiver，一个更好的改进是在构造函数中自定义Receiver，这样就彻底封装起来了。


实际上，命令模式是生活中太常见的一个问题了。这就是甲方和乙方的关系，甲方（即调用者）只需要提出需求（即命令（Command）），然后交付给乙方的产品经理（即Invoker），对于每一种需求，大约都有一套既定的流程和实施者，此时经理把需求往需求池中一挂（即触发`command.execute`方法），自会有人（即Receiver来解决）。



参考
* [java设计模式之命令模式](https://www.cnblogs.com/liaoweipeng/p/5693154.html)
