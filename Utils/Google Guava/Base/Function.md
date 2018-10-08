# Function

JDK8开始，[`java.util.function`](/源码解读/java.util/java.util.function/java.util,function.md)中提供了函数式编程的实现。其中[`Function`]((/源码解读/java.util/java.util.function/Function.md))的函数描述符是：`F -> T`，即将一个类型转化为另一个类型实例。

`Google Guava`中`Function`是对JDK8中`Function`的扩充。

## public interface Function<F, T> extends java.util.function.Function<F, T>{..}
接口声明，继承了JDK8中的`Function`接口。

## T apply(@Nullable F input);
对于JDK8中`Function`的重写（上面有注解`@Override`）。


## boolean equals(@Nullable Object object);
用来判等。是父类方法的重写。父类是接口`Function`，这里涉及到一个问题，接口到底有没有继承`Object`，答案是没有，否则就是类的多继承，不符合Java语言的特点。但是接口自有一套与`Object`类似的继承规则，使得接口包含了与继承`Object`类似的效果。
