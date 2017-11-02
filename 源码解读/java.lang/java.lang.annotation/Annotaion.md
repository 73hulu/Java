# Annotaion

Annotation是一个接口，定义如下：

![Annotation](http://ovn0i3kdg.bkt.clouddn.com/Annotation.png)

`Annotation`是所有注解类型扩展的通用接口。 请注意，手动扩展此接口的接口不会定义注释类型。 还要注意，这个接口本身并没有定义一个注释类型。

这里面需要注意的是`annotationType`方法，它将返回此注释的注释类型。定义如下：
```java
Class<? extends Annotation> annotationType();
```
