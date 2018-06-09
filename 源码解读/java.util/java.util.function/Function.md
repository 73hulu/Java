# Function
`Function`代表的是接口一个参数，经过处理后产生另外一个类型结果的函数，函数描述符是`T -> R`， 其结构如下：

![Function](https://ws1.sinaimg.cn/large/006tNc79gy1fs2zu6xstxj30ja05sgmd.jpg)

##  R apply(T t);
`Function`的关键函数，函数签名是`T -> R`。

## default <V> Function<V, R> compose(Function<? super V, ? extends T> before){..}
在apply之前需要执行的操作。这使得函数签名变成`V -> T -> R`。
```Java
default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
    Objects.requireNonNull(before);
    return (V v) -> apply(before.apply(v));
}
```

##  default <V> Function<T, V> andThen(Function<? super R, ? extends V> after){..}
```Java
default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
    Objects.requireNonNull(after);
    return (T t) -> after.apply(apply(t));
}
```
与`compose`相反，`andThen`约定的是apply之后的操作。


## static <T> Function<T, T> identity(){..}
这个就比较奇怪了，返回的永远都是自己本身。
```Java
static <T> Function<T, T> identity() {
    return t -> t;
}
```
