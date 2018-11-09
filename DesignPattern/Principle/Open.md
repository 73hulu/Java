# 开闭原则

> 一个软件实体如类、模块和函数应该对扩展开放，对修改关闭。


开闭原则非常抽象。总的来说，开闭原则是对前5种原则的总纲领：
* 单一职责原则告诉我们实现类要职责单一；
* 里氏替换原则告诉我们不要破坏继承体系；
* 依赖倒置原则告诉我们要面向接口编程；
* 接口隔离原则告诉我们在设计接口的时候要精简单一；
* 迪米特法则告诉我们要降低耦合。

而开闭原则是总纲，他告诉我们要对扩展开放，对修改关闭。用一句话总结就是：用抽象构建框架，用实现扩展细节：


参考
* [设计模式六大原则（6）：开闭原则](http://blog.csdn.net/zhengzhb/article/details/7296944)