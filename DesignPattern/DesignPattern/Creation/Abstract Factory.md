# Abstract Factory

## 什么是抽象工厂
抽象工厂模式，提供了一系列相关或相互依赖对象的**接口**，无需指定它们具体的**实现类**。是一种**类创建模式**。

抽象工厂的结构如下：

![Abstract Factory](https://ws2.sinaimg.cn/large/006tNbRwly1fx41b4jac9j30s00bkjsa.jpg)

这里面有这样几种角色：
1. 抽象工厂（AbstractFactory）：声明一个接口，这个接口中包含创建抽象产品对象的方法
2. 具体工厂（ConcreteFactory）：实现创建具体产品对象的方法。
3. 抽象产品（AbstractProduct）：声明一个接口，这个接口中包含产品对象类型。
4. 具体产品（ConcreteProduct）：定义一个产品对象，这个产品对象是由相关的具体工厂创建的。


以一个电子工厂、苹果厂商、三星厂商、手机、电脑为例子。

有一类工厂生产电子产品，名字叫`ElectronicFactory`，这种工厂生产两种商品：手机（`Telephone`）和电脑（`Computer`）。有两个具体的工厂，苹果工厂（`AppleFactory`）和三星工厂（`SamsungFactory`）分别生产自己的手机和电脑，那么以上关系就可以用抽象工厂模式来完成。

ElectronicFactory.java
```java
public interface ElectronicFactory {

    public Telephone productTelephone();

    public Computer productComputer();

}
```
Telephone.java

```java
public interface Telephone {
    public String getProductInfo();
}
```
Computer.java
```java
public interface Computer {
    public String getProductInfo();
}
```

AppleFactory.java
```java
public class AppleFactory implements ElectronicFactory {
    @Override
    public Telephone productTelephone() {
        return new AppleTelephone();
    }

    @Override
    public Computer productComputer() {
        return new AppleComputer();
    }
}
```

AppleTelephone.java
```java
public class AppleTelephone implements Telephone {

    @Override
    public String getProductInfo() {
        return "苹果手机，采用iso系统";
    }
}
```

AppleComputer.java
```java
public class AppleComputer implements Computer {

    @Override
    public String getProductInfo() {
        return "苹果电脑，使用mac系统";
    }
}
```

SamsungFactory.java
```java
public class SamsungFactotry implements ElectronicFactory {
    @Override
    public Telephone productTelephone() {
        return new SamsungTelephone();
    }

    @Override
    public Computer productComputer() {
        return new SamungComputer();
    }
}
```

SamsungTelephone.java
```java
public class SamsungTelephone implements Telephone {
    @Override
    public String getProductInfo() {
        return "三星手机，使用android系统";
    }
}
```
SamungComputer.java
```java
public class SamungComputer implements Computer {
    @Override
    public String getProductInfo() {
        return "三星电脑，使用windows系统";
    }
}
```

测试代码为：
```java

public static void main(String[] args) {
    ElectronicFactory factory = new AppleFactory();
    Telephone telephone = factory.productTelephone();
    Computer computer = factory.productComputer();

    System.out.println(telephone.getProductInfo());
    System.out.println(computer.getProductInfo());

    ElectronicFactory factory1 = new SamsungFactotry();
    Telephone telephone1 = factory1.productTelephone();
    Computer computer1 = factory1.productComputer();
    System.out.println(telephone1.getProductInfo());
    System.out.println(computer1.getProductInfo());

}
```
上面的测试代码创建了两个工厂创建得到的四种产品。当然你也可以用`new AppleComputer`或者`new AppleTelephone`的方法创建某种具体的产品，这样做的不好的地方是，你每次都需要手动去创建具体的类，而且，不能保证一个程序中都使用了一个工厂创建出来的一套产品（比如，自己new的话可能创建了一个苹果手机和一个三星电脑）。使用抽象工厂的模式创建类就避免了以上缺点：
1. 抽象工厂**隔绝了具体类的生成**。用户并不需要知道什么时候被创建。由于这种隔离，更换一个具体的工厂就变的容易。所有的具体的工厂都实现了抽象工厂中的方法，因此，只要改变工厂的具体实现，就可以在某种程度上改变整个软件系统的行为。
2. 当一个产品族中的多个对象被设计成一个工厂时，它能**保证客户端始终只是用同一个产品族中的对象**。就像上面的例子，如果`ElectronicFactory`的实现类是`AppleFactory`，那么之后由`Factory`创建出来的产品就是“苹果”的产品族。
3. 增加新的具体工厂和产品族很方便，不需要修改已经建成的“工厂”和“产品”，符合开闭原则。

但是也有缺点！

在添加一个新的产品时候，难以扩展抽象工厂生产出来新种类的产品。这是因为在抽象工厂角色中规定了所有可能被创建的产品集合。要支持新产品，意味着要对该抽象工厂进行扩展。一旦扩展，那么实现该抽象接口的所有具体工厂类都要实现该抽象方法，非常不方便。就像上面的例子，如果`ElectronicFactory`可以生产"相机"，那么`AppleFactory`和`SamsungFactotry`都需要实现“生产相机”的方法。如果有成千上百个工厂呢？修改的成本就非常高了。

## 抽象工厂的使用场景
一个对象族（或一组没有任何关系的对象）都有相同的约束，就可以使用抽象工厂。

> 一个普通的工厂通常是一个单例(Singleton)


参考
* [[设计模式]抽象工厂模式](http://www.cnblogs.com/jingmoxukong/p/4211446.html)
