# IntToLongFunction/IntToDoubleFunction/LongToDoubleFunction/LongToIntFunction

`Function`系中还提供了基本类型int、long和double之间的转换，int和long之间可以相互住转换，两者都能向double转换，所以才有了以下四类：

## IntToLongFunction
其中只有一个抽象方法，函数描述符是`int -> long`。
```java
@FunctionalInterface
public interface IntToLongFunction {

    /**
     * Applies this function to the given argument.
     *
     * @param value the function argument
     * @return the function result
     */
    long applyAsLong(int value);
}
```

## IntToDoubleFunction
其中只有一个抽象方法，函数描述符是`int -> double`。
```java
@FunctionalInterface
public interface IntToDoubleFunction {

    /**
     * Applies this function to the given argument.
     *
     * @param value the function argument
     * @return the function result
     */
    double applyAsDouble(int value);
}
```

## LongToIntFunction
其中只有一个抽象方法，函数描述符是`long -> int`。
```Java
@FunctionalInterface
public interface LongToIntFunction {

    /**
     * Applies this function to the given argument.
     *
     * @param value the function argument
     * @return the function result
     */
    int applyAsInt(long value);
}
```

## LongToDoubleFunction
其中只有一个抽象方法，函数描述符是`long -> double`。
```Java
@FunctionalInterface
public interface LongToDoubleFunction {

    /**
     * Applies this function to the given argument.
     *
     * @param value the function argument
     * @return the function result
     */
    double applyAsDouble(long value);
}
```
