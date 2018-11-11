# 结构型模式
结构型模式的目的是为了处理类或对象的组合。有以下7种设计模式：

##  [适配器模式（Adapter）](./Adapter.md) —— 转换
将一个类的接口转换成客户希望的另外一个接口。意见典型的例子是`Executors`中的`RunableAddapter`，这个方法将`Runable`的实例适配成`Callable`的实例。另外一个适配器模式的使用例子是`OutputStreamWriter`将`OutputStream`这种面向字节的IO流转化成面向字符的IO流。

所以这种设计模式比较适合于在已经存在实现但不满足要求的情况下，利用适配器来进行转化。适配器的写法有很多，比如依靠继承关系实现的叫做“类适配器”，还有接受对象参数的叫做“对象适配器”。


在JDK和扩展包中，以下类中的方法使用了适配器设计模式：
* `java.util.Arrays#asList()`
* `javax.swing.JTable(TableModel)`
* `java.io.InputStreamReader(InputStream)`
* `java.io.OutputStreamWriter(OutputStream)`
* `javax.xml.bind.annotation.adapters.XmlAdapter#marshal()`
* `javax.xml.bind.annotation.adapters.XmlAdapter#unmarshal()`

UML图如下：

![适配器模式](https://ws4.sinaimg.cn/large/006tNbRwly1fx26558flgj30w40f0gm9.jpg)

> 适配器模式和装饰器模式在通用的类图上没有太多的相似点，差别很大，但是它们的功能有相似的地方：都是包装作用，都是通过委托方式实现其功能。不同点在于：装饰器包装的自己的兄弟类，隶属于同一个家族（相同的接口或父类）；适配器模式则修饰非许愿关系类，把一个非本家族的对象伪装成本家族的对象，注意是伪装。

## [桥接模式(Bridge) ](./Bridge.md)—— 解耦
将抽象部分与实现部分分离，使它们都可以独立的变化。其目的在于**解耦**。

抽象部分非常好理解，一个顶层的接口，其实现类自有不同。如果想要实现一个新的能力，这个类监有该顶层接口的能力。那么可以定义一个抽象类，让其持有一个接口对象，形成**聚合关系**。然后基于这个抽象类再做具体的实现。这样，原接口的实现和具有更广功能的实现就能独立变化。

在JDK和扩展包中，以下类中的方法使用了桥接设计模式：

* AWT (提供了抽象层映射于实际的操作系统)
* JDBC

UML图如下：

![桥接模式](https://ws4.sinaimg.cn/large/006tNc79gy1ftix9kmbboj30f2077jro.jpg)

## [组合模式(Composite)](./Composite.md) —— 统一层级操作
将对象组合成树形结构以表示“部分-整体”的层次结构，使得对单个对象和组合对象的使用具有唯一性。

简单来说，我们在开发过程中会有很多对象层层包含的关系，能够组成树形结构，它们虽然意义不同，但是操作和数据有着类似的性质，只有细微的差别。如果我们对每一层的数据都进行定制化的开发，这样的做法是愚蠢的。组合模式的应用能够统一不同层级上节点的操作。

在JDK和扩展包中，以下类中的方法使用了组合设计模式：

* `javax.swing.JComponent#add(Component)`
* `java.awt.Container#add(Component)`
* `java.util.Map#putAll(Map)`
* `java.util.List#addAll(Collection)`
* `java.util.Set#addAll(Collection)`

UML图如下：

![组合设计模式](https://ws1.sinaimg.cn/large/006tNbRwly1fx26aspk83j30pq0j4dgp.jpg)


## [装饰器模式(Decorator)](./Decorator.md) —— 动态添加职责
动态地给一个对象添加一些额外的职责。就增加功能来说，Decorator 模式相比生成子类更为灵活。

在写法上，一个明显的特征是继承和对象持有。

在JDK和扩展包中，以下类中的方法使用了装饰器设计模式：


* `java.io.BufferedInputStream(InputStream)`
* `java.io.DataInputStream(InputStream)`
* `java.io.BufferedOutputStream(OutputStream)`
* `java.util.zip.ZipOutputStream(OutputStream)`
* `java.util.Collections#checked[List|Map|Set|SortedSet|SortedMap]()`

UML图如下：

![装饰者模式](https://ws1.sinaimg.cn/large/006tNbRwly1fx41j36e1oj30eo0dogmc.jpg)

> 装饰器模式和代理模式的类图基本一致，实际上，装饰器就是代理模式一个特殊应用。两者的共同点是都具有相同的接口，不同点则是代理模式着重对代理过程的控制（即对类的行为有决定权），而装饰器模式则是对类的功能进行加强或减弱，它着重类的功能变化。

## [外观模式(Facade)](./Facade.md) —— 封装
“外观模式”也叫“门面模式”，目的是为了封装。外部系统可能会访问很多内容，为了不暴露内部系统的信息，可以将这个内部内容封装起来，封装到一个对象中，让外部系统只能通过这个对象进行访问，这就是外观模式。

在JDK和扩展包中，以下类中的方法使用了外观设计模式：

* `java.lang.Class`
* `javax.faces.webapp.FacesServlet`

UML图如下：

![门面模式示意图](https://ws2.sinaimg.cn/large/006tKfTcgy1ftj3h9nxxcj30eo0fgmz0.jpg)

## [享元模式(Flyweight)](./Flyweight.md) —— 共享池
使用共享对象可有效支持大量细粒度的对象。通俗来说，就是池技术。

吃技术的存在目的是为了共享对象，以减少内存开销。比如数据库连接池，限定了连接的上限，避免了无休止的连接请求。

在JDK和扩展包中，以下类中的方法使用了外观设计模式：
* `java.lang.String#intern()`
* `java.lang.Integer#valueOf(int)`
* `java.lang.Boolean#valueOf(boolean)`
* `java.lang.Byte#valueOf(byte)`
* `java.lang.Character#valueOf(char)`

UML图如下：

![享元模式](https://ws4.sinaimg.cn/large/006tNbRwly1fx26dn4xnlj30k50bbq3u.jpg)

## [代理模式(Proxy)](./Proxy.md) —— 控制访问
非常重要的设计模式！也叫做委托模式。为其他对象提供一种代理以控制对这个对象的访问。

* `java.lang.reflect.Proxy`
* `java.rmi.*`

UML图如下：

![代理模式](http://ww1.sinaimg.cn/large/006rMFVegy1fdpnfxbh3oj30j60ayglq.jpg)
