# Builder

创建者模式将一个复杂对象的创建和表示相分离，使得同样的创建过程可以创建不同的表示。是一种对象创建型模式。

## 构造者模式的含义
使用创建者模式，用户只需要指定需要创建的的类型，具体的创建过程不需要知道。

![Builder](http://ovn0i3kdg.bkt.clouddn.com/Builder.png)

创建者模式中涉及到这几种角色：
1. 产品（Product）：这就是最终被需要的产品，产品由多个部件构成
2. 抽象创建者（AbstractBuilder）：抽象创建者，是一个接口，它确定产品由哪些部分构成，并提供一个得到产品建造后果的方法
3. 具体创建者（ConcreteBuilder）：实现抽象创建者中的具体方法。
4. 指挥类（Director）：指挥建造product的过程。
5. 客户端（Client）：用户不需要知道创建过程，而是指定一个指挥类，并给它配置一个创建者既可以了。


举一个例子：客户想到建造一个房子，房子由很多个部分构成，比如地基、屋顶、墙等。客户只需要指定一个设计师并给他配置一个建筑师，把建造房子的任务交给他们即可，自己不需要知道房子建造的过程。这个过程就是下面这样的：
```java
public class Test {
    public static void main(String[] args) {
        Builder builder = new Worker();
        Director director = new Director();
        director.conduct(builder);//设计师指导建筑师怎么建造房子
        House house = builder.getHouse();
        house.show();
    }
}
```
从上面代码可以看到，客户端没有参与到房子的建造中，他所做的工作就是指派一个设计师和一个建筑师，然后让设计师指导建筑师建造房子，最后由建筑师交付房子即可。

上面最关键的一个步骤就是设计师指导建筑师建造房子的过程：
```java
public class Director {
    public void conduct(Builder builder) {
        builder.buildFoundation();
        builder.buildWall();
        builder.buildRoof();
        builder.buildDoor();
        builder.buildWindow();
    }
}
```
而设计师不关心具体的操作的“人”是谁，他只关心来的人是一个建筑师，即Builder的实例，那么他就有建造房子的能力，那么他需要要求建筑师按照自己的“设计”来建造房子，这里的“设计”的意思比较简单，就是建造的顺序。

而`Builder`是建筑师这一个类别的抽象，它表明了建筑师应该会的几种技能：
```java
public interface Builder {
     void buildFoundation();
     void buildWall();
     void buildRoof();
     void buildWindow();
     void buildDoor();
     House getHouse();
}
```
而要建成的House是什么样的情况呢？
```java
public class House {

    List<String> parts = new ArrayList<>();

    public void addPart(String part){
        parts.add(part);
    }

    public void show(){
        for (String part : parts){
            System.out.println(part + " ");
        }
    }
}
```
一个house需要由很多个部分组成，所以其中定义了一个保存“部分”的线性表。

那么在真正实施的时候，真正的`Builder`需要将各个部分都创建好，并且将创建好的产品`Houser`返回。
```java
public class Worker implements Builder{
    private House house = new House();

    @Override
    public void buildFoundation() {
        house.addPart("foundation");
    }

    @Override
    public void buildWall() {
        house.addPart("wall");
    }

    @Override
    public void buildRoof() {
        house.addPart("roof");
    }

    @Override
    public void buildWindow() {
        house.addPart("window");
    }

    @Override
    public void buildDoor() {
        house.addPart("door");
    }

    @Override
    public House getHouse() {
        return house;
    }
}
```

## 建造者模式的优点
1. 封装性： 使得客户端不必知道产品内部组成的逻辑。
2. 建造者独立，容易扩展
3. 便于控制细节风险

## 建造者模式的使用场景
1. 相同的方法，不同的执行顺序，产生不同的时间处理结果，可以采用建造者模式
2. 多个部件或零件，都可以装配到一个对象中，但是产生的结果运行又不相同时候，可以使用该模式。
3. 产品类非常复杂， 或者产品类中调用顺序不同产生了不同的效能，可以使用这种方法。
4. 在对象创建过程中会使用到系统中一些其他对象，这些对象在产品对象的创建过程中不容易得到，也可以建造者模式封装该对象的创建过程。这种场景只能是一个补偿方法。

> 注意和其他创建型设计模式的区别。建造者模式关注的是零件类型和装配工艺（顺序）。

参考
* [[设计模式]建造者模式](http://www.cnblogs.com/jingmoxukong/p/4213402.html)
