# Command

命令模式
> 将一个请求**封装**为一个对象，从而使你可以用**不同的请求**对客户进行**参数化**；
对请求排队或请求日志，以及支持可撤销的操作

不少Command模式的代码都是针对图形界面的，它实际就是菜单命令，我们在一个下拉菜单选择一个命令时，然后会执行一些动作。将这些命令封装成在一个类中，然后用户(调用者)再对这个类进行操作，这就是Command模式，换句话说，本来用户(调用者)是直接调用这些命令的，如菜单上打开文档(调用者)，就直接指向打开文档的代码，使用Command模式，就是在这两者之间增加一个中间者，将这种直接关系拗断，同时两者之间都隔离,基本没有关系了。

显然这样做的好处是符合封装的特性，降低耦合度，Command是将对行为进行封装的典型模式，Factory是将创建进行封装的模式。

![命令模式](http://img.blog.csdn.net/20170621121426157?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2p5dHRrbA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



参考
* [java设计模式之命令模式](https://www.cnblogs.com/liaoweipeng/p/5693154.html)
