# UnaryOperator

`UnaryOperator`继承自`Function`，接受一个操作数并返回一个相同类型的对象，类定义如下：
```Java
@FunctionalInterface
public interface UnaryOperator<T> extends Function<T, T> {

    /**
     * Returns a unary operator that always returns its input argument.
     *
     * @param <T> the type of the input and output of the operator
     * @return a unary operator that always returns its input argument
     */
    static <T> UnaryOperator<T> identity() {
        return t -> t;
    }
}
```
`UnaryOperator`是一元运算，与其对应的二元运算是`BiOperator`。

下面是一个应用`UnaryOperator`运算的例子：
```Java
public class UnaryOperatorTest {
    public static void main(String[] args) {
        List<Integer> list = Arrays.asList(10, 20, 30, 40, 50);
        UnaryOperator<Integer> operator =  i -> i * i;
        unaryOperatorFun(operator, list).forEach(System.out::println);

    }

    private static List<Integer> unaryOperatorFun(UnaryOperator<Integer> unaryOperator, List<Integer> list){
        return list.stream().map(i -> unaryOperator.apply(i)).collect(Collectors.toList());
    }
}
```
