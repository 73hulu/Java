# Consumer

函数式接口。唯一的接口是`void accept(T t)`，所以`Consumer`的函数描述符是`T -> void`。主要用来进行遍历处理。

具体的结果是：

![Consumer](https://ws2.sinaimg.cn/large/006tNc79gy1fs1ubfaxusj30im03amxh.jpg)

## void accept(T t);
抽象方法。主要用来进行遍历处理。

## default Consumer<T> andThen(Consumer<? super T> after) {..}
链式处理。
```Java
default Consumer<T> andThen(Consumer<? super T> after) {
    Objects.requireNonNull(after);
    return (T t) -> { accept(t); after.accept(t); };
}
```
