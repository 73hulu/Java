# FilterWriter

抽象类，除了构造函数都重写了自父类`Write`中的方法。结构如下：

![FilterWriter](http://ovn0i3kdg.bkt.clouddn.com/FilterReader.png)


### protected FilterWriter(Writer out) {...}
构造方法。接受一个 `Writer`实例作为源，所以它是一个处理流而不是一个节点流。
```java
protected FilterWriter(Writer out) {
   super(out);
   this.out = out;
}
```
实际上，该类后面所有的方法实现都是将问题重新抛给了这个源自己去解决的。子类继承`FilterWriter`的话，必须重写其中的抽象方法，还要有自己的特色，但是目前为止，JDK中并没有一个提供一个`FilterWriter`的子类。
