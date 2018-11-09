# 设计模式

> * [java设计模式demo —— java-design-patterns](https://github.com/iluwatar/java-design-patterns)
> * 《设计模式之禅》

常见的23种设计模式，如果进行组织呢？

一种常见的组织方式是按照“目的”来划分，即模式是用来完成什么工作的，这种分类标准下，一共分为3大类。
1. 创建型
与对象的创建有关。有五种：
  * **工厂方法模式(Factory Method)**
  * **抽象工厂模式(Abstract Factory)**
  * 创建者模式(Builder)
  * **单例模式(Singleton)**
  * 原型模式(Prototype)

2. 结构型
处理类或对象的组合。有七种：
  * **适配器模式（Adapter）**
  * 桥接模式(Bridge)
  * **组合模式(Composite)**
  * **装饰器模式(Decorator)**
  * 外观模式(Facade)
  * 享元模式(Flyweight)
  * **代理模式(Proxy)**
3. 行为型
对类或对象怎样交互和怎样分配职责进行描述。共十一种：
  * 责任链模式(Chain of Responsibility)
  * 解释器模式(Interpreter)
  * **模板方法模式(Template Method)**
  * **命令模式(Command)**
  * 迭代子模式(Iterator)
  * 中介者模式(Mediator)
  * 备忘录模式(Memento)
  * **观察者模式(Observer)**
  * 状态模式(State)
  * **策略模式(Strategy)**
  * 访问者模式(Visitor)


还有一种分类标准是“范围”，即模式主要作用于类还是对象。
类模式处理类和子类之间的关系，这是通过继承关系完成的，在编译时刻便确定下来，是静态的。
对象处理模式处理对象间的关系，这些关系在运行时是可以变化的，具有动态性。大部分的设计模式是对象模式。


根据以上两个标准得到的设计模式的空间可以用下面这张表来表示：

![23种设计模式分类](http://ovn0i3kdg.bkt.clouddn.com/23%E7%A7%8D%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%E5%88%86%E7%B1%BB.png)


设计模式之间的关系可以用下面这张图来表示

![设计模式之间的关系](http://dl.iteye.com/upload/attachment/0083/1179/57a92d42-4d84-3aa9-a8b9-63a0b02c2c36.jpg?imageView/2/w/400/h/500)
