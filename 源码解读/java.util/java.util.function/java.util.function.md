# java.util.function

java8 开始Java支持函数式编程，在子包java.util.function中包含了很多内建的函数式接口，这些接口都增加了`@FunctionalInterface`注解，以便能使用在lambda上。下面列出了Java8中的内建函数式接口：

![java中内建函数式接口](https://ws1.sinaimg.cn/large/006tKfTcgy1fs4vn1ec7dj31260dzwhf.jpg)
![Java中内建函数式接口](https://ws3.sinaimg.cn/large/006tKfTcgy1fs4vns2f8vj31270dl778.jpg)

总的来说，Java8 中的函数式接口可以分为以下四种类型：

## 断言型接口——Predicate【T -> boolean】
获取判断的结果。基础接口为`Predicate`，接口抽象方法为`boolean test(T t);`，接受一个T对象，返回boolean，函数描述符为`T -> boolean`。

`Predicate`本身又提供了`and `、`or`和`negative`逻辑，利用这些逻辑，我们可以进行断言的链式操作。

`BiPredict`提供了两个参数的断言支持，接口抽象方法为`boolean test(T, U);`，接受两个对象，返回boolean，函数描述符为`(T, U) -> boolean`。同样提供了`and `、`or`和`negative`逻辑。

无论是`Predicate`还是`BiPredict`，都是对于类的操作，而对于基本数据类型来说，会存在自动装箱和拆箱的开销。为了避免这一开销，Java8专门提供了针对于基本数据类型的段言型接口。

与`Predicate`对应，Java8针对`int`、`long`和`double`提供了`IntPredicate`、`LongPredicate`和`DoublePredict`。**注意，并非所有的基本数据类型都有断言型接口**。

另外，Java8并没有开放针对双参数的基本数据类型的断言型接口。

`Predicate`用来作为判断条件，比较常见的使用常见是`filter`参数或`if`条件。

## 函数型接口——Function 【T -> R】
用来进行数据转换。基础接口为`Function`。接口抽象方法为`R apply(T t)`， 接受一个T类型对象，返回一个R类型对象，函数描述符为`T -> R`。

`Function`本身提供了一些接口方法，比如前置操作`compose`、后置操作`andThen`和本身操作`identity`。

`BiFunction`是两个参数版本的`Function`操作，抽象方法为`R apply(T t, U u)`，接受两个对象，返回一个R类型的对象，函数描述符为`(T, U) -> R`。另外，除了`apply`方法，它还剩下一个方法`andThen`，用来设定后置操作。

Java8同样了对于基本数据类型的支持。对应于`Function`，Java8 提供了对于`int`、`long`和`double`数据类型的支持和它们三者和对象之间的转换操作，各自的关系如下：

![Function的扩展](https://ws1.sinaimg.cn/large/006tKfTcgy1fs4xa69gwfj30fb0aft9n.jpg)

基于`BiFunction`，Java只提供了object到三种基本数据类型(int | long | double)之间的函数型接口，分别对应`ToIntBiFunction`、`ToLongBiFunction`和`ToDoubleBiFunction`。


特别的，对于同一类型的数据转换，Java8中有专门定制的两个类：`UnaryOperator`和`BinaryOperator`，前者的函数描述符为`T -> T`，后者的函数描述符为`(T, T) -> T`。后者提供了`minBy`和`maxBy`方法，用来比较数字或字符串的大小。


## 消费型接口——Consumer【T -> ()】
用来进行数据消费。基础接口为`Consumer`。接口抽象方法为`void accept(T t)`，接受一个T类型对象，无返回，函数描述符为`T -> ()`。

`Consumer`本身还提供了 `andThen`方法，用来定义后置操作。

`BiConsumer`是双参数版本的`Consumer`，函数描述符为`(T t, U u) -> void`。

基于`Consumer`，Java8提供了对于`int`、`long`和`double`三种类型的数据消费的支持，分别对应`IntConsumer`、`LongConsumer`和`DoubleConsumer`。

基于`BiConsumer`，Java8提供了对于`int`、`long`和`double`三种类型的数据消费的支持，分别对应`ObjIntConsumer`、`ObjLongConsumer`和`ObjDoubleConsumer`。这三个类中的apply都接受一个对象和一个基本数据类型参数，无返回值。

## 供应型接口——Supplier【() -> T】
用于产生所需。基础接口为`Supplier`，接口抽象方法为`T get()`，无参数，返回T类型独享，函数描述符为`() -> T`。

`Supplier`接口本身无其他方法。

Java8提供了对于基本数据类型`boolean`、`int`、`long`和`double`的支持，分别对应类`BooleanSupplier`、`IntSupplier`、`LongSupplier`和`DoubleSupplier`。
