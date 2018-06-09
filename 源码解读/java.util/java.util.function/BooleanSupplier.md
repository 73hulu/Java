# BooleanSupplier/IntSupplier/LongSupplier/DoubleSupplier

`Supplier`的函数函数描述符是`() -> T`，对于基本基本数据类型来说，为了避免拆箱装箱的麻烦，提供了对于基本数据类型`boolean`、`int`、`long`和`double`的支持。所以才有了以下四个类。

## BooleanSupplier
函数描述符是`() -> boolean`。
```Java
@FunctionalInterface
public interface BooleanSupplier {

    /**
     * Gets a result.
     *
     * @return a result
     */
    boolean getAsBoolean();
}
```

## IntSupplier
函数描述符是`() -> int`。
```java
@FunctionalInterface
public interface IntSupplier {

   /**
    * Gets a result.
    *
    * @return a result
    */
   int getAsInt();
}
```

## LongSupplier
函数描述符是`() -> long`。
```Java
@FunctionalInterface
public interface LongSupplier {

    /**
     * Gets a result.
     *
     * @return a result
     */
    long getAsLong();
}
```

## DoubleSupplier
函数描述符是`() -> double`。
```Java
@FunctionalInterface
public interface DoubleSupplier {

    /**
     * Gets a result.
     *
     * @return a result
     */
    double getAsDouble();
}
```
