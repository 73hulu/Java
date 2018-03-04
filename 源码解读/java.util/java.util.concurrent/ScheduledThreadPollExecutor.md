# ScheduledThreadPollExecutor
`ScheduledThreadPollExecutor`继承自`ThreadPoolExecutor`，并实现了`ScheduledExecutorService`接口。

从名字就能看出来，这个类与定时相关。

定时任务类,每个调度任务都会分配到线程池中的一个线程去执行,也就是说,任务是并发执行,互不影响。

需要注意,只有当调度任务来的时候，`ScheduledExecutorService`才会真正启动一个线程,其余时间`ScheduledExecutorService`都是出于轮询任务的状态

类的结构如下：

![ScheduledThreadPollExecutor](http://ovn0i3kdg.bkt.clouddn.com/ScheduledThreadPoolExecutor.png)
