# java.util

Java工具包(java.util)是非常非常重要的包，它提供了一些使用的数据结构，包含集合框架、遗留的 collection 类、事件模型、日期和时间设施、国际化和各种实用工具类（字符串标记生成器、随机数生成器和位数组、日期Date类、堆栈Stack类、向量Vector类等）。

主要包含的接口如下：

| 接口 | 功能 |
| :------------- | :------------- |
| Collection<E> |  集合层次机构中的根接口|
|Set<E>	   |  不能包含重复元素的集合 |
|List<E>   |有序集合，也叫序列|
|Map<K,V>  | 键值对  |
|Queue<E>   | 队列接口，设计用于在处理之前保留元素的集合。|
|Deque<E>   | 支持两端元素插入和移除的线性集合。  |
| Comparator<T>	 | 比较接口，对一些对象的集合施加了一个整体的排序。   |
|  Iterator<E> | 一个集合的迭代器   |
|  ListIterator<E> |  用于列表的迭代器，允许程序员沿任一方向遍历列表，在迭代期间修改列表，并获取列表中迭代器的当前位置。 |
|SortedSet<E>   | 进一步提供基于元素的排序的set  |
|SortedMap<K,V>   |  进一步提供key的排序的map |


常用的重要的类如下：

| 类名 | 功能 |
| :------------- | :------------- |
| AbstractCollection<E>	       | 该类提供了Collection接口的骨架实现，以最大限度地减少实现此接口所需的工作量。       |
|AbstractSet<E>   | 该类提供了Set接口的骨架实现，以最大限度地减少实现此接口所需的工作量。  |
|HashSet<E>|此类实现了Set接口，由哈希表（实际上是HashMap实例）支持。|
|LinkedHashSet<E> |  哈希表和链接列表实现的Set接口，具有可预测的迭代顺序。 |
|AbstractList<E> |   该类提供了List接口的骨架实现，以最大限度地减少实现此接口所需的工作量。  |
|ArrayList<E>	 | List接口的可调整大小的数组实现  |
|LinkedList<E>   | List和Deque接口的双向链表实现。  |
|AbstractMap<K,V> |该类提供了Map接口的骨架实现，以最大限度地减少实现此接口所需的工作量。   |
|HashMap<K,V>|基于哈希表的Map接口实现。|
|LinkedHashMap<K,V>	|哈希表和链接列表实现的Map接口，具有可预测的迭代顺序。|
|Hashtable<K,V>   |   哈希表类，该类实现了一个哈希表，它将键映射到值。|
|WeakHashMap<K,V>   |  基于哈希表的实现Map界面，具有弱键。 |
|AbstractQueue<E>   |  该类提供了Queue接口的骨架实现 |
|PriorityQueue<E>   |  具有优先级的队列 |
|Collections   | 这个类完全由静态方法组成或返回集合。  |
|Stack<E>	   |  栈类，Stack类代表一个最先进先出（LIFO）堆栈的对象。 |
|Vector<E>   | 向量类，Vector类实现可扩展的对象数组。  |
|Formatter   |  printf样式格式字符串的解释器。 |
|  Random |  随机数生成器 |
| StringTokenizer	|   字符串tokenizer类允许应用程序将字符串拆分成令牌。|
|Calendar   |  日历类 |
|  Date | 日期类，Date代表一个特定的时间，以毫秒的精度。  |
|Timer   |  线程调度任务以供将来在后台线程中执行的功能。 |
|UUID   |   一个代表不可变的通用唯一标识符（UUID）的类。|
|Scanner   |  一个简单的文本扫描器，可以使用正则表达式解析原始类型和字符串。 |
|BitSet   |  位集合类，封装了一组二进制数据的操作 |


参考
* [Java 8 API](https://docs.oracle.com/javase/8/docs/api/java/util/package-summary.html)
* [java util包概述](http://www.cnblogs.com/frankliiu-java/articles/1944276.html)
