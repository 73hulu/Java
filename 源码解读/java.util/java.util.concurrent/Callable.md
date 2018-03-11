# Callable

`Callable`和`Runnable`一样，都表示一项任务，都能够让线程执行，从名字就能看出来。与`Runnable`一样，`Callable`是一个接口，并且是一个“功能接口”，里面之定义了一个方法`call`，接口结构如下：

![Callable](http://ovn0i3kdg.bkt.clouddn.com/Callable.png)

整个接口定义如下：

```java
@FunctionalInterface
public interface Callable<V> {
    /**
     * Computes a result, or throws an exception if unable to do so.
     *
     * @return computed result
     * @throws Exception if unable to compute a result
     */
    V call() throws Exception;
}
```

很容易就能看出`Runnable`和 `Callable`的区别：
1. `Runnable`在java.lang包中，`Callable`在包`java.util.concurrent`中，是1.5之后添加的，为了区分线程的工作单元和执行机制。在`Executor`框架中，`Runnable`和`Callable`都是线程的工作单元。
2. `Runnable`中定义的`run`方法返回值是`void`，并且不能抛出可检异常，而`Callable`中定义的`call`方法具有返回值，并且会抛出可检异常。


`Callable`产生结果，这个结果是异步的（因为线程的调度无法预估），这个结果可以被`Future`接口的实例拿到。
