# ExecutorService

`Executor`接口定义了`execute`方法来启动线程，`ExecutorService`接口继承了该`Executor`，并定义了一些线程的管理操作。其接口结构如下：

![ExecutorService](http://ovn0i3kdg.bkt.clouddn.com/ExecutorService.png)


API如下：

| 方法 | 说明 |
| :------------- | :------------- |
| void shutdown(); | 这个方法会将之前提交的任务执行有序关闭，且不会等待这些任务是否已经完成，并且不会再接受新任务    |
|List<Runnable> shutdownNow()   |  尝试停止所有正在执行的任务，暂停等待任务的处理，并返回正在等待执行的任务列表。此方法不会等待主动执行的任务终止。 使用awaitTermination来做到这一点。 |
|boolean isShutdown();   |判断executor是否已经关闭   |
|boolean isTerminated();   | 如果所有任务在关闭后都完成，则返回true。 请注意，除非shutdown或shutdownNow先被调用，否则isTerminated从来就不是真的。  |
| boolean awaitTermination(long timeout, TimeUnit unit) throws InterruptedException；   |   阻塞，直到所有任务在关闭请求后发生或发生超时，或者当前线程中断，无论哪个先发生。|
|<T> Future<T> submit(Callable<T> task);   | 提交Callable类型返回值任务，并返回表示未完成任务结果的Future。 未来的get方法将在成功完成时返回任务的结果。 如果您想立即阻止等待任务，可以使用结构result = exec.submit（aCallable）.get（）; |
|<T> Future<T> submit(Runnable task, T result);   |  提交Runnable类型任务并返回表示该任务的Future。 未来的get方法将在成功完成时返回给定的结果。 |
|Future<?> submit(Runnable task);   |   提交Runnable类型任务并返回表示该任务的Future。 Future的get方法在成功完成后将返回null。|
|<T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks) throws InterruptedException; | 执行给定的任务，返回一份持有其状态和结果的期货清单，当全部完成时。 Future.isDone（）对于返回列表的每个元素都是true。 请注意，完成的任务可能正常结束或通过抛出异常终止。 如果在进行此操作时修改了给定的集合，则此方法的结果未定义。  |
|<T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks,long timeout, TimeUnit unit) throws InterruptedException;   | 比上一个方法多了一个时间限定   |
|<T> T invokeAny(Collection<? extends Callable<T>> tasks) throws InterruptedException, ExecutionException;   | 执行给定的任务，当所有完成或超时到期时返回一份持有其状态和结果的期货清单，以先发生者为准。 Future.isDone（）对于返回列表的每个元素都是true。 返回后，尚未完成的任务将被取消。 请注意，完成的任务可能正常结束或通过抛出异常终止。 如果在进行此操作时修改了给定的集合，则此方法的结果未定义。  |


下面是是源码中的示例：
```Java
class NetworkService implements Runnable {
   private final ServerSocket serverSocket;
   private final ExecutorService pool;

   public NetworkService(int port, int poolSize)
       throws IOException {
     serverSocket = new ServerSocket(port);
     pool = Executors.newFixedThreadPool(poolSize);
   }

   public void run() { // run the service
     try {
       for (;;) {
         pool.execute(new Handler(serverSocket.accept()));
       }
     } catch (IOException ex) {
       pool.shutdown();
     }
   }
 }

 class Handler implements Runnable {
   private final Socket socket;
   Handler(Socket socket) { this.socket = socket; }
   public void run() {
     // read and service request on socket
   }
 }

```


`ExecutorService`有两个具体常用的实现类：`ThreadPoolExecutor`和`ScheduledThreadPollExecutor`。也就是我们常说的线程池。




参考
* [ExecutorService引发的血案（二）ExecutorService使用](http://blog.csdn.net/u010940300/article/details/50251841)
