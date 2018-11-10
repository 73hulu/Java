# 创建型模式

创建型模式的目的是为了创建对象。

常见的有以下5种 + 1种（这种叫“简单工厂”，它不被列入到[23种](http://ichennan.com/2016/08/09/DesignPattern.html)设计模式中），下面是这六种设计模式的简单概括：

## 简单工厂（Simple Factory）—— 上帝模式
也叫“静态工厂模式”、“上帝工厂”，即用参数控制，想要什么就得到什么。例如传入1造火箭、传入2煮面条、传入3种土豆，简直无所不能。可以想象内部的实现就是就是一堆if-else的判断。

缺点很明显，太简单粗暴了。任何东西都能被得到，负担太重，当需要的产品很多的时候，工厂方法的代码量可能会很庞大，毫无设计感可言。另外，在遵循开闭原则的条件下，简单工厂对于增加新产品无能为力。因为增加新产品只能靠修改工厂方法来实现。一般不要用，所以它也不在23种设计模式之中。



## [抽象工厂模式（Abstract Factory）](./Abstract Factory.md)—— 产品簇
提供了一系列相关或相互依赖对象的接口，无需指定它们具体的实现类。说明白点就是面向接口编程，制定了一类工厂的全部功能，然后让不同的实现类去实现这个工厂的所有能力。所有它的好处就是提供了一个产品簇（全都归属于某个特定工厂）。缺点在于如果工厂增加了某项能力，需要从顶层接口处修改，那么将会影响所有的工厂实现（全部要去实现这个新的能力），不符合开闭原则。

UML图如下：

![Abstract Factory](http://ovn0i3kdg.bkt.clouddn.com/Abstract%20Factory.png)

在JDK和扩展包中，以下类中的方法使用了抽象工厂设计模式：
* `java.util.Calendar#getInstance()`
* `java.util.Arrays#asList()`
* `java.util.ResourceBundle#getBundle()`
* `java.net.URL#openConnection()`
* `java.sql.DriverManager#getConnection()`
* `java.sql.Connection#createStatement()`
* `java.sql.Statement#executeQuery()`
* `java.text.NumberFormat#getInstance()`
* `java.lang.management.ManagementFactory (所有getXXX()方法)`
* `java.nio.charset.Charset#forName()`
* `javax.xml.parsers.DocumentBuilderFactory#newInstance()`
* `javax.xml.transform.TransformerFactory#newInstance()`
* `javax.xml.xpath.XPathFactory#newInstance()`

## [工厂方法模式(Factory Method)](./Factory Method.md) —— 延迟实现
定义一个用于创建对象的接口，让子类决定实例化哪一个类，工厂方法使一个类的实例化延迟到其子类。

简单来说，就是顶层的接口定义了某个事物具有一项能力，比如说“工厂能生产产品”，至于生产什么产品，顶层接口中并不定义，需要在实现了中实现。这种设计模式避免了抽象工厂的弊端（工厂方法中每个实现类只负责一项功能，如果要增加新能力，则需要重新写一个实现类来实现该能力，对已实现的实现类不需要修改），缺点是功能单一，如果需要n种产品，那么就需要n个工厂的实现类。解决的办法是在实现类之上封装一个调度者，避免调用者和工厂类直接联系，这种方法叫做“多工厂方法”。

工厂方法是一种极端的抽象工厂。“极端之处”在于一个工厂只生产一种产品。

在JDK和扩展包中，以下类中的方法使用了工厂方法设计模式：
* `java.lang.Object#toString()` (在其子类中可以覆盖该方法)
* `java.lang.Class#newInstance()`
* `java.lang.Integer#valueOf(String)` (Boolean, Byte, Character,Short, Long, Float 和 Double与之类似)
* `java.lang.Class#forName()`
* `java.lang.reflect.Array#newInstance()`
* `java.lang.reflect.Constructor#newInstance()`

UML图如下：

![Factory Method](http://img.blog.csdn.net/20160828082911344)


## [创建者模式(Builder)](./Builder.md) —— 配件装配
将一个复杂对象的创建和表示相分离，使得同样的创建过程可以创建不同的表示。一个复杂的对象可以看做是一个工艺品，不同零件的装配顺序不同如果会导致不同的结果，那么这种创建模式就非常合适。
它的实现机制就是将零件的不同装配阶段抽象出来，然后依次组装。比如“造房子”这件事，可以抽象为“雇主”、“设计师”、“建造者”来分别进行，不同的抽象实现最后得到的房子也是不一样的。而我们只用关心最初的抽象（即雇主指派设计师设计）和最后的产出（设计师交付房子），而中间的实现（设计师指派建造者造房子，建造者劳动造房子）是被封装好的，不需要关心。

在JDK和扩展包中，以下类的方法使用了创建者模式：
* `java.lang.StringBuilder#append()`
* `java.lang.StringBuffer#append()`
* `java.nio.ByteBuffer#put()` (CharBuffer, ShortBuffer, IntBuffer,LongBuffer, FloatBuffer 和DoubleBuffer与之类似)
* `javax.swing.GroupLayout.Group#addComponent()`
* `java.sql.PreparedStatement`
* `java.lang.Appendable`的所有实现类

UML图如下：

![Builder](http://ovn0i3kdg.bkt.clouddn.com/Builder.png)


## [单例模式(Singleton)](./Singleton.md) —— 独生子
保证一个类只要一个实例。这一特别的需求有的是因为某些事物本身不能存在多个（比如只能接一个打印机，那么就只能有一个打印机实例），有的是因为要节省内存消耗，不能产生多个实例。

这是常考的一种设计模式，除了需要注意一些高级的单例模式的写法，在开发中还需要注意多线程对共享单例的影响。

在JDK和扩展包中，以下类的方法使用了单例模式：
* `java.lang.Runtime#getRuntime()`
* `java.awt.Desktop#getDesktop()`

UML图如下：

![Singleton](http://ovn0i3kdg.bkt.clouddn.com/Singleton.png)


## [原型模式(Prototype)](./Prototype.md) —— 快速大量拷贝
用原型实例指定创建对象的种类，并通过拷贝这些原型创建的对象。简单老说，需要在**大量创建**某一类对象时，这些对象只有细微的差别。这时候将类设计成一个模板（即其属性内容与设定为最广泛、最普通的值），这个类实现`Clonable`接口并重写`clone`方法（注意深拷贝和浅拷贝）。在使用的时候，先创建一个对象，然后调用该对象的`clone`方法来产生新对象并修改不同的地方。

这种方法特别适合创建大规模的对象，其原因在于`clone`是在内存中以二进制流的形式进行拷贝，比new一个对象快多了。另外还需注意，这种方法不会调用构造函数。

在JDK和扩展包中，以下类的方法使用了原型模式：
* `java.lang.Object#clone()` (支持浅克隆的类必须实现java.lang.Cloneable接口)

UML图如下：

![Prototype](http://ovn0i3kdg.bkt.clouddn.com/prototype.png)
