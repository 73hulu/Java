# Stream

这个真的是Java8中最重要的一个设计了——流计算。`Stream`是流式计算中最基本的接口，结构如下：

![Stream](https://ws1.sinaimg.cn/large/006tKfTcgy1fs5auhg9naj30pu1787dy.jpg?imageView/2/w/400)

注意看这些方法的返回值，有一些返回`Stream`对象的方法我们可以构成链式计算。

##  public interface Stream<T> extends BaseStream<T, Stream<T>>
类声明，其中`BaseStream`的接口声明如下：
```Java
public interface BaseStream<T, S extends BaseStream<T, S>>
        extends AutoCloseable
```
可以看到，该接口继承了`AutoCloseable`接口，说明流可以自动关闭。话是这么说，但是事实上，所有的流都不需要关闭，通常，只有源为IO通道的流（如文件返回行（路径、字符集））将需要关闭，大多数流都由集合、数组或生成函数支持，这些函数不需要特殊的资源管理。（如果流确实需要关闭，则可以在try-with-resource语句中声明为资源）。

## 筛选操作
```Java
Stream<T> filter(Predicate<? super T> predicate); // 谓词筛选

Stream<T> distinct(); //去重

Stream<T> limit(long maxSize); // 截断，具有短路性， 原理是构建了一个指定大小的流，而不用对整个流做处理，这种也行能将无限流整合成有限流

Stream<T> skip(long n); // 跳过，与limit互补，如果元素不够跳过的个数，则返回空流
```
其中`distinct + toList`操作可以使用`toSet`操作来代替。

必要的`distinct`操作可以减轻后序流操作的开销。

## 映射操作
切片可以理解为投影操作，即从众多的属性中摘取其中的一些特性。
```Java
<R> Stream<R> map(Function<? super T, ? extends R> mapper); // 对流中的每个元素应用函数

IntStream mapToInt(ToIntFunction<? super T> mapper);

LongStream mapToLong(ToLongFunction<? super T> mapper);

DoubleStream mapToDouble(ToDoubleFunction<? super T> mapper);

<R> Stream<R> flatMap(Function<? super T, ? extends Stream<? extends R>> mapper); // 将各个流扁平化成单个流

IntStream flatMapToInt(Function<? super T, ? extends IntStream> mapper);

LongStream flatMapToLong(Function<? super T, ? extends LongStream> mapper);

DoubleStream flatMapToDouble(Function<? super T, ? extends DoubleStream> mapper);
```
其中`flatMap`的作用是将各个流扁平化成单个流，比如要提取给定单词列表["Hello","World"]，返回列表["H","e","l", "o","W","r","d"]，怎么办？

首先可能想到的是使用`map` + `Arrays.stream()`方法，后者的作用是将数组转化为流，所以有了以下代码：
```java
menu.stream()
    .map(Dish::getName)
    .map(word -> word.split(""))
    .map(Arrays::stream) // 将数组转为流
    .distinct()
    .collect(toList());
```
然而这是错误的，这个方法的返回值是`List<String[]>`而不是`List<String>`。

`flatMap`能解决这个问题：
```Java
return menu.stream()
    .map(Dish::getName)
    .map(word -> word.split(""))
    .flatMap(Arrays::stream) // 将各个流扁平化成单个流
    .distinct()
    .collect(toList());
```
使用flatMap方法的效果是，各个数组并不是分别映射成一个流，而是映射成流的内容。所有使用map(Arrays::stream)时生成的单个流都被合并起来， 即扁平化为一个流。

另外，map、filter等是可以嵌套使用的，如下：
```Java
/**
 * 只返回总和被3整数的数对
 * @return
 */
private static List<int[]> getFilterPairsList(){
    List<Integer> numbers1 = Arrays.asList(1, 2, 3);
    List<Integer> numbers2 = Arrays.asList(3, 4);
    return numbers1.stream()
            .flatMap(i -> numbers2.stream()
            .filter(j -> (i + j) % 3 == 0)
            .map(j -> new int[]{i, j}))
            .collect(toList());
}
```

## 查找和匹配
以下运算都使用了短路运算
```Java
boolean anyMatch(Predicate<? super T> predicate); // 检查谓词是否至少匹配一个元素

boolean allMatch(Predicate<? super T> predicate); //检查谓词是否匹配所有元素

boolean noneMatch(Predicate<? super T> predicate); // / 检查谓词是否全都匹配一个元素

Optional<T> findFirst(); // 找到第一个

Optional<T> findAny();  // 找到任意一个
```
`findAny`和`findFirst`其实有时候都能实现同一个功能，但是`findFirst`在并行上限制很多，所以尽可能还是用`findAny`。

## 规约
`reduce`操作使得每个元素按照流水线的方式进行计算。
```Java
T reduce(T identity, BinaryOperator<T> accumulator); // identity ： 初始值, accumulator： 累加器

Optional<T> reduce(BinaryOperator<T> accumulator); // accumulator: 累加器， 注意返回值是Optional类型，当流中没有一个元素时候，由于没有初始值，所以无结果，所以结果会被包含在Optional中

<U> U reduce(U identity,
                 BiFunction<U, ? super T, U> accumulator,
                 BinaryOperator<U> combiner);
```
下面是使用 `reduce`的一些例子：
```Java
// 求和
private static int reduceTest(){
   int[] numbers = new int[]{1, 2, 3};

   return Arrays.stream(numbers).reduce(0, Integer::sum)
}
// 得到最大值
private static OptionalInt reduceMax(){
    int[] numbers = new int[]{1, 2, 3};

    return Arrays.stream(numbers).reduce(Integer::max);
}
```
> 分支合并框架

> ## 流操作的状态
> 有状态的操作： 后一次的操作需要知道前一次操作的结果，比如`sorted`、`limit`、`skip`、`distinct`、`reduce`。
> 无状态操作： 操作彼此无关，如`filter`、`map`
> 流操作的状态如下：
>
>![流操作的状态](https://ws2.sinaimg.cn/large/006tKfTcgy1fs7hozbh06j30ua0nk0ww.jpg)


## Stream<T> peek(Consumer<? super T> action);
这个方法会产生一个包含所有元素的新流，接受`Consumer`实例。注意，这个方法产生的是“流”，所以是中间操作，应用如下：
```Java
Stream.of("one", "two", "three", "four")
        .filter(e -> e.length() > 3)
        .peek(e -> System.out.println("Filtered value: " + e))
        .map(String::toUpperCase)
        .peek(e -> System.out.println("Mapped value: " + e))
        .collect(Collectors.toList());
```
程序将会输出：
```Java
Filtered value: three
Mapped value: THREE
Filtered value: four
Mapped value: FOUR
```
这个方法常常应用在debug方法中，用来查询流中间的内容。注意与`forEach`方法作区分，`forEach`方法是终端操作。

## 原始类型流特化
为了减少装箱拆箱地带来成本，Java8引入了三个原始的比特流：`IntStream`，`DoubleStream`和`LongStream`，能够分别将流中的元素转化为int、long和double。

这些原始流中具有特定的对于数字计算有意义的常用的方法，比如`sum`、`avg`、`max`、`min`等。

流转换的方法有三个：
```Java
IntStream mapToInt(ToIntFunction<? super T> mapper);

LongStream mapToLong(ToLongFunction<? super T> mapper);

DoubleStream mapToDouble(ToDoubleFunction<? super T> mapper);
```
相反，将数值流转为对象流的方法是数值流中的`boxed`方法。基本数据流的特性可以参考具体该接口的源码解读。

## 构建流
`Stream`提供了创建流的几种方法：
```Java
// 创建空流
public static<T> Stream<T> empty() {
    return StreamSupport.stream(Spliterators.<T>emptySpliterator(), false);
}

// 由值创建流
public static<T> Stream<T> of(T t) {
    return StreamSupport.stream(new Streams.StreamBuilderImpl<>(t), false);
}
@SafeVarargs
@SuppressWarnings("varargs") // Creating a stream from an array is safe
public static<T> Stream<T> of(T... values) {
    return Arrays.stream(values);
}

// 生成无限流
public static<T> Stream<T> iterate(final T seed, final UnaryOperator<T> f) {
    Objects.requireNonNull(f);
    final Iterator<T> iterator = new Iterator<T>() {
        @SuppressWarnings("unchecked")
        T t = (T) Streams.NONE;

        @Override
        public boolean hasNext() {
            return true;
        }

        @Override
        public T next() {
            return t = (t == Streams.NONE) ? seed : f.apply(t);
        }
    };
    return StreamSupport.stream(Spliterators.spliteratorUnknownSize(
            iterator,
            Spliterator.ORDERED | Spliterator.IMMUTABLE), false);
}

public static<T> Stream<T> generate(Supplier<T> s) {
    Objects.requireNonNull(s);
    return StreamSupport.stream(
            new StreamSpliterators.InfiniteSupplyingSpliterator.OfRef<>(Long.MAX_VALUE, s), false);
}
```
常用`of`方法来创建流，如
```Java
Stream<String> stream = Stream.of("Java8", "Lambda", "In", "Action");
```
另外，源码中看到，还能通过数组来创建流，因为`Arrays`类提供多个重载的`stream`方法，可以生成基本数据流和对象流：
```Java
int[] numbers = {2, 3, 5, 7, 11, 13};
int sum = Arrays.stream(numbers).sum();
```

另外，NIO中`Files`也提供了文件流的创建方法`lines`，这里先略过。

`Stream`还提供了创建无限流的方法：`Stream.iterator`和`Stream.generate`。从集合创建出来的流是有固定大小的，与之相对的“无限流”没有固定的大小，可以无穷无尽地计算下去，通常使用`limit`进行截断。

> 流可以是无界的，这是流和集合之间的区别。

其中`iterator`是依次生成的，我们可以根据这个特性来生成斐波那契数列：
```Java
      Stream.iterator(new int[]{0, 1}, t -> new int[]{t[1], t[0] + t[1]})
      .limit(20)
      .map(t -> t[0])
      .forEach(System.out::println);
```

而`generate`不是依次对每个新生成的值应用函数的。它接受一个Supplier<T>类型的Lambda提供新的值。

比如生成5个任意的0~1之间的双精度浮点数，那么就可以这么做：
```Java
Stream.generate(Math::random)
  .limit(5)
  .forEach(System.out::println)
```

## 计数
终端操作，计算流中元素的个数。
```Java
long count();
```

## 排序
中间操作，且是有状态的操作：
```Java
Stream<T> sorted();

Stream<T> sorted(Comparator<? super T> var1);
```
注意第二个重载方法的参数为`Comparator`实例，可以自己实现，也可以使用`Comparator`接口提供的一些已有的排序，例如`Comparator.naturalOrder`、`Comparator.reverseOrder`。

## 高级归约
流处理的目的是为了完成各种数据提取、转换和归约，下面的方法就是流归约的方法，是终端操作：
```Java
Object[] toArray();

<A> A[] toArray(IntFunction<A[]> var1);

<R> R collect(Supplier<R> var1, BiConsumer<R, ? super T> var2, BiConsumer<R, R> var3);

<R, A> R collect(Collector<? super T, A, R> var1);
```
其中`toArray`方法能将流转为数组，`collect`的两个重载方法非常重要，一般来说，参数定义了归约操作的结果，例如`Collector.toList`就是将流转为线性组，另外还能进行分组等操作，具体参考`Collectors`。

## builder
```Java
static <T> Stream.Builder<T> builder() {
   return new StreamBuilderImpl();
}
```

## 流连接
```Java
static <T> Stream<T> concat(Stream<? extends T> var0, Stream<? extends T> var1) {
    Objects.requireNonNull(var0);
    Objects.requireNonNull(var1);
    java.util.stream.Streams.ConcatSpliterator.OfRef var2 = new java.util.stream.Streams.ConcatSpliterator.OfRef(var0.spliterator(), var1.spliterator());
    Stream var3 = StreamSupport.stream(var2, var0.isParallel() || var1.isParallel());
    return (Stream)var3.onClose(Streams.composedClose(var0, var1));
}
```
