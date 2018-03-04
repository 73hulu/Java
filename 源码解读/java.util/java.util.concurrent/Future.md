# Future
`Future`可以拿到异步执行任务的结果。它定义了计算是否进行，取消计算进行，等待计算完成，取得执行结果。

这是一个接口，定义如下：

![Future](http://ovn0i3kdg.bkt.clouddn.com/Future.png)


##  boolean cancel(boolean mayInterruptIfRunning);
该方法用来尝试取消任务过程。如果任务已经完成，该尝试将失败。如果成功，并任务根本没开始之前就被调用了，那么任务将永远不被执行。如果任务已经开始了，但是没有结束，那么`mayInterruptIfRunning`参数确定执行这个任务的线程是否应该被中断以试图停止任务。

在此方法返回后，对`isDone（）`的后续调用将始终返回true。 如果此方法返回`true`，则后续调用`isCancelled（）`将始终返回`true`。

## boolean isCancelled();
查看任务是否已经被取消。

## boolean isDone();
查看任务是否已经结束。这里的“结束”并不代表执行结束，它可能是执行结束，还有可能是被抛出了异常，还有可能是被取消了。符合以上情况就会返回true。

## V get() throws InterruptedException, ExecutionException;
取得任务的结果，注意，返回值只有当任务成功执行结束后才能正确返回，如果任务正在被执行中，调用该方法的线程将被阻塞，直到任务结束。
如果当前线程在等待的时候被中断，那么该方法将抛出`InterruptedException`。如果任务执行的线程抛出了异常，那么该方法将抛出`ExecutionException`。如果任务取消了，那么该方法将抛出`CancellationException`。V get(long timeout, TimeUnit unit)

## V get(long timeout, TimeUnit unit) throws InterruptedException, ExecutionException, TimeoutException;
`get`方法的重载，设定了等待的时间。如果在等待的时间内还得不到任务结果，那么方法将抛出`TimeoutException`。


`Future`的一个实现类是`FutureTask`。
