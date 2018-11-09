# 集合

`google.guava.collect`几乎是Google Guava的核心。不单单因为这是被使用最多的类，而且它提供了对于JDK单薄的集合类的诸多概念的补充和提炼。

首先最重要的概念是“不可变集合”。

## 不可变集合

Immutable，不可变，字面上来讲就是确定了之后内容不再改变。对于集合集合而言，初步的理解是初始化之后，集合的内容不再改变。

这并非是第一次遇到“不可变的”这一概念。

在JDK中，有一个经典的重要的基础类——`String`。查看源码可以发现，设计者通过`final`和`private`的设计让`String`称为不可变类。这样做有什么好处？这个问题在[在java中String类为什么要设计成final？](https://www.zhihu.com/question/31345592)中被讨论，用一个词来概括老说，就是“安全”。

安全表现在很多方面。比如对于哈希集合来说，我们常常用某一对象作为键值或元素，如果对象可变，那么将会在无意中改变对象值，违法哈希集合“不能相同”的约定。例如下面这个例子：
```Java
class Test {

  public static void main(String[] args){
    HashSet<StringBuilder> hs = new HashSet<StringBuilder>();
    StringBuffer sb1 = new StringBuffer("aaa");
    StringBuffer sb2 = new StringBuffer("aaabbb");
    hs.add(sb1);
    hs.add(sb2); ①// 这时候hs里是{"aaa", "aaabbb"}

    StringBuffer sb3 = sb1;②
    sb3.append("bbb"); ③// 这时候hs中是{"aaabbb", "aaabbb"} 违反了HashSet元素唯一性的约定
    System.out.println(hs);
  }
}
```
在上面例子中，`HashSet`中键值类型是`StringBuffer`，其内容是可变的。到第①步的时候，程序完全没有问题，问题在②操作将`sb3`指向了`sb1`，并且对其进行修改，因此`sb1`指向的地址中的内容变化了。这将倒置`HashSet`中出现两个相等的键值"aaabbb"，**破坏了HashSet键值唯的唯一性，所以千万不要用可变类型作为`HashSet`或者`HashMap`的键值。**

“安全”的另一个示例是在高并发状态下的安全性。高并发状态下多线程同时读一个资源，不会引发竞态条件。可以，当对同一资源进行写操作时会有危险。**不可变对象不能被写，所以线程安全。**

不可变对象因为固定不变，所以可以作为常量来安全使用。
> 什么样的对象能作为常量呢？不可变的对象！

JDK1.2开始提供了集合框架以满足开发需求。在`Collections`工具类中提供了不可变集合方法`UnmutableXXX`。其原理都是将原来的集合进行一层的封装，并且屏蔽其中对结合的修改方法（“屏蔽”的含义就是对修改方法进行包装并抛出异常）。这是一种“假”的“不可变”，因为如果使用员阿里的集合引用还是能够改变集合成员的，是“可变的”。同时由于包装太过简单，包装过的集合保有可变集合的开销，比如并发修改的检查、散列表的额外空间等等。


因为Google Guava为JDK中的每个集合类都提供了`ImmutableXXX`类，提供高效且安全的、真正的不可变集合类。

清单如下:

![可变集合和不可变集合的对应关系](https://ws1.sinaimg.cn/large/006tNbRwly1fw4ou56，4f8j319g144jsx.jpg)

> [ImmutableCollectionsExplained](https://github.com/google/guava/wiki/ImmutableCollectionsExplained)
