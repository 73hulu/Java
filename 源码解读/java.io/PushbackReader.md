# PushbackReader

"push back"是回退的意思，所以`PushbackReader`就是回退流。这是一个将字符退回到输入流中的字符输入流。什么叫做退回？

在JAVA IO中所有的数据都是采用顺序的读取方式，即对于一个输入流来讲都是采用从头到尾的顺序读取的，如果在输入流中某个不需要的内容被读取进来，则只能通过程序将这些不需要的内容处理掉，为了解决这样的处理问题，在JAVA中提供了一种回退输入流，可以把读取进来的某些数据重新回退到输入流的缓冲区之中。下面这幅图解释了回退流的工作原理


![回退流的工作原理](http://ovn0i3kdg.bkt.clouddn.com/%E5%9B%9E%E9%80%80%E6%B5%81%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86.png)

> 回退流和一般流中reset方法有区别？


结构如下：

![PushbackReader](http://ovn0i3kdg.bkt.clouddn.com/PushbackReader.png)
参考
* [跟我学IO（PushbackReader类）](http://www.bug315.com/article/225.htm)
* [Java IO操作——回退流PushbackInputStream](http://blog.csdn.net/u013087513/article/details/52171078)
