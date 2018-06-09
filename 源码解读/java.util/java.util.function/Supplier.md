# Supplier
`Supplier`的函数描述符是`void -> T`， 这与`Consumer`正好相反，可以用来创建对象。结构如下：

![Supplier](https://ws4.sinaimg.cn/large/006tKfTcgy1fs4r1totibj30hu03ajrg.jpg)

只有唯一的抽象方法`get`。

## T get();
这个方法用来获取产生的对象。
