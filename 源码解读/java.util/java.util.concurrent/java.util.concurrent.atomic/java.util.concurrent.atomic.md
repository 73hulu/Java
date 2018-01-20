# java.util.concurrent.atomic

这个包是JUC的子包，里面提供了一组原子变量类。

其基本的也正是在多线程的环境下，当有多个线程同时执行这些类的实例包含的方法时候，具有排他性，即当某个线程进入方法，执行其中的指令时，不会被其他线程打断，而别的线程就行自旋锁一样，一直等到该方法执行完成，才有JVM等待队列中选择一个另一个线程进入。这是一种逻辑上的理解。实际上是借助硬件的相关指令来实现的，不会阻塞线程（或者说只是在硬件级别上阻塞了）。可以对基本数据、数组中的基本数据、对类中的基本数据进行操作。原子变量类相当于一种泛化的`volatile`变量，能够支持原子的和有条件的读-该-写操作。

这个包的组成结构如下：

![atomic](http://ovn0i3kdg.bkt.clouddn.com/java.util.concurrent.atomic.png)

大致可以分为四组：

| 分组 | 意义 |包含的类 |
| :------------- | :------------- |
| 标量类(Scalar)      |   用来处理布尔、整数、长整数、对象四种数据，其内部实现不是简单的使用`synchronized`，而是使用CAS + volatile和native方法，从而避免了`synchronized`的高开销，执行效率大大为提升   |`AtomicBoolean`、 `AtomicInteger`、`AtomicLong`和`AtomicReference`|
|数组类   | 进一步扩展了原子操作，对数组提供了支持，它们内部并不是向`AtomicInteger`一样维持一个`valatile`变量，而是全部由native方法实现  | `AtomicIntegerArray`、`AtomicLongArray`、`AtomicReferenceArray` |
|更新器类   | 基于反射，对指令类的volatile字段进行原子更新  | `AtomicLongFieldUpdater` 、`AtomicIntegerFieldUpdater` 、`AtomicReferenceFieldUpdater`|
|复合标量类   |  防止ABA问题出现而构造的类。如什么是ABA问题呢，当某些流程在处理过程中是顺向的，也就是不允许重复处理的情况下，在某些情况下导致一个数据由A变成B，再中间可能经过0-N个环节后变成了A，此时A不允许再变成B了，因为此时的状态已经发生了改变，他们都是对atomicReference的进一步包装，AtomicMarkableReference和AtomicStampedReference功能差不多，有点区别的是：它描述更加简单的是与否的关系，通常ABA问题只有两种状态，而AtomicStampedReference是多种状态，那么为什么还要有AtomicMarkableReference呢，因为它在处理是与否上面更加具有可读性。 | `AtomicMarkableReference` 、`AtomicStampedReference` |


参考
* [java.util.concurrent包详细分析](http://blog.csdn.net/windsunmoon/article/details/36903901s)
