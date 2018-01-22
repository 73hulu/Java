# Executor
`java.lang`包为我们提供了线程操作的两个——`Runnable`和`Thread`，前者用来定义一个线程的工作单元，后者即为线程实现。`Thread`和`Object`都提供了线程状态转化的一些方法，所以我实现多线程程序的时，我们不仅需要手动创建线程，还需要手动进行线程的状态管理。

JDK1.5开始，JUC提供了`Executor`框架，线程池代替了原来线程的手动创建，而线程的状态转化无需我们过问。

`Executor`是最基本的接口。其结构如下

![Executor](http://ovn0i3kdg.bkt.clouddn.com/Executor.png)

这也是一个功能性接口，只有一个方法`executor`。实际上，`Executor`的作用完全是为了代替手动创建`Thread`。例如之前`new Thread(new (RunableTask())).start()`完全可以用下面的方法代替：
```java
Executor executor = (anExecutor);
executor.execute(new RunableTask());
executor.execute(new RunableTask());
```
其中`anExecutor`当然是一个`Executor`的实现类的一个实例。而`execute`方法接收一个`Runnable`类型对象，这个方法的实现可能是：
```java
class DirectExecutor implements Executor{
  @Override
  public void execute(Runnable runnable){
    runnable.run();
  }
}
```
更常用的一种方法是：
```java
class DirectExecutor implements Executor{
  @Override
  public void execute(Runnable runnable){
    new Thread(runnable).start();
  }
}
```
下面这段代码是源码中给出的使用示例：
```java
class SerialExecutor implements Executor {
   final Queue<Runnable> tasks = new ArrayDeque<Runnable>();
   final Executor executor;
   Runnable active;

   SerialExecutor(Executor executor) {
     this.executor = executor;
   }

   public synchronized void execute(final Runnable r) {
     tasks.offer(new Runnable() {
       public void run() {
         try {
           r.run();
         } finally {
           scheduleNext();
         }
       }
     });
     if (active == null) {
       scheduleNext();
     }
   }

   protected synchronized void scheduleNext() {
     if ((active = tasks.poll()) != null) {
       executor.execute(active);
     }
   }
 }
```
`SerialExecutor`是一个执行器，它用一个队列保存工作单元，并且按照顺序执行。
