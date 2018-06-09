# IntPredicate
`IntPredicate`是为了避免基本数据类型装箱拆箱的开销而单独创建的针对基本数据类型int的函数式接口，其接口与`Predicate`没有两样。其结构如下：

![IntPredicate](https://ws3.sinaimg.cn/large/006tKfTcgy1fs4pwchjc4j30hw05mwf3.jpg)

与`IntPredicate`类似的还是有`LongPredict`和`DoublePredict`， 注意并不是每一种基本数据类型都有对应的`Predicate`的。

四个方法的内容不做具体展开，可以参考`Predicate`源码解读。
