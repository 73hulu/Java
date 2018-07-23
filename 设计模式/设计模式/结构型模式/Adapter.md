# Adapter

将一个类的接口转换成客户希望的另外一个接口。Adapter模式使得原本由于接口不兼容而不能一起工作的那些类可以一起工作。

> `Executors`中定义的`RunableAddapter`就是适配器模式的实践者。
> 适配器模式又叫做“变压器模式”，也叫“包装器模式”（Wrapper）。但是包装。但是包装器可不止这一个，它还包括装饰器模式。

### 适配器模型
* Target : 定义用户实际需要的接口。
```java
abstract class Target {
    public abstract void Request();
}
```
* Adaptee : 定义一个需要适配的接口。
```java
class Adaptee {
    public void SpecificRequest() {
        System.out.println("特殊请求");
    }
}
```
* Adapter : 通过在内部包装一个 Adaptee 对象，把源接口转换成目标接口。
```JAVA
class Adapter extends Target {

    private Adaptee adaptee = new Adaptee();

    @Override

    public void Request() {

        adaptee.SpecificRequest();
    }
}
```
* 测试代码
```java
public class AdapterPattern {

    public static void main(String[] args) {

        Target target = new Adapter();

        target.Request(); //特殊请求
    }
}
```

以上这种通过继承进行的适配，叫做“类适配器”。还有一种是对对象的合成，叫做“对象适配器”。
## 使用场景

想要使用一个已经存在的类，但如果它的方法不满足需求时；

两个类的职责相同或相似，但是具有不同的接口时要使用它；

应该在双方都不太容易修改的时候再使用适配器模式适配，而不是一有不同时就使用它。


> 适配器模型是一种补充模型。

参考
* [JAVA 设计模式 适配器模式](http://www.cnblogs.com/jingmoxukong/p/4224192.html)
