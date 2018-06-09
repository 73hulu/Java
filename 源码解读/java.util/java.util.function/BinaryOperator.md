# BinaryOperator
`BinaryOperator`继承自`BiFunction`，是二元运算，接受两个操作数并返回一个相同类型的对象，定义如下：
```java
@FunctionalInterface
public interface BinaryOperator<T> extends BiFunction<T,T,T> {
    /**
     * Returns a {@link BinaryOperator} which returns the lesser of two elements
     * according to the specified {@code Comparator}.
     *
     * @param <T> the type of the input arguments of the comparator
     * @param comparator a {@code Comparator} for comparing the two values
     * @return a {@code BinaryOperator} which returns the lesser of its operands,
     *         according to the supplied {@code Comparator}
     * @throws NullPointerException if the argument is null
     */
    public static <T> BinaryOperator<T> minBy(Comparator<? super T> comparator) {
        Objects.requireNonNull(comparator);
        return (a, b) -> comparator.compare(a, b) <= 0 ? a : b;
    }

    /**
     * Returns a {@link BinaryOperator} which returns the greater of two elements
     * according to the specified {@code Comparator}.
     *
     * @param <T> the type of the input arguments of the comparator
     * @param comparator a {@code Comparator} for comparing the two values
     * @return a {@code BinaryOperator} which returns the greater of its operands,
     *         according to the supplied {@code Comparator}
     * @throws NullPointerException if the argument is null
     */
    public static <T> BinaryOperator<T> maxBy(Comparator<? super T> comparator) {
        Objects.requireNonNull(comparator);
        return (a, b) -> comparator.compare(a, b) >= 0 ? a : b;
    }
}
```
可以看到，该接口中定义了`minBy`和`maxBy`方法，用来比较数字或字符串的大小。注意到，这是一个静态方法，接受一个`Comparator`接口对象，返回的是`BinaryOperator`接口对象。

例如：
```Java
// maxBy & minBy
BinaryOperator<String> bi = BinaryOperator.maxBy(Comparator.naturalOrder());
System.out.println(bi.apply("a", "b")); // b
```

下面是一个使用`BinaryOperator`的例子：

```Java
public class BinaryOperatorTest {
    public static void main(String[] args) {
        Map<String, String> map = ImmutableMap.of("A", "a", "B", "b", "C", "c");
        BinaryOperator<String> operator = (s1, s2) -> s1 + "-" + s2;
        binaryOperationFun(operator, map).forEach(System.out::println);

    }

    private static List<String> binaryOperationFun(BinaryOperator<String> binaryOperator, Map<String, String> map){
        List<String> biList = new ArrayList<>();
        map.forEach((s1, s2) -> biList.add(binaryOperator.apply(s1, s2)));
        return biList;
    }
}
```
