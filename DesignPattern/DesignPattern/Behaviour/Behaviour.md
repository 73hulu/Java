# 行为型模式
行为型模式对类或对象怎样交互和怎样分配职责进行描述。有以下11种模式定义：

## 责任链模式(Chain of Responsibility) —— 击鼓传花
这种方法有点类似于“击鼓传花”。每一个处理者在无法处理的时候将问题抛给下一个人，直到有一人解决这个问题。不同在于，游戏“击鼓传花”的参与者在鼓声停下来的时候需要得出结果，由解决问题的人（最后手握花球的人）需要将答案报给问题的提出者（击鼓者），而责任链模式中，问题的提出者只需要和链中第一个人有交流即可。

在JDK和扩展包中，以下类中的方法使用了责任链模式：

* `java.util.logging.Logger#log()`
* `javax.servlet.Filter#doFilter()`

## 命令模式(Command) —— 甲方乙方
命令模式是生活中太常见的一个问题了。这就是甲方和乙方的关系，甲方（即调用者）只需要提出需求（即命令（Command）），然后交付给乙方的产品经理（即Invoker），对于每一种需求，大约都有一套既定的流程和实施者，此时经理把需求往需求池中一挂（即触发`command.execute`方法），自会有人（即Receiver来解决）。

在JDK和扩展包中，以下类中的方法使用了命令模式：
* `java.lang.Runnable`
* `javax.swing.Action`

## 解释器模式(Interpreter)
* `java.util.Pattern`
* `java.text.Normalizer`
* `java.text.Format`
* `javax.el.ELResolver`

## 迭代器模式(Iterator) —— 优雅遍历
迭代器模式针对容器提供了一种屏蔽内部细节而提供了一种简单的遍历方式。Java提供的`Iterator`一般就能满足要求，尽量不要自己写迭代器模式。注意接口`Iterable`和`Iterator`的区别。

在JDK和扩展包中，以下类中的方法使用了迭代器模式：

* `java.util.Iterator`
* `java.util.Enumeration`

## 中介者模式(Mediator)
* `java.util.Timer` (所有scheduleXXX()方法)
* `java.util.concurrent.Executor#execute()`
* `java.util.concurrent.ExecutorService `(invokeXXX()和submit()方法)
* `java.util.concurrent.ScheduledExecutorService `(所有scheduleXXX()方法)
* `java.lang.reflect.Method#invoke()`


##  备忘录模式(Memento)
* `java.util.Date`
* `java.io.Serializable`
* `javax.faces.component.StateHolder`

##  观察者模式(Observer)
* `java.util.Observer/java.util.Observable`
* `java.util.EventListener` (所有子类)
* `javax.servlet.http.HttpSessionBindingListene`
* `javax.servlet.http.HttpSessionAttributeListener`
* `javax.faces.event.PhaseListener`

## 状态模式(State)
* `java.util.Iterator`
* `javax.faces.lifecycle.LifeCycle#execute()`

## 策略模式(Strategy) —— if-else代替
定义了算法家族，分别封装起来，让它们之间可以互相替换。简单来说，就是当我们在一次处理过程中，做出很多if-else判断来选择不同的处理方法的时候，就可以采取这种模式。

在JDK和扩展包中，以下类中的方法使用了策略模式：
* `java.util.Comparator#compare()`
* `javax.servlet.http.HttpServlet`
* `javax.servlet.Filter#doFilter()`

## 模板方法模式(Template Method)——模板流程化
模板方法就是在顶层抽象类中将共同步骤抽取出来形成规范，而其中的细节由继承者去实现。这是一个完成由继承关系实现的模式。在JDK中，一个非常典型的例子就是`AQS`，在其中使用模板方法实实现了锁的基本操作，而具体的实现尤其各个继承者来具体实现啊。

在JDK和扩展包中，以下类中的方法使用了模板模式：

* `java.io.InputStream, java.io.OutputStream, java.io.Reader和java.io.Writer`的所有非抽象方法
* `java.util.AbstractList, java.util.AbstractSet和java.util.AbstractMap`的所有非抽象方法
* `javax.servlet.http.HttpServlet#doXXX()`

## 访问者模式(Visitor)
* `javax.lang.model.element.AnnotationValue和AnnotationValueVisitor`
* `javax.lang.model.element.Element和ElementVisitor`
* `javax.lang.model.type.TypeMirror和TypeVisitor`
