# ObjIntConsumer/ObjLongConsumer/object

基于`BiConsumer`，提供对于`int`、`long`和`double`三种基本数据类型的支持。以`ObjIntConsumer`为例：

```java
@FunctionalInterface
public interface ObjDoubleConsumer<T> {

    /**
     * Performs this operation on the given arguments.
     *
     * @param t the first input argument
     * @param value the second input argument
     */
    void accept(T t, double value);
}
```
`accept`接受两个参数： T类型对象和double类型数据，无返回值。函数描述符为`(T, double) -> void`。
