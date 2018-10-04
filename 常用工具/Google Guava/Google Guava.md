# Google Guava

当我还在一家机器人公司实习的时候，一个同事告诉我，作为应届生如果好好看过三类源码，那么国内大小公司的offer都不在话下。这三类源码分别是：JDK源码、Google Guava和Linux内核。我一直记得这句话，并尽力按照这样的顺序进行源码学习。之前已经完成了JDK8的重要源码源码的学习（需要反复研读体会设计思想），深觉源码阅读对技术人员形成完整代码结构和良好设计思维的重要性。而Google Guava是对JDK的补充，弥补了JDK源码的不足，同时提炼出一些经典的设计思想（比如不可修改类），帮助技术人员写出更加简洁、安全、优雅的代码。

Google Guava是什么东西？首先要追溯到2007年的“Google Collections Library”项目，它提供对Java 集合操作的工具类。后来Guava被进化为Java程序员开发必备的工具。Guava可以对字符串，集合，并发，I/O，反射进行操作。

>" 这个库简化了你的代码，使它易写、易读、易于维护。它能提高你的工作效率，让你从大量重复的底层代码中脱身。"

> * [源码](https://github.com/google/guava)
> * [Java DOC](https://google.github.io/guava/releases/snapshot-jre/api/docs/)
> * [Google Guava官方教程 (英文)](https://github.com/google/guava/wiki) ：已经做好总结，强烈推荐按照这个顺序查看源码
> * [Google Guava官方教程（中文版）](http://ifeve.com/google-guava/)




下载源码后查看目录，如下：

![Guava目录](https://ws4.sinaimg.cn/large/006tNc79ly1fvt3wd6gsmj30em0saaae.jpg)

其中比较重要的有以下几种：

| 类型 | 功能 |
| :------------- | :------------- |
| base | 基本工具，让Java的使用更加舒适 |
|collect   | 对JDK集合的扩展，非常重要  |
|cache   |  本地缓存的实现 |
|concurrent   | 并发实现  |
|primitives   |原生类型   |
|io   |  简化I/O尤其是I/O流和文件的操作 |
|hash   |  提供比Object.hashCode()更复杂的散列实现，并提供布鲁姆过滤器的实现 |
|eventbus   |  发布-订阅模式的组件通信，但组件不需要显式地注册到其他组件中|
|reflect   |  Java反射机制 |
|math   |   数学运算|

本章将根据以上分类进行重要源码的解读。
