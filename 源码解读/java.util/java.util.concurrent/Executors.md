# Executors

`Executor`和`Executors`又是相差一个字符s，想到之前也有三对同样的命名：`Array`和`Arrays`、`Collection`和`Collections`、`Object`和`Objects`（这三对在之前说了说了很多遍了，今天终于来了新成员）。

`Executor`是一个接口，实际上是一个执行器，而`Executors`是一个工具类，专门为执行器提供服务的。所以里面有很多的静态方法，结果如下图所示：

![Executors](http://ovn0i3kdg.bkt.clouddn.com/Executors.png)

这个类非常重要，因为它提供了生成线程池的一些常用方法。
