# Modifier

之前在`Field`和`Method`中遇到`getModifiers`类的时候很困惑，为什么这个方法的返回值类型是int，看这个类的实现就知道了。类的结构如下：

![Modifier](http://ovn0i3kdg.bkt.clouddn.com/Modifier.png)

这个类将所有的修饰符编号，这个编号就是int型，这也就是为什么`getModifiers`方法的返回值类型是int。

类中的方法很简单，就是在判断是不是具有某个修饰符，以及类、接口、构造函数、属性、方法各自都具有什么样的修饰符？

例如一个方法有多个修饰符，那么这个方法调用`getModifiers`的值是多少，做个试验。

算了不试验了，我要回家了~~~
