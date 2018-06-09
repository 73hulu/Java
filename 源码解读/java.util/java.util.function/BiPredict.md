# BiPredict

`Predicate`的函数描述符是`T -> boolean`， 即对一个参数做出的判断。而`BiPredict`接口两个参数，函数描述符是`(T, U) -> boolean`。同样具有`and`、`negate`和`or`的逻辑。具体如下：

![BiPredict](https://ws1.sinaimg.cn/large/006tKfTcgy1fs4qljo1nlj30ic05uwfb.jpg)

具体方法及意义可以参考`Predicate`。
