# 依赖倒置原则

## 含义
> 依赖倒置原则：
>
> 1.High level modules should not depend upon low level modules. Both should depend upon abstractions.
** 高层模块不应该依赖于底层模块，二者都应该依赖于抽象 **
>
> 2.Abstractions should not depend upon details. Details should depend upon abstractions
** 抽象不应该依赖于细节，细节应该依赖于抽象**

说法还挺复杂，我们来逐词整理一下这里的说法。

首先，“高层模块”和"底层模块"很好理解。每一个逻辑的实现都是有原子逻辑组成的，不可分割的逻辑就是低层逻辑，底层逻辑的组成了高层逻辑。比如A依赖于B，那么A是高层逻辑，B是低层逻辑。

再者，什么是“抽象”。在Java中，抽象是指接口或抽象类，两者都不能被实例化。

最后，什么是“细节”。细节是相对抽象而言的，在Java中就是指接口或者抽象类的具体实现，他们是能够被实例化的。

所以总结下来，在Java中，依赖倒置原则就是：
* 模块之间的依赖是通过接口或抽象发生的，实现类之间不能发生直接的依赖关系。
* 接口或抽象不依赖于实现类。
* 实现类依赖于接口或抽象类。

更加精简的说法是——“面向接口编程”。实例可参考文末链接中的文章，讲的很通俗易懂了。这里需要注意的是，一个正确的做法是：高层类和低层类都需要抽象出各自的接口，让接口之间相互交互。

依赖倒置实现业务解耦，先有接口，后有实现，所以两个类之间有依赖关系，只要制定出两者之间的接口（或抽象类）,就可以实现并行开发，而项目之间的单元测试也可以独立进行。

TDD(Test-Driven Development, 测试驱动开发)开发模式是依赖开发模式的最高级应用。

> “倒置”这个词是相对于“正置”这个词而言的。依赖正置是指类间的依赖是实实在在的

## 依赖的三种写法
对象的依赖传递有三种方式：构造方法传递、setter方法传递和接口传递。

例如，我们定义两个接口:`IDriven`和`ICar`，前者表示“会开车”这种技能，后者表示“（车）能跑”这种技能。定义如下：
```JAVA
public interface IDriven{
  //驾驶汽车
  public void drive();
}

public interface ICar{
  //跑
  public void run();
}
```
两者各自有实现类`Driver`和`Car`，前者需要驾驶汽车，即我们需要传递一个`ICar`的对象给`Driver`，让其进行构造。有三种方式进行参数传递。

### 构造方法传递依赖对象
即在高层实现类的构造函数中设置依赖关系：
```JAVA
public class Driver implements IDriven{
    private ICar car;

    //构造函数注入
    public Driver(ICar icar){
      this.car = icar;
    }

    public void drive(){
      this.car.run();
    }
}
```
### setter方法传递依赖对象
即在低层中设置setter方法来声明依赖关系，同时在高层中实现该方法。上例中`IDriven`的接口应该改写成：
```JAVA
public interface IDriven{
  public void setCar(ICar icar);
  public void drive();
}
```
`ICar`接口写法不变。而`Driver`实现类改写成这样：
```JAVA
public class Driver implements IDriven{
    private ICar car;

    public void setCar(ICar icar){
      this.car = icar;
    }
    public void drive(){
      this.car.run();
    }
}
```

### 接口声明中传递依赖对象
在低层的接口方法声明中设置参数声明依赖关系。上例中`IDriven`接口应该改写成：
```JAVA
public interface IDriven{
  public void drive(Icar car);
}
```
`Icar`的接口声明不变。而`Driver`实现类的定义如下：
```JAVA
public Driver implements IDriven{
  public void drive(Icar car){
    car.run();
  }
}
```
这种写法侵入性太强了，不建议使用。

## 最佳实践
* 每个类都有接口或抽象类，或者兼有之。
* 变量的显示类型尽量是接口抽象类。（并不是绝对的，比如一些工具类，不需要有抽象实现）
* 任何类都不应该从具体类中派生。
* 尽量不要复写基类的方法
* 结合里式替换原则。



参考
* [设计模式六大原则（3）：依赖倒置原则](http://blog.csdn.net/zhengzhb/article/details/7289269)
* [JAVA设计模式之依赖倒转原则](https://www.cnblogs.com/SamFlynn/p/4499698.html)
