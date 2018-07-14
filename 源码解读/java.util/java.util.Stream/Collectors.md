# Collectors

提供了流的终端操作，具体来讲，它接收的参数是将流中的元素累积到汇总结果的各种方式，我们称之为“收集器”。

![Collectors](https://ws1.sinaimg.cn/large/006tKfTcgy1fs7j0qcqa5j30gl0r50zv.jpg?imageView/2/w/200)

预定义收集器包括将流元素归约和汇总到一个值。如下：
## toList / toSet / toCollection
###  toList
将流中所有元素收集到List中
```java
List<Menu> menu = Menu.getMunus.stream().collect(Collectors.toList());
```
### toSet
将流中的所有元素收集到Set中，删除重复项
```java
Set<Menu> menus = Menu.getMenus.stream().collect(Collectors.toSet());
```
### toCollection
将流中所有元素收集到给定的供应源的集合中
```java
ArrayList<Menu> menu = Menu.getMenus.stream().collect(Collectors.ttoCollection(ArrayList::new))
```

## 计算和统计
### counting
计算流中元素的个数，和`Strem.count()`的功能一样，这个方法，将每个元素映射为1，然后调用了`reduce`方法进行求求和，源码如下：
```java
public static <T> Collector<T, ?, Long>
    counting() {
        return reducing(0L, e -> 1L, Long::sum);
    }
```
一个计算例子如下：
```java
Long count = Menu.getMenus.stream().collect(counting);
```


### summingInt  / summingLong / summingDouble
对流中元素的一个整数属性求和
```java
Integer count = Menu.getMenus.stream().collect(summingInt(Menu::getCalories));
```

### averagingInt / averageingLong / averageingDouble
计算流中元素integer属性的平均值，
```java
Double averageing = Menu.getMenus.stream().collect(averagingInt(Menu::getCalories))；
```

### summarizingInt / summarizingLong / summarizingDouble
计算流中元素的统计特性（最大 / 最小 / 平均 / 计数 / 总和）
```java
IntSummaryStatistics menuStatistics = menu.stream().collect(summarizingInt(Dish::getCalories));
```
打印结果：
```java
IntSummaryStatistics{count=9, sum=4300, min=120, average=477.777778, max=800}
```
### maxBy & minBy
一个包括了流中按照给定比较器选出的最大 / 小元素的optional，如果为空返回的是`Optional.empty()`。
```java
Optional<Menu> fattest = Menu.getMenus.stream().collect(maxBy(Menu::getCalories));
```

## reduce 【广式归约汇总】
之前的所有方法都是`reduce`工厂方法定义的规约的一个特殊情况。所以`reduce`是归约的一般化，其作用是从一个作为累加器的初始值开始,利用binaryOperator与流中的元素逐个结合,从而将流归约为单个值。
例如：
```java
int count=Menu.getMenus.stream().collect(reducing(0,Menu::getCalories,Integer::sum));
```
有三种重载：
```java
public static <T, U>
    Collector<T, ?, U> reducing(U identity,
                                Function<? super T, ? extends U> mapper,
                                BinaryOperator<U> op) {
    return new CollectorImpl<>(
            boxSupplier(identity),
            (a, t) -> { a[0] = op.apply(a[0], mapper.apply(t)); },
            (a, b) -> { a[0] = op.apply(a[0], b[0]); return a; },
            a -> a[0], CH_NOID);
}

public static <T> Collector<T, ?, Optional<T>>
    reducing(BinaryOperator<T> op) {
        class OptionalBox implements Consumer<T> {
        T value = null;
        boolean present = false;

        @Override
        public void accept(T t) {
            if (present) {
                value = op.apply(value, t);
            }
            else {
                value = t;
                present = true;
            }
        }
    }

    return new CollectorImpl<T, OptionalBox, Optional<T>>(
            OptionalBox::new, OptionalBox::accept,
            (a, b) -> { if (b.present) a.accept(b.value); return a; },
            a -> Optional.ofNullable(a.value), CH_NOID);
}

public static <T> Collector<T, ?, T>
    reducing(T identity, BinaryOperator<T> op) {
        return new CollectorImpl<>(
            boxSupplier(identity),
            (a, t) -> { a[0] = op.apply(a[0], t); },
            (a, b) -> { a[0] = op.apply(a[0], b[0]); return a; },
            a -> a[0],
            CH_NOID);
}
```
注意，无论哪种重载，`reducing`其中最重要的参数是`BinaryOperator`接口实例，这就说明这个方法必须接口两个相同类型的参数，然后返回同样类型的值。

## groupingBy / groupingByConcurrent
分组操作，这是一个非常非常常用的操作。`groupingBy`方法有是三个重载：
```java
// 单级分类， 使用分类函数
public static <T, K> Collector<T, ?, Map<K, List<T>>>
    groupingBy(Function<? super T, ? extends K> classifier) {
        return groupingBy(classifier, toList());
    }

// 用来进行多级分类
public static <T, K, A, D>
    Collector<T, ?, Map<K, D>> groupingBy(Function<? super T, ? extends K> classifier,
                                          Collector<? super T, A, D> downstream) {
        return groupingBy(classifier, HashMap::new, downstream);
    }

public static <T, K, D, A, M extends Map<K, D>>
Collector<T, ?, M> groupingBy(Function<? super T, ? extends K> classifier,
                              Supplier<M> mapFactory,
                              Collector<? super T, A, D> downstream) {
    Supplier<A> downstreamSupplier = downstream.supplier();
    BiConsumer<A, ? super T> downstreamAccumulator = downstream.accumulator();
    BiConsumer<Map<K, A>, T> accumulator = (m, t) -> {
        K key = Objects.requireNonNull(classifier.apply(t), "element cannot be mapped to a null key");
        A container = m.computeIfAbsent(key, k -> downstreamSupplier.get());
        downstreamAccumulator.accept(container, t);
    };
    BinaryOperator<Map<K, A>> merger = Collectors.<K, A, Map<K, A>>mapMerger(downstream.combiner());
    @SuppressWarnings("unchecked")
    Supplier<Map<K, A>> mangledFactory = (Supplier<Map<K, A>>) mapFactory;

    if (downstream.characteristics().contains(Collector.Characteristics.IDENTITY_FINISH)) {
        return new CollectorImpl<>(mangledFactory, accumulator, merger, CH_ID);
    }
    else {
        @SuppressWarnings("unchecked")
        Function<A, A> downstreamFinisher = (Function<A, A>) downstream.finisher();
        Function<Map<K, A>, M> finisher = intermediate -> {
            intermediate.replaceAll((k, v) -> downstreamFinisher.apply(v));
            @SuppressWarnings("unchecked")
            M castResult = (M) intermediate;
            return castResult;
        };
        return new CollectorImpl<>(mangledFactory, accumulator, merger, finisher, CH_NOID);
    }
}
```
这里的源码实现可以不用在意，但是一些常见的使用常见需要熟悉：
1. 单级别分类:
利用单参数的`groupingBy`方法，这是上是`groupingBy(f, toList())`的简略写法。如需要将菜肴按照类型分类：
```java
Map<Dish.Type, List<Dish>> dishesByType = menu.stream().collect(groupingBy(Dish::getType));
```
结果就是这样：
```java
{FISH=[prawns, salmon], OTHER=[french fries, rice, season fruit, pizza],  MEAT=[pork, beef, chicken]}
```
2. 多级分类
利用`groupingBy`的双参数重载方法，可以进行n级的分类，可以得到一个n级树形结构的n级Map。
  ```
  /**
   * 二级分类
   * RES: {MEAT={DIET=[chicken], NORMAL=[beef], FAT=[pork]},   FISH={DIET=[prawns], NORMAL=[salmon]},  OTHER={DIET=[rice, seasonal fruit], NORMAL=[french fries, pizza]}}
   * @param menu
   * @return
   */
  private Map<Dish.Type, Map<CaloricLevel, List<Dish>>> getByCaloricLevel(List<Dish> menu){
          return menu.stream().collect(
                  groupingBy(Dish::getType,
                          groupingBy(dish -> {
                              if (dish.getCalories() <= 400) return CaloricLevel.DIET;
                              else if (dish.getCalories() <= 700) return CaloricLevel.NORMAL;
                              else return CaloricLevel.FAT;
                          })
                  )
          );

  }
```
3. 分组统计
更多的时候，我们需要的不仅仅是原数据的分类，而是分类之后的统计。例如：
```java
/**
 * 按照子组收集数据
 * RES: {FISH=Optional[salmon], OTHER=Optional[pizza], MEAT=Optional[pork]}
 * @param menu
 * @return
 */
private Map<Dish.Type, Optional<Dish>> getMostCaloricValueByType(List<Dish> menu){
       return menu.stream().collect(
               groupingBy(Dish::getType,
                       maxBy(Comparator.comparingInt(Dish::getCalories)))
       );
   }
```

4. 分组转换
分组收集之后将元素进行转换
```java
/**
 * 按子组收集数据，查找某个子组中热量最高的Dish
 * RES: FISH=salmon, OTHER=pizza, MEAT=pork}
 * @return
 */
private Map<Dish.Type, Dish> getMostCaloricByType(List<Dish> menu){
    return menu.stream().collect(
            groupingBy(Dish::getType,
                    collectingAndThen(
                            maxBy(Comparator.comparingInt(Dish::getCalories)),
                    Optional::get))
    );
}

```

5. groupingBy和mapping结合使用
```java
/**
 * groupingBy 和 mapping 收集器的结合
 * 获取每类食物下的热量分布
 * RES: {OTHER=[DIET, NORMAL], MEAT=[DIET, NORMAL, FAT], FISH=[DIET, NORMAL]}
 * @param menu
 * @return
 */
private Map<Dish.Type, Set<CaloricLevel>> getCaloricLevelByType(List<Dish> menu){
//        return menu.stream().collect(
//                groupingBy(Dish::getType, mapping(
//                        dish -> {
//                            if (dish.getCalories() <= 400) return CaloricLevel.DIET;
//                            else if (dish.getCalories() <= 700) return CaloricLevel.NORMAL;
//                            return CaloricLevel.FAT;
//                        }, toSet()
//                ))
//        );
    // 控制set的类型
    return menu.stream().collect(
            groupingBy(Dish::getType, mapping(
                    dish -> {
                        if (dish.getCalories() <= 400) return CaloricLevel.DIET;
                        else if (dish.getCalories() <= 700) return CaloricLevel.NORMAL;
                        else return CaloricLevel.FAT;
                    },
                    toCollection(HashSet::new)
            ))
    );
}
```


## mapping
接受两个参数，一个函数对流中的元素进行变换，另外一个将变换的结果收集起来。
```java
public static <T, U, A, R>
    Collector<T, ?, R> mapping(Function<? super T, ? extends U> mapper,
                               Collector<? super U, A, R> downstream) {
        BiConsumer<A, ? super U> downstreamAccumulator = downstream.accumulator();
        return new CollectorImpl<>(downstream.supplier(),
                                   (r, t) -> downstreamAccumulator.accept(r, mapper.apply(t)),
                                   downstream.combiner(), downstream.finisher(),
                                   downstream.characteristics());
    }
```

## partitioningBy
分区函数，这个的分区是指根据谓词进行分区，即非true即false。所以自然可以想到，`partitioningBy`的参数一定有`Predicate`接口实例。
```java
public static <T>
    Collector<T, ?, Map<Boolean, List<T>>> partitioningBy(Predicate<? super T> predicate) {
        return partitioningBy(predicate, toList());
    }

public static <T, D, A>
    Collector<T, ?, Map<Boolean, D>> partitioningBy(Predicate<? super T> predicate,
                                                    Collector<? super T, A, D> downstream) {
        BiConsumer<A, ? super T> downstreamAccumulator = downstream.accumulator();
        BiConsumer<Partition<A>, T> accumulator = (result, t) ->
                downstreamAccumulator.accept(predicate.test(t) ? result.forTrue : result.forFalse, t);
        BinaryOperator<A> op = downstream.combiner();
        BinaryOperator<Partition<A>> merger = (left, right) ->
                new Partition<>(op.apply(left.forTrue, right.forTrue),
                                op.apply(left.forFalse, right.forFalse));
        Supplier<Partition<A>> supplier = () ->
                new Partition<>(downstream.supplier().get(),
                                downstream.supplier().get());
        if (downstream.characteristics().contains(Collector.Characteristics.IDENTITY_FINISH)) {
            return new CollectorImpl<>(supplier, accumulator, merger, CH_ID);
        }
        else {
            Function<Partition<A>, Map<Boolean, D>> finisher = par ->
                    new Partition<>(downstream.finisher().apply(par.forTrue),
                                    downstream.finisher().apply(par.forFalse));
            return new CollectorImpl<>(supplier, accumulator, merger, finisher, CH_NOID);
        }
    }
```
一个典型的应用例子是：
```java
/**
 * 判断一个数是不是质数
 * @param candidate
 * @return
 */
private boolean isPrime(int candidate){
    int candidateRoot = (int) Math.sqrt((double) candidate);
    return IntStream.rangeClosed(2, candidateRoot)
            .noneMatch(i -> candidate % i == 0);
}

/**
 * 将1~n的数划分为质数和非质数
 * @param n
 * @return
 */
private Map<Boolean, List<Integer>> partitionPrimes(int n){
    return IntStream.rangeClosed(2, n).boxed()
            .collect(
                    partitioningBy(candidate -> isPrime(candidate))
            );
}
```

![收集器](https://ws3.sinaimg.cn/large/006tKfTcgy1fsb2e2i071j31bq1a4n9y.jpg)
![收集器](https://ws1.sinaimg.cn/large/006tKfTcgy1fsb2eq684yj31bo0i20xz.jpg)
