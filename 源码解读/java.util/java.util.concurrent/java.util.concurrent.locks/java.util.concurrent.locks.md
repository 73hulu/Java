# java.util.concurrent.locks

这个包中的接口和类提供了一个框架，用于锁定和等待与内置同步和监视器不同的条件。该框架允许在使用`locks`和`Condition`方面具有更大的灵活性的语法。

主要的接口

| 接口 | 功能 |
| :------------- | :------------- |
| Lock | 锁，比synchronized提供更广泛的线程操作 |
| Condition    | 进程间通信  |
| ReadWriteLock  |维护一对关联的锁，一个用来读，一个用来写   |


主要的类：

| 类名 | 功能 |
| :------------- | :------------- |
| AbstractOwnableSynchronizer      | Item Two       |
|AbstractQueuedLongSynchronizer   |   |
|AbstractQueuedSynchronizer   |   |
|LockSupport   |   |
|ReentrantLock	|   |
|ReentrantReadWriteLock   |   |
|ReentrantReadWriteLock.ReadLock   |   |
|ReentrantReadWriteLock.WriteLock	   |   |
|StampedLock	   |   |

`Lock`接口支持在语义（可重入，公平等）方面不同的锁定规则，并且可以在非块结构的上下文中使用，包括手工和锁重新排序算法。主要实现是`ReentrantLock`。

`ReadWriteLock`接口类似地定义了可能在读者之间共享的锁，但是对作者是排他性的。只提供了一个实现`ReentrantReadWriteLock`，因为它涵盖了大多数标准的使用上下文。但程序员可能会创建自己的实现来覆盖非标准要求。

`Condition`接口描述可能与锁相关联的条件变量。这些使用方式与使用`Object.wait`访问的隐式监视器相似，但提供扩展功能。特别地，多个条件对象可以与单个锁相关联。为了避免兼容性问题，条件方法的名称与对应的对象版本不同。

`AbstractQueuedSynchronizer`类作为一个有用的超类，用于定义依赖排队阻塞线程的锁和其他同步器。 `AbstractQueuedLongSynchronizer`类提供相同的功能，但扩展了对64位同步状态的支持。两者都扩展了`AbstractOwnableSynchronizer`类，一个简单的类，可以帮助记录当前持有排他同步的线程。

`LockSupport`类提供了较低级别的阻止和解除阻塞支持，对于那些开发自己的定制锁类的开发人员来说，这是非常有用的


> Java 8 API https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/locks/package-summary.html
