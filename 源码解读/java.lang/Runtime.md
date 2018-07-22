# Runtime

`Runtime`类代表着Java程序的运行时环境，每个Java程序都有一个Runtime实例，该类会自动被创建，可以通过`Runtime.getRuntime()`方法来获取当前程序的`Runtime`实例。

> Runtime是单例模式的应用。


![Runtime](https://ws4.sinaimg.cn/large/006tNc79gy1ftiovgmywpj30i60vudl3.jpg)


其中一个比较重要的方法是`exec`，这个方法用来执行外部命令。

比如Linux下调用系统命令
```Java
String [] cmd={"/bin/sh","-c","ln -s exe1 exe2"};
Process proc =Runtime.getRuntime().exec(cmd);
```
