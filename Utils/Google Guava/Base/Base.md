# Base

这是最基础的包，主要包含一些工具类，使得Java语言使用更加便利。按照官方文档，可以大致分为以下三类：

![google guava base](https://ws3.sinaimg.cn/large/006tNc79ly1fvt5klra7tj30cg0scgm4.jpg)


`google.guava.base`首先包含了对于字符串处理工具类：

* [`Ascii`](./Ascii.md)中针对Ox00-Ox7F的字符和字符串提供了一些静态处理方法，从源码中学习到的最关键的方法在于大小写字母之间的一些规律，该类提出了`CASE_MASK = Ox20`，例如大小写的转化就是其中与`CASE_MASK`的异或结果，取得字母c在字母表中的索引的方法是`(char)(c | CASE_MASK) - 'a')` 。


* [`CaseFormat`](./CaseFormat.md)用来对进行字符串形式的快速转换，这里的“形式”指Java或C++中变量和类命名的常用形式，例如驼峰式、下划线式等。注意`CaseFormat`是枚举型，特殊的是，这个枚举型中包含了抽象方法，**该情况下，枚举类生命中不需要`abstract`关键字**，注意学习这种用法。

* [`CharMatcher`](./CharMatcher.md)是`Predicate`的一个实现，用来判定某种特性是否成立，该类被应用到了`CaseFormat`中，用来区分不同的`CaseFormat`类型。这个类在一定程度上完成了类似于正则表达式的功能。


* [`Joiner`](./Joiner.md)用来拼接字符串，它的作用与JDK8中`Collector.joining()`和`StringJoiner`类似。`Joiner`无公开的构造方法，需要通过静态方法`on`方法来指定分隔符的方式创建实例。特别的是，`Joiner`可以对`null`值进行特殊的处理，同时对`Map`实例具有同样的拼接效果，即创建内部类`MapJoiner`的实例。另外`Joiner`是不可变的，不可变的原因在于其中用于保存分隔符的变量`seperator`被final修饰。因此`Joiner`的实例可以被`static final`修饰，作为常量。特别注意`skipNulls`的用法，该方法将会返回一个全新的`Joiner`实例。


* [`Splitter`](./Splitter.md)具有和`Joiner`完全相反的功能，用来分割字符串，比JDK中`String`实例方法`split`具有更好的定制性。有两种静态方法来创建基础实例：一是通过`on`方法指定分隔符（字符/字符串/正则），而是通过`fixedLength`来创建实例，注意后者中`length`指代结果集合中元素的长度（限定结果集合的长度可以通过`limit`方法）。`Splitter`提供了很多定制化方法，以实例方法的形式呈现，例如去除元素两侧指定子串`trimResults`、限定结果集个数`limit`，结果集去除空串`omitEmptyStrings`等方法。需要注意，`Splitter`实例是不可变的，所以这些方法会在基础的`Splitter`方法之上重新`new`一个实例，类似于**包装器设计模式**，所以我们常常以“链式写法”来使用`Splitter`，例如`Splitter.on(',').omitEmptyStrings().trimResults().splitToList("a ,, b, c");`另外，在这个类的实现中，特别体现了**迭代器设计模式**思想，巧妙又经典，值得一看。

* [`Strings`](./Strings.md)是JDK`String`的工具类，以静态方法的行驶提供辅助功能。这些功能大概不太经常会用，比如对于字符串为null和空串的转化、快速创建子串周期重复的字符串、获取字符序列的公共前缀、后缀、格式化等。

第二部分是函数是编程部分，这部分内容是对于JDK函数式编程（参见java.util.function）的补充。这一部分大概率用不到，因为JDK8中的函数式编程已经足够满足日常开发需要，并且lambda表达式还能简化开发。所以这部分只用看看就好。
* [`Function`](./Function.md)继承了JDK8中的`Function`接口，只是用来作为一个过渡而已。
* [`Functions`](./Functions.md)提供了一些日常的类型转化类型，但是基本上不太常用到。
* [`Predicate`](./Predicate.md)继承了JDK8中`Predicate`接口。
* [`Predicates`](./Predicates.md)提供了一些常见的判定类型的实现。
...
因为这部分实际上不太常用，所以不再赘述。这部分源码学习到的一点技巧是**用枚举型来实现单例**，对于一些工具类的编写，可以借助内部来类进行良好的组织。而不是一个方法一个坑。


第三部分是一些辅助类。有一些比较好用的，例如：
* [`Optional`](./Optional.md)：同样在JDK8中实现了对于Null值的预防处理。还是建议用JDK中的`Optional`实现。
* [`Stopwatch`](./Stopwatch.md)：实际上就是计时器的封装，我们在系统中简单使用`System.currentTime()`也能达到效果。
* [`Preconditions`](./Preconditions.md)： 比较常用的类，用于前置条件检查，即我们所作的参数检查。实现都非常简单，但是对于代码语义的理解和组织规范有很大的帮助。建议常用。
