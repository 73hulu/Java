# 行为型模式
行为型模式对类或对象怎样交互和怎样分配职责进行描述。有以下11种模式定义：

## 职责链模式(Chain of Responsibility)
* `java.util.logging.Logger#log()`
* `javax.servlet.Filter#doFilter()`

## 命令模式(Command)
* `java.lang.Runnable`
* `javax.swing.Action`

## 解释器模式(Interpreter)
* `java.util.Pattern`
* `java.text.Normalizer`
* `java.text.Format`
* `javax.el.ELResolver`

## 迭代器模式(Iterator)
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

## 策略模式(Strategy)
定义了算法家族，分别封装起来，让它们之间可以互相替换。简单来说，就是当我们在一次处理过程中，做出很多if-else判断来选择不同的处理方法的时候，就可以采取这种模式。

在JDK和扩展包中，以下类中的方法使用了策略模式：
* `java.util.Comparator#compare()`
* `javax.servlet.http.HttpServlet`
* `javax.servlet.Filter#doFilter()`

## 模板方法模式(Template Method)


* `java.io.InputStream, java.io.OutputStream, java.io.Reader和java.io.Writer`的所有非抽象方法
* `java.util.AbstractList, java.util.AbstractSet和java.util.AbstractMap`的所有非抽象方法
* `javax.servlet.http.HttpServlet#doXXX()`

## 访问者模式(Visitor)
* `javax.lang.model.element.AnnotationValue和AnnotationValueVisitor`
* `javax.lang.model.element.Element和ElementVisitor`
* `javax.lang.model.type.TypeMirror和TypeVisitor`
