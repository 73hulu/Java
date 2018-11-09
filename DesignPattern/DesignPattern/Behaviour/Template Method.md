# Template Method

定义了一个操作中的算法的**骨架**，而将部分步骤的实现在子类中完成。
模板方法模式使得子类可以不改变一个算法的结构即可重定义该算法的某些特定步骤。

注意，这是一个完全基于**继承**的代码复用的基本技术，没有关联关系。

模板方法模式需要开发抽象类和具体子类的设计师之间的协作。一个设计师负责给出一个算法的轮廓和骨架，另一些设计师则负责给出这个算法的各个逻辑步骤。

代表这些具体逻辑步骤的方法称做基本方法(primitive method)；而将这些基本方法汇总起来的方法叫做模板方法(template method)，这个设计模式的名字就是从此而来。

其UML图如下：

![模板设计模式](https://ws1.sinaimg.cn/large/006tNbRwly1fx1q6qgjifj30oa0hgt9o.jpg)


首先定义一个抽象类：
```java
// AbstractClass : 抽象类，定义并实现一个模板方法。这个模板方法定义了算法的骨架，而逻辑的组成步骤在相应的抽象操作中，推迟到子类去实现。顶级逻辑也有可能调用一些具体方法。
abstract class AbstractClass {

    public abstract void PrimitiveOperation1();

    public abstract void PrimitiveOperation2();

    public void TemplateMethod() {

        PrimitiveOperation1();

        PrimitiveOperation2();
    }
}
```
在定义该抽象类的不同实现：
```java
// ConcreteClass : 实现实现父类所定义的一个或多个抽象方法。
class ConcreteClassA extends AbstractClass {

    @Override

    public void PrimitiveOperation1() {
        System.out.println("具体A类方法1");
    }

    @Override
    public void PrimitiveOperation2() {
        System.out.println("具体A类方法2");
    }
}

class ConcreteClassB extends AbstractClass {

    @Override
    public void PrimitiveOperation1() {
        System.out.println("具体B类方法1");
    }

    @Override
    public void PrimitiveOperation2() {
        System.out.println("具体B类方法2");
    }    
}
```
测试代码如下：
```java
public class TemplateMethodPattern {

    public static void main(String[] args) {
        AbstractClass objA = new ConcreteClassA();
        AbstractClass objB = new ConcreteClassB();    
        objA.TemplateMethod();
        objB.TemplateMethod();
    }
}
```

这种设计模式适用于抽象具有共同流程的情景，例如茶和咖啡是随处可见的饮料。冲泡一杯茶或冲泡一杯咖啡的过程大致相同：
```
泡茶：
烧开水 ==> 冲泡茶叶 ==> 倒入杯中 ==> 添加柠檬
泡咖啡：
烧开水 ==> 冲泡咖啡 ==> 倒入杯中 ==> 添加糖和牛奶
```
都需要“烧水”、“冲泡”、“倒水”、“添加（可选）”的过程，那么这种情况就非常适用于模板设计模式：
```java
// 首先定义
abstract class Beverage {

    // 模板方法，决定了算法骨架。相当于TemplateMethod()方法
    public void prepareBeverage() {
        boilWater();
        brew();
        pourInCup();
        if (customWantsCondiments()){
            addCondiments();
        }
    }

    // 共性操作，直接在抽象类中定义
    public void boilWater() {
        System.out.println("烧开水");
    }

    // 共性操作，直接在抽象类中定义
    public void pourInCup() {
        System.out.println("倒入杯中");
    }

    // 钩子方法，决定某些算法步骤是否挂钩在算法中
    public boolean customWantsCondiments() {
        return true;
    }

    // 特殊操作，在子类中具体实现
    public abstract void brew();

    // 特殊操作，在子类中具体实现
    public abstract void addCondiments();
}
```
然后自定义具体的茶和咖啡类：
```java
class Tea extends Beverage {

    @Override
    public void brew() {
        System.out.println("冲泡茶叶");
    }

    @Override
    public void addCondiments() {
        System.out.println("添加柠檬");
    }
}
class Coffee extends Beverage {
    @Override
    public void brew() {
        System.out.println("冲泡咖啡豆");
    }

    @Override
    public void addCondiments() {
        System.out.println("添加糖和牛奶");
    }  
}
```

测试代码：
```java
public static void main(String[] args) {

    System.out.println("============= 准备茶 =============");
    Beverage tea = new Tea();
    tea.prepareBeverage();

    System.out.println("============= 准备咖啡 =============");
    Beverage coffee = new Coffee();
    coffee.prepareBeverage();
}
```
> 为了防止恶意破坏，一般的模板方法上面都加上了final防止重写

模板方法封装不变部分，扩展可变部分；提取公共代码，便于维护。但是不太符合一般的变成习惯：按照一般的设计习惯，抽象类负责声明最抽象、最一般的属性和方法，实现类完成具体的事物属性和方法。在末班设计模式中确实点到了，即抽象类定义了抽象部分，由子类实现，子类执行的结果影响了父类的结果，也就是子类对父类产生了影响。

模板方法使用的场景是：
* 多个子类有公共的方法，并且逻辑基本相同时。
* 重要，负责的算法，可以将核心算法设计为模板方法，周边的相关细节功能由各个子类实现。
* 重构时，模板方法模式是一个经常使用的模式，把相同的代码抽取到父类中，然后通过“钩子函数”约束其行为。
