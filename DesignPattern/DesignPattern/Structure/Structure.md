# 结构型模式
结构型模式的目的是为了处理类或对象的组合。有以下7种设计模式：

##  适配器模式（Adapter） —— 转换
将一个类的接口转换成客户希望的另外一个接口。意见典型的例子是`Executors`中的`RunableAddapter`，这个方法将`Runable`的实例适配成`Callable`的实例。另外一个适配器模式的使用例子是`OutputStreamWriter`将`OutputStream`这种面向字节的IO流转化成面向字符的IO流。

所以这种设计模式比较适合于在已经存在实现但不满足要求的情况下，利用适配器来进行转化。适配器的写法有很多，比如依靠继承关系实现的叫做“类适配器”，还有接受对象参数的叫做“对象适配器”。

在JDK和扩展包中，以下类中的方法使用了适配器设计模式：
* `java.util.Arrays#asList()`
* `javax.swing.JTable(TableModel)`
* `java.io.InputStreamReader(InputStream)`
* `java.io.OutputStreamWriter(OutputStream)`
* `javax.xml.bind.annotation.adapters.XmlAdapter#marshal()`
* `javax.xml.bind.annotation.adapters.XmlAdapter#unmarshal()`

## 桥接模式(Bridge) —— 解耦
将抽象部分与实现部分分离，使它们都可以独立的变化。其目的在于**解耦**。

抽象部分非常好理解，一个顶层的接口，其实现类自有不同。如果想要实现一个新的能力，这个类监有该顶层接口的能力。那么可以定义一个抽象类，让其持有一个接口对象，形成**聚合关系**。然后基于这个抽象类再做具体的实现。这样，原接口的实现和具有更广功能的实现就能独立变化。

在JDK和扩展包中，以下类中的方法使用了桥接设计模式：

* AWT (提供了抽象层映射于实际的操作系统)
* JDBC

## 组合模式(Composite) —— 统一层级操作
将对象组合成树形结构以表示“部分-整体”的层次结构，使得对单个对象和组合对象的使用具有唯一性。

简单来说，我们在开发过程中会有很多对象层层包含的关系，能够组成树形结构，它们虽然意义不同，但是操作和数据有着类似的性质，只有细微的差别。如果我们对每一层的数据都进行定制化的开发，这样的做法是愚蠢的。组合模式的应用能够统一不同层级上节点的操作。

在JDK和扩展包中，以下类中的方法使用了组合设计模式：

* `javax.swing.JComponent#add(Component)`
* `java.awt.Container#add(Component)`
* `java.util.Map#putAll(Map)`
* `java.util.List#addAll(Collection)`
* `java.util.Set#addAll(Collection)`

## 装饰器模式(Decorator) —— 动态添加职责
动态地给一个对象添加一些额外的职责。就增加功能来说，Decorator 模式相比生成子类更为灵活。

在写法上，一个明显的特征是继承和对象持有。

在JDK和扩展包中，以下类中的方法使用了装饰器设计模式：


* `java.io.BufferedInputStream(InputStream)`
* `java.io.DataInputStream(InputStream)`
* `java.io.BufferedOutputStream(OutputStream)`
* `java.util.zip.ZipOutputStream(OutputStream)`
* `java.util.Collections#checked[List|Map|Set|SortedSet|SortedMap]()`

## 外观模式(Facade) —— 封装
“外观模式”也叫“门面模式”，目的是为了封装。外部系统可能会访问很多内容，为了不暴露内部系统的信息，可以将这个内部内容封装起来，封装到一个对象中，让外部系统只能通过这个对象进行访问，这就是外观模式。

在JDK和扩展包中，以下类中的方法使用了外观设计模式：

* `java.lang.Class`
* `javax.faces.webapp.FacesServlet`

## 享元模式(Flyweight) —— 共享池
使用共享对象可有效支持大量细粒度的对象。通俗来说，就是池技术。

吃技术的存在目的是为了共享对象，以减少内存开销。比如数据库连接池，限定了连接的上限，避免了无休止的连接请求。

在JDK和扩展包中，以下类中的方法使用了外观设计模式：
* `java.lang.String#intern()`
* `java.lang.Integer#valueOf(int)`
* `java.lang.Boolean#valueOf(boolean)`
* `java.lang.Byte#valueOf(byte)`
* `java.lang.Character#valueOf(char)`

## 代理模式(Proxy) —— 控制访问
非常重要的设计模式！也叫做委托模式。为其他对象提供一种代理以控制对这个对象的访问。

* `java.lang.reflect.Proxy`
* `java.rmi.*`
