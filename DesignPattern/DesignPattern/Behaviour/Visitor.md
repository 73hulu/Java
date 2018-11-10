# Visitor

访问者模式。封装一些作用于某种数据结构中的各元素的操作，它可以在不改变数据结构的前提下定义作用域这些元素的新的操作。

![Visitor](https://ws3.sinaimg.cn/large/006tNbRwly1fx2ya0yk73j30ok0jm75r.jpg)

有以下觉得：
* Visotor（抽象访问者）：抽象类或者接口，声明访问者可以访问哪些元素，具体到程序中就是visit方法的参数定义哪些对象是可以被访问的。
* ConcreteVisitor（具体访问者）：它影响访问者访问到一个类后该干什么，要做什么事情。
* Element（抽象元素）：接口或抽象类，声明接受哪一类访问者的访问，程序都是通过accepte方法中的参数来定义的。
* ConterteElement（具体元素）：通过实现accept方法，通常是visitor.visit(this)，基本上形成了一种模式。
* ObjectStruct（结构对象）：元素产生着，一般容纳咋多个不同类、不同接口的容器，如List、Set、Map等，在项目中，一般很少抽象出这个角色。

```java
// 抽象元素
public abstract class Element{
  // 定义业务逻辑
  public abstract void doSomething();
  // 允许谁来访问
  public abstract void accept(IVisotor visitor);
}

// 具体元素
public class ConcreteElement1 extends Element{
  // 完成业务逻辑
  public void doSomething(){

  }
  // 允许哪个访问者访问
  public void accept(IVisotor visitor){
    visotor.visit(this);
  }
}
public class ConcreteElement2 extends Element{
  // 完成业务逻辑
  public void doSomething(){

  }
  // 允许哪个访问者访问
  public void accept(IVisotor visitor){
    visotor.visit(this);
  }
}

// 抽象访问者
public interface IVisitor{
  // 可以访问哪些对象
  public void visit(ConcreteElement1 el1);
  public void visit(ConcreteElement2 el2);
}
// 具体的访问者
public class Visotor implement IVisitor{
  // 访问el1元素
  public void visit(ConcreteElement1 el1){
    el1.doSomething();
  }
  public void visit(ConcreteElement2 el2){
    el2.doSomething();
  }
}
// 结构对象
public class ObjectStruct{
  // 对象生成器，这里通过一个工厂方法模式模拟
  public static Element creteElement(){
    Random rand = new  Random();
    if(rand.nextInt(100) > 50){
      return new ConcreteElement1();
    }else {
      return new ConcreteElement2();
    }
  }
}
// 场景类
public class Client{
  public static void main(String[] args){
    for(int i = 0; i < 10; i++){
      // 获得元素对象
      Element el = ObjectStruct.creteElement();
      // 接受访问
      el.accept(new visitor());
    }
  }
}
```

访问者模式就是访问者在得到被访问者允许的情况下，转述并整理被访问者的内容。某被访问者的内容太多了，话不知道从何说起，那么它就允许一个访问者来整理并转述自己的话。

被访问者将信息暴露给了访问者，这是该模式的一个弊端。
