# Bridge

将抽象部分与实现部分分离，使它们都可以独立的变化。

重点在于解耦。

![桥接模式](https://ws4.sinaimg.cn/large/006tNc79gy1ftix9kmbboj30f2077jro.jpg)

## 桥接模式
`Implementor` : 定义实现接口
```JAVA
interface Implementor {
    // 实现抽象部分需要的某些具体功能
    public void operationImpl();
}
```

`Abstraction` : 定义抽象接口
```JAVA
abstract class Abstraction {

    // 持有一个 Implementor 对象，形成聚合关系
    protected Implementor implementor;

    public Abstraction(Implementor implementor) {
        this.implementor = implementor;
    }

    // 可能需要转调实现部分的具体实现
    public void operation() {
        implementor.operationImpl();
    }
}
```
`ConcreteImplementor`: 实现 Implementor 中定义的接口
```JAVA
class ConcreteImplementorA implements Implementor {
    @Override
    public void operationImpl() {
        // 真正的实现
        System.out.println("具体实现A");
    }    
}

class ConcreteImplementorB implements Implementor {
    @Override
    public void operationImpl() {
        // 真正的实现
        System.out.println("具体实现B");
    }    
}
```
`RefinedAbstraction`: 扩展 Abstraction 类
```JAVA
class RefinedAbstraction extends Abstraction {

    public RefinedAbstraction(Implementor implementor) {
        super(implementor);
    }
    public void otherOperation() {
        // 实现一定的功能，可能会使用具体实现部分的实现方法,

        // 但是本方法更大的可能是使用 Abstraction 中定义的方法，

        // 通过组合使用 Abstraction 中定义的方法来完成更多的功能。

    }
}
```
测试代码：
```JAVA
public class BridgePattern {

    public static void main(String[] args) {

        Implementor implementor = new ConcreteImplementorA();

        RefinedAbstraction abstraction = new RefinedAbstraction(implementor);

        abstraction.operation(); // 具体实现A

        abstraction.otherOperation(); // 其他操作
    }
}
```
## 桥接模式的优点
1. 抽象和实现的分离：这是桥梁模式的主要特点，它完全是为了解决继承的缺点而提出的设计模式，在该模式下，实现可以不受抽象的约束，不用再绑定在一个固定的抽象层次上。
2. 优秀的扩充能力：
3. 实现细节对客户透明：客户不用关系细节的实现，它已经由抽象层通过聚合关系完成了封装。

## 桥接模式的应用场景

1. 不希望或不适合使用继承的场景
2. 接口或抽象类不稳定的场景
3. 重用性要求比较高的场景：设计的颗粒度越细，则被重用的可能性就越大，而采用继承则受到父类的限制，不可能出现太细的颗粒度。










参考
* [JAVA 设计模式 桥接模式](http://www.cnblogs.com/jingmoxukong/p/4224661.html)
