# BiFunction

`Function`的函数描述符是`T -> R`， 而`BiFunction`的函数描述符是`(T, U) -> R`（当然这一点其实也可以通过`Function`的`compose`方法做到），结构如下：

![BiPredict](https://ws4.sinaimg.cn/large/006tKfTcgy1fs4quo1v77j30i20463yu.jpg)


`BiFunction`相比`Function`而言，其方法上有一定的剪枝，少了`compose`和`identity`方法。具体可以参考`Function`的源码解读。
