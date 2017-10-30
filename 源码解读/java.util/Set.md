# Set

`Set`也是一个接口，直接继承`Collection`接口，不过它对元素做了一些限制。什么限制？**不包含重复元素**。更正式地说，`Set`不包含一对元素e1和e2，使得`e1.equals(e2)`和最多一个`null`元素。正如它的名字`Set`所暗示的那样，这个接口模拟了"数学集合"的抽象。`Set`的接口结构如下：

![Set](http://ovn0i3kdg.bkt.clouddn.com/Set,png)

可以看到这些抽象方法继承自其父类`Collection`。我们需要到具体的实现类才能具体的实现。`Set`接口有三个常用的实现类：`HashSet`、`LinkedHashSet`和`TreeSet`。
