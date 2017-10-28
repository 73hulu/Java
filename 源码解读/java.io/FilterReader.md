# FilterReader

这是一个抽象类，除了构造方法其他方法都重写了来自父抽象类`Reader`的方法。子类必须重写这些方法并且有自己的特色。结构如下：

![FilterReader](http://ovn0i3kdg.bkt.clouddn.com/FilterReader.png)



###  protected FilterReader(Reader in){...}
构造方法，接受一个`Reader`实例作为源，可见`FilterReader`是一个处理类而不是一个节点流。定义如下：
```java
protected FilterReader(Reader in) {
    super(in);
    this.in = in;
}
```

之后的所有方法都是借助这个源本身的同名方法进行的。`FilterReader`的子类必须重写其中的抽象方法，还得有自己特色的方法。JDK提供了这样一个`FilterReader`的子类`PushbackReader`。
