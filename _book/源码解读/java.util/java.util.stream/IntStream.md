# IntStream/LongStream/DoubleStream
为了避免基本数据类型装箱开销而提供的基本类型流，类似的还有`DoubleStream`和`LongStream`。三者的结构差不过，所以这里以`IntStream`为例：

![IntStream](https://ws2.sinaimg.cn/large/006tKfTcgy1fs8bys9tyij30b60ohmzf.jpg)

可以看到，其中很多的方法与`Stream`是一样的（那是因为两者都继承了`BaseStream`接口），所以这里单独看下基本数据流特有的几个方法。

整数有很多统计的场景，`IntStream`为我们提供了这些的接口方法：

| 描述  | 方法 |
| :------------- | :------------- |
| 求和 |  int sum()|
| 计数  |   long count();|
| 求最小值    |  OptionalInt  min()|
| 求最大值   |  OptionInt max() |
| 求平均值   | OptionalDouble average()  |
| 获取数据统计（最大、最小、平均、计数） | IntSummaryStatistics summaryStatistics();  |


另外，三种基本数据流之间也能相互转换

| 流转换 | 方法名 |
| :------------- | :------------- |
| IntStream -> LongStream |  LongStream asLongStream();       |
|  IntStream -> DoubleStream |  DoubleStream asDoubleStream(); |

`LongStream`和`DoubleStream`中也有类似的方法，命名都是"xxxStream"。
`Stream`通过`mapToXxx`方法转换为基本数据流，而基本数据流通过`boxed`方法转换为`Stream`。

另外，`IntStream`和`LongStream`有两个特殊的静态方法方法，用来产生某范围内的整数序列。
```Java
public static IntStream range(int startInclusive, int endExclusive) {
    if (startInclusive >= endExclusive) {
        return empty();
    } else {
        return StreamSupport.intStream(
                new Streams.RangeIntSpliterator(startInclusive, endExclusive, false), false);
    }
}
public static IntStream rangeClosed(int startInclusive, int endInclusive) {
    if (startInclusive > endInclusive) {
        return empty();
    } else {
        return StreamSupport.intStream(
                new Streams.RangeIntSpliterator(startInclusive, endInclusive, true), false);
    }
}
```
两者的区别在于： `range`不包含结束值，而`rangeClosed`包含结束值。例如“生成一个从1到100的偶数流”，则可以这么做：
```java
IntStream evenStream = IntSteam.rangeClosed(1, 100)
                      .filter(n -> n % 2 == 0);
```
下面是一个数据流的比较综合的例子：寻找100以内的勾股数，例如[3,4,5]
首先想到勾股数需要满足`a * a + b * b = c * c` ，其中a和b的范围是[1, 100]，所以首先可以想到：
```Java
IntStream.rangeClosed(1, 100)
    .filter(b -> Math.sqrt(a * a + b * b) % 1 == 0)
    .mapToObj(b -> new int[]{a, b, (int)Math.sqrt(a * a + b * b)})
```
那么a是如何生成呢？
```Java
Stream<int[]> pythagoreanTruples =
  IntStream.rangeClosed(1, 100).boxed()
    .flapMap(a ->
        IntStream.rangeClosed(1, 100)
            .filter(b -> Math.sqrt(a * a + b * b) % 1 == 0)
            .mapToObj(b ->
            new int[]{a, b, (int)Math.sqrt(a * a + b * b)})
    );    
```
至此，上面的代码就可以生成勾股数流了，但是上面并不是最好的方案，因为开发计算过程发生了两次，所以更好的方法如下：
```Java
Stream<double[]> pytghagoreanTruples2 =
  IntStream.rangeClosed(1, 100).boxed()
    .flatMap(a -> IntStream.rangeClosed(1, 100))
            .mapToObj(b -> new double[]{a, b, Math.sqrt(a * a + b * b)})
            .filter(t -> t[2] % 1 == 0);
```
然后我们利用截断操作`limit`，就可以限定从生成的流中要放回多少组勾股数了：
```Java
pytghagoreanTruples2.limit(5)
  .forEach(t ->
      System.out.println(t[0] + “,” + t[1] + ", " + t[2]));
```
