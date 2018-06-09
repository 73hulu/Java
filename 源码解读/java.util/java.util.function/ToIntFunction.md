# ToIntFunction/ToDoubleFunction/ToLongFunction

提供了对象到基本数据类型int、long和double的转换

## ToIntFunction
只有一个抽象方法，函数描述符是`T -> int`。
```java
@FunctionalInterface
public interface ToIntFunction<T> {

    /**
     * Applies this function to the given argument.
     *
     * @param value the function argument
     * @return the function result
     */
    int applyAsInt(T value);
}
```

## ToDoubleFunction
只有一个抽象方法，函数描述符是`T -> double`。
```java
@FunctionalInterface
public interface ToDoubleFunction<T> {

    /**
     * Applies this function to the given argument.
     *
     * @param value the function argument
     * @return the function result
     */
    double applyAsDouble(T value);
}
```

## ToLongFunction
只有一个抽象方法，函数描述符是`T -> long`
```java
@FunctionalInterface
public interface ToLongFunction<T> {

    /**
     * Applies this function to the given argument.
     *
     * @param value the function argument
     * @return the function result
     */
    long applyAsLong(T value);
}
```
