# Runnable

`Runnable`可以理解为可执行的任务，用这个任务来定义线程，即线程应该完成哪项任务。


![Runnable](http://ovn0i3kdg.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-03-03%20%E4%B8%8B%E5%8D%8811.45.10.png)

只有一个方法，所以这是一个函数式接口。

```Java
@FunctionalInterface
public interface Runnable {
    /**
     * When an object implementing interface <code>Runnable</code> is used
     * to create a thread, starting the thread causes the object's
     * <code>run</code> method to be called in that separately executing
     * thread.
     * <p>
     * The general contract of the method <code>run</code> is that it may
     * take any action whatsoever.
     *
     * @see     java.lang.Thread#run()
     */
    public abstract void run();
}

```
继承该接口的类定义了应该完成的任务，然后用这个任务去实例化线程。这是一个创建新城的方法。
