# SynchronousBlockingQueue


特别之处在于它内部没有容器，一个生产线程，当它生产产品（即put的时候），如果当前没有人想要消费产品(即当前没有线程执行take)，此生产线程必须阻塞，等待一个消费线程调用take操作，take操作将会唤醒该生产线程，同时消费线程会获取生产线程的产品（即数据传递），这样的一个过程称为一次配对过程(当然也可以先take后put,原理是一样的)。
```Java
/**
 * Adds the specified element to this queue, waiting if necessary for
 * another thread to receive it.
 *
 * @throws InterruptedException {@inheritDoc}
 * @throws NullPointerException {@inheritDoc}
 */
public void put(E e) throws InterruptedException {
    if (e == null) throw new NullPointerException();
    if (transferer.transfer(e, false, 0) == null) {
        Thread.interrupted();
        throw new InterruptedException();
    }
}

/**
* Retrieves and removes the head of this queue, waiting if necessary
* for another thread to insert it.
*
* @return the head of this queue
* @throws InterruptedException {@inheritDoc}
*/
public E take() throws InterruptedException {
   E e = transferer.transfer(null, false, 0);
   if (e != null)
       return e;
   Thread.interrupted();
   throw new InterruptedException();
}
```
