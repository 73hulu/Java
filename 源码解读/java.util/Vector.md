# Vector

`Vector`是一个历史遗留类，现在一般不怎么用。其结构如下：

![Vector](http://ovn0i3kdg.bkt.clouddn.com/Vector_1.png)
![Vector](http://ovn0i3kdg.bkt.clouddn.com/Vector_2.png)

由于现在一般都不怎么使用了，所以源码解读在此略过。可以这么理解， `Vector`是线程安全的`ArrayList`，但是在设计上没有`LinkedList`细腻。所以我们一般都不再使用`Vector`，如果要实现线程安全的`ArrayList`，可以借助于`collections.synchronizedList`方法来创建线程安全的`ArrayList`。
