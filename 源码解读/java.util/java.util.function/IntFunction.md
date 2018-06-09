# IntFunction

为了避免基本数据类型与其包装类之间转换的开销，Java中的八种基本数据类型都有其对应的`Function`接口。不同的是，`Function`的函数描述符是`T -> R`。接口中有4个方法，而`xxxFunction`中只有主要的抽象方法`apple`，函数描述符是`int -> R`。

![IntFunction](https://ws4.sinaimg.cn/large/006tKfTcgy1fs4q8ufp12j30i6030glp.jpg)

与`IntFunction`类似的还有`LongFunction`和`DoubleFunction`，注意并不是每种基本数据类型都有对应的`Function`。
