# Decorator

> **动态** 地给对象添加一些额外的 **职责**。就增加功能来说，Decorator模式相比生成子类更加灵活
> 允许向一个现有的对象添加新的功能，同时又不改变其结构。装饰者可以在所委托被装饰者的行为之前或之后加上自己的行为，以达到特定的目的。



![装饰者模式](http://img.blog.csdn.net/20160523105203595)


装饰器有两个角色：组件（被装饰者）和装饰者。
* 抽象组件（Component）：需要装饰的抽象对象。
* 具体组件（ConcreteComponent）：是我们需要装饰的对象
* 抽象装饰类（Decorator）：内含指向抽象组件的引用及装饰者共有的方法。
* 具体装饰类（ConcreteDecorator）：被装饰的对象。


举个例子：我们要了一杯咖啡，需要往里面加奶和糖等。这里，咖啡就是我们的组件，奶和糖就是我们的装饰者。现在我们来计算调制这样一杯咖啡需要花费多少。

![装饰着模式](http://img.blog.csdn.net/20160523113517058)


首先定义抽象组件，咖啡是一种饮料，饮料有的属性比如都需要花钱、都有一个描述，所以：
```Java
package DesignPattern.Strategy.Decorator;

public interface Drink {
    public float cost();
    public String getDescription();
}
```
之后就创建组件咖啡
```Java
package DesignPattern.Strategy.Decorator;

public class Coffee implements Drink {
    final private String description = "coffee";
    //每杯 coffee 售价 10 元
    public float cost() {
        return 10;
    }

    public String getDescription() {
        return description;
    }
}
```


接着创建我们的装饰者的抽象类，把它叫做“调味抽象类”，就是奶、糖这种装饰者的抽象类：
```Java
package DesignPattern.Strategy.Decorator;

public abstract class CondimentDecorator implements Drink {
    protected Drink decoratorDrink;

    public CondimentDecorator(Drink decoratorDrink) {
        this.decoratorDrink = decoratorDrink;
    }

    public float cost() {
        return decoratorDrink.cost();
    }

    public String getDescription() {
        return decoratorDrink.getDescription();
    }
}
```
可以看到，这个接口接受了一个Drink的对象，表示这个装饰着来装饰“什么东西”。

然后我们创建真正的装饰者：
```Java
//Suger 糖装饰者
package DesignPattern.Strategy.Decorator;

public class Sugar extends CondimentDecorator {
    public Sugar(Drink decoratorDrink) {
        super(decoratorDrink);
    }

    @Override
    public float cost() {
        return super.cost() + 1;
    }

    @Override
    public String getDescription() {
        return super.getDescription() + " sugar";
    }
}
//Milk牛奶装饰者
package DesignPattern.Strategy.Decorator;

public class Milk extends CondimentDecorator {
    public Milk(Drink decoratorDrink) {
        super(decoratorDrink);
    }

    @Override
    public float cost() {
        return super.cost() + 2;
    }

    @Override
    public String getDescription() {
        return super.getDescription() + " milk";
    }
}
```

怎么调制呢?
```Java
package DesignPattern.Strategy.Decorator;

public class CoffeeShop {
    public static void main(String[] args) {
        //点一杯coffee
        Drink drink = new Coffee();
        System.out.println(drink.getDescription() + ":" + drink.cost());
        //加一份奶
        drink = new Milk(drink);
        System.out.println(drink.getDescription() + ":" + drink.cost());
        //加一份糖
        drink = new Sugar(drink);
        System.out.println(drink.getDescription() + ":" + drink.cost());
        //再加一份糖
        drink = new Sugar(drink);
        System.out.println(drink.getDescription() + ":" + drink.cost());
    }
}
```
这个变化过程是一步步的，咖啡-》加了奶的咖啡-》加了糖的咖啡，但是它的本质是不变的，都是Drink，实际上，如果将`sugar`和`Milk`改成`CoffeeWithSuger`和`CoffeeWithMilk`更容易理解一些。

在Java I/O库中有两个对称性，他们分别是：
1. 输入-输出对称：比如InputStream 和OutputStream 各自占据Byte流的输入和输出的两个平行的等级结构的根部；而Reader和Writer各自占据Char流的输入和输出的两个平行的等级结构的根部。

2. byte-char对称：InputStream和Reader的子类分别负责byte和Char流的输入；OutputStream和Writer的子类分别负责byte和Char流的输出


这些作为根类，如果我们想通过缓冲，字节，或者是管道，这个时候我们就需要使用装饰器来进行装饰，然后通过装饰器来实现相应的操作，根类具有read方法，对于装饰类，通过构造函数将基类的一个实例注入进去，然后通过委托模式，首先通过基类的read方法获取字节流，然后根据相应的操作，实现字节读取等。
```Java
InputStreamReader input = new InputStreamReader(System.in);
BufferedReader reader = new BufferedReader(input);
String line = reader.readLine();
```


参考
* [设计模式（5）装饰器模式（讲解+应用）](https://segmentfault.com/a/1190000003796704)
* [ 设计模式 —— 装饰器模式（Decorator Pattern）](http://blog.csdn.net/wwh578867817/article/details/51480441)
* [JAVA 设计模式 装饰者模式](http://www.cnblogs.com/jingmoxukong/p/4226237.html)
