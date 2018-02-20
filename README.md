# Introduction

![Java基础知识体系](http://ovn0i3kdg.bkt.clouddn.com/Java%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86%E4%BD%93%E7%B3%BB.png)


复习的正确顺序：
* JDK
* Object -> Objects
* 基本数据类型 -> 八种包装类 -> 包装类源码
* String 高效编程 ->  String -> StringBuilder - > StringBuffer -> **正则Pattern** -> Java编译技术
* OOP -> 修饰符 -> 类、抽象类、内部类和接口 -> 设计模式
* ClassLoader -> 类编译、加载和执行 -> 反射Class -> reflect -> 虚拟机
* 防御式编程 -> 异常 -> 断言
* 集合框架 -> Collection| ArrayList |LinkedList | HashSet | LinkedHashSet | TreeSet | HashMap | LinkedHashMap| TreeMap | PriorityQueue| ConcurrentHashMap | Collections | Arrays-> 空间增长原则 -> 数据结构（数组、线性表、栈、队列、树） -> 算法
* 线程 -> Thread -> ThreadGroup -> ThreadPool -> concurrent包


需要注意的点：
###### Object
* clone方法注意是protected、需要实现cloneable接口，返回值是Object
* equals -> hashcode
* Objects对object有哪些改进？

###### 包装类
* Integer和Long中计算数字长度的方法
* toString方法如何实现？
* hashcode的计算方法
* 自动装箱和自动拆箱

###### String
* getCharts方法的返回值是void，需要将结果保存的字符数组作为参数传入。
* equals方法和hashcode方法的写法
* replace(char)方法是字符匹配，replace(String)方法是正则匹配
* split第1个参数是正则式，第二个参数（正数、负数和0）的含义？
* 学会用join方法
* 常用toCharArray方法
* 常用format方法，百分号转义？
* 理解intem方法在1.7之前和之后的区别，深刻理解方法区常量池保存堆中的内存

###### StringBuilder
* 空间增长策略（2x + 2），最大值、初始化长度
* applend(null)会追加"null"
* 线程不安全


###### StringBuffer
* 线程安全

###### ClassLoader
* 双亲委托
* 三种loader的默认路径和自定义路径
* 自定义classloader需要重写findClass方法，在其中调用defineClass方法
* defineClass方法被调用时生成class对象
* classLoader工作过程：装载 | 链接（检查、准备、解析） | 初始化

###### 反射
* 反射的意义，为什么用反射？
* 创建Class对象的三种办法
* 类实例化的三种方法
* 触发方法的两种方法
* declare有无的区别
* Field中操作属性的方法（get和set）
* 基本数据类型和void也有class对象、数组也有class对象

##### **集合框架和算法**
* 框架大概情况
* iterable和iterator的区别
* Iterator中remove方法的用法
* 迭代器的操作是否会影响原来的集合？
* 实现自定义的迭代器类? 迭代器类需要实现Iterable接口，其中iterator方法中定义实现Iterator接口的内部类，返回一个接口实例。
* ListIterator对Iterator做了哪些改进？
* 线性表、栈、队列（peek | element | poll | remove | offer）的实现和方法
* toArray方法写法
* ArrayList和数组的相互转换
* HashMap中三种视图和各自的迭代器
* HashMap中的存储结构Node<K,V> table
* HashMap默认初始化大小，默认装载因子、扩容策略
* HashMap中对哈希值的重新定义
* HashMap寻址和冲突解决办法 putVal
* map如何进行遍历
* 弄明白`HashMap`、`LinkedList`中数据结构的继承关系
* 数据结构中数组、线性表、栈、队列、树
  - 数组存储地址的计算
  - 三种特殊矩阵的压缩存储原理、稀疏矩阵的表示
  - 线性表的顺序存储和链式存储的实现
  - 栈的顺序存储和链式存储的实现
  - 队列的顺序存储(循环队列)和链式存储的实现
  - 二叉树定义、常用结论
  - 二叉树的三种遍历方式的非递归实现（重要）
  - BST、AVL、B-、B+、B*树之间的区别和联系，各自的查找、插入和删除实现过程
  - Collections中的算法实现：排序(sort)、反转(rotate)、二分查找（binarySearch）、混排(shuffle)、填充（fill）、拷贝(copy)、替代（replaceAll）、子串匹配(indexOfSubList和lastIndexOfSubList)、最大(max)、最小(min)
  - Arrays中的算法实现
  - 查找算法（7种）：顺序、二分、差值、斐波那契、树表（BST、AVL、B+、B-、B*、红黑树）、分块查找、哈希查找 及各自的效率
  - 排序算法（5类8种）：插入排序（直接插入、希尔）、选择（简单选择、堆排序）、交换（冒泡、快排）、归并、基数排序 及各自的效率
