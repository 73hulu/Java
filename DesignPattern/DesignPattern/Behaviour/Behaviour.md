# 行为型模式
行为型模式对类或对象怎样交互和怎样分配职责进行描述。有以下11种模式定义：

## 责任链模式(Chain of Responsibility) —— 击鼓传花
这种方法有点类似于“击鼓传花”。每一个处理者在无法处理的时候将问题抛给下一个人，直到有一人解决这个问题。不同在于，游戏“击鼓传花”的参与者在鼓声停下来的时候需要得出结果，由解决问题的人（最后手握花球的人）需要将答案报给问题的提出者（击鼓者），而责任链模式中，问题的提出者只需要和链中第一个人有交流即可。

在JDK和扩展包中，以下类中的方法使用了责任链模式：

* `java.util.logging.Logger#log()`
* `javax.servlet.Filter#doFilter()`

UML图如下：

![责任链模式](https://ws1.sinaimg.cn/large/006tNbRwly1fx1wrk2fkbj30qi0c6mxp.jpg)

## 命令模式(Command) —— 甲方乙方
命令模式是生活中太常见的一个问题了。这就是甲方和乙方的关系，甲方（即调用者）只需要提出需求（即命令（Command）），然后交付给乙方的产品经理（即Invoker），对于每一种需求，大约都有一套既定的流程和实施者，此时经理把需求往需求池中一挂（即触发`command.execute`方法），自会有人（即Receiver来解决）。

在JDK和扩展包中，以下类中的方法使用了命令模式：
* `java.lang.Runnable`
* `javax.swing.Action`


UML图如下：

![命令模式](https://ws3.sinaimg.cn/large/006tNbRwly1fx41mnnjhtj30iw09a74v.jpg)

> 命令模式和策略模式的类图非常相似，只是命名模式多了一个接收者角色，但两者还是有较大的区别的。策略模式的意图是封装算法，它认为“算法”已经是一个完整的、不可拆分的原子业务（注意这里是一个原子业务，而不是原子对象），即其意图是为了让这些算法独立，并且可以相互替换，让行为的变化独立与拥有行为的客户；而命令模式则是对动作的解耦，把一个动作的执行分为执行对象（接受者橘色）、执行行为（命令角色），让两者相互独立而不相互影响。


## 解释器模式(Interpreter) —— 递归文法分析
解析器模式是一个简单的语法分析工具，简而言之，就是定义终结符表达式和非终结符表达式，递归解析语法，已有很多轮子，不常用。

在JDK和扩展包中，以下类中的方法使用了解释器模式：

* `java.util.Pattern`
* `java.text.Normalizer`
* `java.text.Format`
* `javax.el.ELResolver`

UML图如下：

![解释器模型](https://ws1.sinaimg.cn/large/006tNbRwly1fx30imjtroj30q00feq3v.jpg)

## 迭代器模式(Iterator) —— 优雅遍历
迭代器模式针对容器提供了一种屏蔽内部细节而提供了一种简单的遍历方式。Java提供的`Iterator`一般就能满足要求，尽量不要自己写迭代器模式。注意接口`Iterable`和`Iterator`的区别。

在JDK和扩展包中，以下类中的方法使用了迭代器模式：

* `java.util.Iterator`
* `java.util.Enumeration`

UML图如下：

![迭代器模式](https://ws2.sinaimg.cn/large/006tNbRwly1fx1xwaetujj30n60k4gml.jpg)


## 中介者模式(Mediator)——星型拓扑
中介者模式将互相依赖的对象构成的网状结构简化成了星形拓扑，起到中场调度的作用，把原有一对一的依赖变成了一对一的依赖。典型的例子就是MVC中的C。

* `java.util.Timer` (所有scheduleXXX()方法)
* `java.util.concurrent.Executor#execute()`
* `java.util.concurrent.ExecutorService `(invokeXXX()和submit()方法)
* `java.util.concurrent.ScheduledExecutorService `(所有scheduleXXX()方法)
* `java.lang.reflect.Method#invoke()`

UML图如下：

![中介者模式](https://ws2.sinaimg.cn/large/006tNbRwly1fx31g702b6j30fg094t8v.jpg)


##  备忘录模式(Memento) —— 重置回滚
备忘录模式就是给了一颗后悔药，让你能保存当前的状态，在以后能重置回滚。该模式的一个典型应用就是事务管理的回滚。

在JDK和扩展包中，以下类中的方法使用了备忘录模式：

* `java.util.Date`
* `java.io.Serializable`
* `javax.faces.component.StateHolder`

UML图如下：

![备忘录模式](https://ws4.sinaimg.cn/large/006tNbRwly1fx325g2x91j30qq06a3z5.jpg)

##  观察者模式(Observer) —— 订阅发布
观察者模式也叫做“订阅发布”模式，即被观察者一旦发生某种变化，观察者就响应这种改变。这么做到呢？在Java中，`java.util.Observable`和`java.util.Observer`接口已经为我们完成了观察者模式的基础的搭建。

在JDK和扩展包中，以下类中的方法使用了观察者模式：

* `java.util.Observer/java.util.Observable`
* `java.util.EventListener` (所有子类)
* `javax.servlet.http.HttpSessionBindingListene`
* `javax.servlet.http.HttpSessionAttributeListener`
* `javax.faces.event.PhaseListener`

UML图如下：

![观察者模式](https://ws4.sinaimg.cn/large/006tNbRwly1fx27dy5l2vj30hq0a9gnd.jpg)

## 状态模式(State) —— 自动执行的状态机
状态模式适用于当某个对象在它的状态发生改变时，它的行为也随着发生比较大的变化（对象的状态最好不要超过5个）。用一个环境类定义所有状态类型及当前状态，将具体的行动委托给当前的状态去处理。就像是一个指挥者和执行者。每个执行者在执行完之后告诉指挥者下一个状态是什么，指挥者更新当前状态，并去寻找下一个状态，指示它去完成任务。

* `java.util.Iterator`
* `javax.faces.lifecycle.LifeCycle#execute()`

UML图如下：

![状态模式](https://ws4.sinaimg.cn/large/006tNbRwly1fx2zk6w79bj30ok0jm75r.jpg)


## 策略模式(Strategy) —— if-else代替
定义了算法家族，分别封装起来，让它们之间可以互相替换。简单来说，就是当我们在一次处理过程中，做出很多if-else判断来选择不同的处理方法的时候，就可以采取这种模式。

在JDK和扩展包中，以下类中的方法使用了策略模式：
* `java.util.Comparator#compare()`
* `javax.servlet.http.HttpServlet`
* `javax.servlet.Filter#doFilter()`

UML图如下：

![策略模式](https://ws3.sinaimg.cn/large/006tNbRwly1fx27bk7gz6j30if06tjs9.jpg)


## 模板方法模式(Template Method)——模板流程化
模板方法就是在顶层抽象类中将共同步骤抽取出来形成规范，而其中的细节由继承者去实现。这是一个完成由继承关系实现的模式。在JDK中，一个非常典型的例子就是`AQS`，在其中使用模板方法实实现了锁的基本操作，而具体的实现尤其各个继承者来具体实现啊。

在JDK和扩展包中，以下类中的方法使用了模板模式：

* `java.io.InputStream, java.io.OutputStream, java.io.Reader和java.io.Writer`的所有非抽象方法
* `java.util.AbstractList, java.util.AbstractSet和java.util.AbstractMap`的所有非抽象方法
* `javax.servlet.http.HttpServlet#doXXX()`

UML图如下：

![模板设计模式](https://ws1.sinaimg.cn/large/006tNbRwly1fx1q6qgjifj30oa0hgt9o.jpg)

## 访问者模式(Visitor) —— 授权转述整理
访问者模式就是访问者在得到被访问者允许的情况下，转述并整理被访问者的内容。某被访问者的内容太多了，话不知道从何说起，那么它就允许一个访问者来整理并转述自己的话。

在JDK和扩展包中，以下类中的方法使用了访问者模式：

* `javax.lang.model.element.AnnotationValue和AnnotationValueVisitor`
* `javax.lang.model.element.Element和ElementVisitor`
* `javax.lang.model.type.TypeMirror和TypeVisitor`

UML图如下：

![Visitor](https://ws3.sinaimg.cn/large/006tNbRwly1fx2ya0yk73j30ok0jm75r.jpg)
