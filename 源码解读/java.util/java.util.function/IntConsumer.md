# IntConsumer

与`IntPredicate`的存在类似，`IntConsumer`也是为了避免基本数据类型的装箱拆箱操作而创建的函数式接口。结构如下：

![IntConsumer](https://ws4.sinaimg.cn/large/006tKfTcgy1fs4q62090yj30i403ojrp.jpg)

与`IntConsumer`类似的还有`LongConsumer`和`DoubleConsumer`，注意，并不是每一种基本数据类型都有对应的`Consumer`的。

其接口方法参考`Consumer`的源码解析。
