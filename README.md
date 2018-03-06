# Introduction

![Java基础知识体系](http://ovn0i3kdg.bkt.clouddn.com/Java%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86%E4%BD%93%E7%B3%BB.png)


复习的正确顺序：
* JDK
* Object -> Objects
* 基本数据类型 -> 八种包装类 -> 包装类源码
* String 高效编程 ->  String -> StringBuilder - > StringBuffer -> **正则Pattern** -> Java编译技术
* OOP -> 修饰符 -> 类、抽象类、内部类和接口 -> 设计模式
* 类编译、加载和执行 -> ClassLoader -> 反射Class -> reflect -> 虚拟机
* 防御式编程 -> 异常 -> 断言
* 集合框架 -> Collection| ArrayList |LinkedList | HashSet | LinkedHashSet | TreeSet | HashMap | LinkedHashMap| TreeMap | PriorityQueue| ConcurrentHashMap | Collections | Arrays-> 空间增长原则 -> 数据结构（数组、线性表、栈、队列、树） -> 算法
* 线程 -> Thread -> ThreadGroup -> ThreadPool -> concurrent包


需要注意的点：
## Object
* clone方法注意是protected、需要实现cloneable接口，返回值是Object
* equals -> hashcode
* Objects对object有哪些改进？

## 包装类
* Integer和Long中计算数字长度的方法
* toString方法如何实现？
* hashcode的计算方法
* 自动装箱和自动拆箱
* Integer中的valueOf方法是如何实现的。
* Integer中的parseInt方法是如何实现的？

## String
* getCharts方法的返回值是void，需要将结果保存的字符数组作为参数传入。
* equals方法和hashcode方法的写法
* replace(char)方法是字符匹配，replace(String)方法是正则匹配
* split第1个参数是正则式，第二个参数（正数、负数和0）的含义？
* 学会用join方法
* 常用toCharArray方法
* 常用format方法，百分号转义？
* 理解intem方法在1.7之前和之后的区别，深刻理解方法区常量池保存堆中的内存

## StringBuilder
* 空间增长策略（2x + 2），最大值、初始化长度
* applend(null)会追加"null"
* 线程不安全


## StringBuffer
* 线程安全

## OOP
* 面向对象三个重要的特性
* 9种修饰符的含义
* 内部类、抽象类、接口
* 类的编译、加载、执行过程
* 类加载器的双亲委托
* 三种loader的默认路径和自定义路径
* 自定义classloader需要重写findClass方法，在其中调用defineClass方法
* defineClass方法被调用时生成class对象
* ClassLoader工作过程：装载 | 链接（检查、准备、解析） | 初始化
* 类的初始化顺序
* JVM运行时的内存分区
* GC含义
* 判定一个对象是垃圾的方法（2种）
* 内存回收算法（4种），各自的特点是适合的对象
* 内存回收器（7种），各自适合什么对象，各自工作流程
* 内存溢出和内存泄漏的区别
* 内存调优
* OOM日志查看


## 反射
* 反射的意义，为什么用反射？
* 创建Class对象的三种办法
* newInstance调用的是类的无参构造方法，且返回的是Object类型的对象
* 类实例化的三种方法
* invoke方法返回值是Object方法
* 触发方法的两种方法
* declare有无的区别
* Field中操作属性的方法（get和set），在对private属性进行操作之前需要setAccessible(true)
* 基本数据类型和void也有class对象、数组也有class对象
* Array创建数组和操作数组的方法。
* Array不能实例化，newInstance方法返回值是Object

## 异常处理机制
* 异常和断言的区别
* 异常机制结构，常见的免检异常
* finally的作用，return语句，资源释放顺序
* try-with-resources语句的作用
* 1.7对于catch语句的改进
* multi-catch

## **集合框架和算法**
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


## 多线程
* 背景：CPU、多核处理器、多处理器、多道程序设计、并行、并发
* 进程和线程的概念、区别
* 多核心处理器、内核线程、超线程技术
* 线程竞争状态：**只有对共享区域的数据进行操作的时候才会有这样的问题**，准确来说，是对**方法区**和**堆区**的数据进行操作的时候才有这个问题。对于局部变量啊这种的，不是共享的数据，不会有问题！！！！不能理解线程的原因在这里，清醒一点！！！
* synchronized关键字获取对象锁和类锁，两者有什么区别？
* 假设对象可能有很多方法，其中包括很多synchronied方法和非synchronized方法，那么同一时刻，只能有一个synchronized可能被访问，但是！其他非synchronized方法照样能被访问。 房间和钥匙的比喻！！！牢记
* 理解“synchronied是线程级的而不是方法级的”； “加锁的是对象而不是代码”
* “同步”和“互斥”有什么区别。同步是只等待同一个资源，使得有先有后，方法就是加**同一把**锁。互斥是指同一时刻只有一个线程能获得，方法是加锁。
* synchronied叫做监视器锁，理解监视器的概念和实现
* 梳理JUC包都包含哪些类，尤其是locks子包中都有哪些重要的接口和类
* AQS提供线程队列的操作，LockSupport提供线程唤醒和阻塞原语。
* 理解各种锁与AQS之间的关系，为什么说AQS提供了基本同步框架。
* Lock接口中lock、tryLock、lockInterruptibly三种方法的用法和区别，以及各自的使用场景
* ReentrantLock如何实现AQS和Lock。
* 什么是公平锁和非公平锁？ReenrantLock默认是什么锁。
* 线程同步的意思？ 阻断其他线程
* ReentrantReadWriteLock性能？读锁和写锁如何工作?适用于什么场景。
* 三种线程同步辅助类Semaphore、CountDownLatch、CyclicBarrier的作用和使用场景。
* 创建线程的四种方法，哪种比较好。
* Executor框架主要包含哪些东西？
* 为什么要使用线程池。
* Executor和Executors的区别和联系
* 创建线程的四种方式
* 线程池和线程组有什么区别和联系
* 线程的生命周期！！！！各种状态转化方法的区别
* 线程中断是什么意思？哪些可以中断，哪些不能中断？如何检测中断？


I/O
* I/O流的分类（字节字符、输入输出、目标媒介）
* 文件File
