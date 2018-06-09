# lambda表达式
## 什么是lambda表达式
lambda就是将一块代码赋值给一个“值”。


Java8之前，将下面的代码赋值个一个变量“aBlockOfCode”是不可能的事情：
```java
public void doSomething(String s){
  System.out.println(s);
}
```
而在Java8中可以做到：
```java
aBlockOfCode = (s) - > System.out.println(s);
```

这就是lambda表达式。

会有这样一个问题，变量`aBlockOfCode`的类型是什么？

**Lambda的类型都是一个接口，而lambda表达式本身，就是那段代码，需要是这个接口实现的。** 什么意思呢？就是说我们可以给`aBlockOfCode`加上这个这样一个类型：
```java
@FactionalInterface
interface MyLambdaInterface{
  void doSomething(String s);
}
```
然后`aBlockOfCode`的正确写法应该是：
```
MyLambdaInterface aBlockOfCode = (s) - > System.out.println(s);
```
其中`MyLambdaInterface`只有一个接口，叫做函数式接口。


## lambda表达式有什么作用呢？

最直观的感受是直观简洁。因为上一段代码如果在Java7中，那么它将写成这样
![Java7写上段代码](https://mmbiz.qpic.cn/mmbiz_jpg/KyXfCrME6UK27qRPAv9NJFPsU1I9dDdzSz8z1cicbzOqDC9FzibXeDvo6GRTBW2DlpOfYdgiaYhqiaxibTXZ5Tepu8Q/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

另外，Lambda结合`FunctionalInterface Lib`, `forEach`, `stream()`，`method reference`等新特性可以使代码变的更加简洁！


假设`Person`的定义和`List<Person>`的值都给定。

![Person和List<Person>定义](https://mmbiz.qpic.cn/mmbiz_jpg/KyXfCrME6UK27qRPAv9NJFPsU1I9dDdzwtQurYU5v71qRN0UjydYKawt486Dl6cDDq7MdD4nKfDubTp0dYcf2A/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

现在需要你打印出`guiltyPersons` List里面所有`LastName`以"Z"开头的人的`FirstName`。

用原生的lambda表达式可能需要定义两个函数式接口，定义一个静态函数，调用静态函数并给参数赋值Lambda表达式。

![原生lambda写法](https://mmbiz.qpic.cn/mmbiz_jpg/KyXfCrME6UK27qRPAv9NJFPsU1I9dDdzZnESujYiaEBBnSap9MO6iayZcd2ibXQYOHkKTOqcvaMTwN3Ypq40Im75g/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

还可以更加简单，因为`java.util.function`这个包中已经定义了大量可能用到的函数式接口，他们这一对的接口定义是这样的：

![函数式接口定义](https://mmbiz.qpic.cn/mmbiz_jpg/KyXfCrME6UK27qRPAv9NJFPsU1I9dDdzLNyAn8KsjPWUbpgb9l5ibAp9iaUK7gogicCLRvdIoL1GlDfkd9O8z8ibFg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

所以我们根本不需要自己定义两个函数式接口。做以下步骤的简化：
1. 利用函数式接口简化包：
```java
public static void CheckAndExecute(List<Person> personList,
                                  Predicate<Person> perdicate,
                                  Consumer<Person> consumer){
      for(Person p : personList){
        if(predicate.test(p)) {
          consumer.accept(p);
        }
      }
}
```
然后我们调用的时候这样：
```
CheckAndExecute(guiltyPersons,
                p -> p.getLastName().startWith("Z"),
                P -> System.out.println(p.getFirstName()));
```
下一个步骤是在`CheckAndExecute`中，有点繁琐，我们可以用forEach循环代替for循环，因为forEach循环可以接受一个`Consumer<T>`类型的变量：
```
public static void CheckAndExecute(List<Person> personList,
                                  Predicate<Person> perdicate,
                                  Consumer<Person> consumer){
      personList.forEach(p -> {
        if(predicate.test(p))
          consumer.accept(p);
        })
}
//引用
CheckAndExecute(guiltyPersons,
                p -> p.getLastName().startWith("Z"),
                P -> System.out.println(p.getFirstName()));
```
下一步，由于静态函数其实设置对list进行了一通操作，这里我们可以甩掉静态函数，直接使用stream()特性来完成。`stream()`的几个方法都接受`Predicate<T>`、`Consumer<T>`等参数（参考java.util.Stream）包：
```java
personList.stream()
.filter(p -> p.getLastName().startWith("z"))
.foreach(p->System.out.println(p.getFirstName()));
```
至此已经非常简练了，但是我们改变一下需求，这里要求打印全部信息，那么可以用`Method reference`来简化。`Method reference`, 就是用已经写好的别的Object/Class的method来代替Lambda expression，格式如下：

  ![Method reference](https://mmbiz.qpic.cn/mmbiz_jpg/KyXfCrME6UK27qRPAv9NJFPsU1I9dDdzRaYImmLaVBWT0YMtDeQtlX5zfnmdMSsYLbXqwt6FQ924dfBUcShB6w/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

  所以上面的代码可以变成这样：
  ```
  personList.stream()
  .filter(p -> p.getLastName().startWith("Z"))
  .forEach(System.out::println)
  ```

  这基本上就是能写的最简洁的版本了。
另外，lambda还可以配合`Optional<T>`可以使Java对于null的处理变的异常优雅.
这里假设我们有一个person object，以及一个person object的Optional wrapper:
```java
Person person = foAndGetAFunckingPerson();
Optional<Person> personOpt = Optional.ofNull(person);
```
如果不是用`Optional<T>`，将写成下面这种也别繁琐的写法

  ![Without Optional<T>](https://mmbiz.qpic.cn/mmbiz_jpg/KyXfCrME6UK27qRPAv9NJFPsU1I9dDdzqtSIbs0TY1U8BXANy4mzeSBIOrHJokILQD1xqNurBgMWKJH5GZGG3A/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

  lambda + Optional的其他对于null值的检查如下：

  ![lambda + Optional](https://mmbiz.qpic.cn/mmbiz_jpg/KyXfCrME6UK27qRPAv9NJFPsU1I9dDdzqtSIbs0TY1U8BXANy4mzeSBIOrHJokILQD1xqNurBgMWKJH5GZGG3A/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)



## Lambda的基本语法
Lambda的基本语法如下：
```Java
(params) -> expression
```
或
```Java
(params) -> { statements; }
```
注意“表达式(expression)”和“语句(statements)”的区别

函数式接口：只定义一个抽象方法的接口，注意，这里是指只含有一个本身特有的抽象方法，不包含从父接口继承得到的。

## 方法引用
方法引用就是用已经实现的方法实现来创建lambda表达式，其本质上还是lambda表达式， 只是一种lambda的快捷写法。

注意，这里的方法是“引用”，而不是“调用”，所以只用说明而已。比如`Apple::getWeight`就是在“引用”`Apple`中的`getWeight`方法，但并不是在调用，所以不用加括号。它实际是`(Apple a) -> a.getWeight()`的简略写法。

有三种方法可以作为方法引用：
1. 指向静态方法的方法引用。如Integer的parseInt可以写成Integer::parseInt
2. 指向任意类型实例方法的方法引用。如`String::length()`
3. 指向现有对象的实例方法。如有一个对象`transaction`，它是`Transaction`类型的对象，支持实例方法`getValue`，那么就可以写成`transaction::getValue`

特别的，对于构造函数，可以使用`ClassName::new`，假如一个构造函数没有参数，那么它适合做`Supplier`签名`() -> Apple`：
```Java
Supplier<Apple> c1 = Apple::new; // 相当于Supplier<Apple> c1 = () -> new Apple()
Apple a = c1.get();
```
如果构造函数有参数，那么函数签名就适合于`Function<Integer, Apple>`：
```Java
Function<Integer, Apple> c2 = Apple::new; // 相当于Function<Integer, Apple> c2 = (weight) -> new Apple(weight)
Apple a = c2.apply(100);
```

如果有两个参数，那么函数签名就适合于`BiFunction`：
```Java
BiFunction<Integer, String, Apple> c3 = Apple::new;
Apple a3 = c3.apple(110, "red");
```

注意到，**不将构造函数实例化却能够引用它**，这个特性我们可以引用于“工厂方法”模式中：
```Java
public interface Builder {
  void add(WeaponType name, Supplier<Weapon> supplier);
}

public interface WeaponFactory {

  /**
   * Creates an instance of the given type.
   * @param name representing enum of an object type to be created.
   * @return new instance of a requested class implementing {@link Weapon} interface.
   */
  Weapon create(WeaponType name);

  /**
   * Creates factory - placeholder for specified {@link Builder}s.
   * @param consumer for the new builder to the factory.
   * @return factory with specified {@link Builder}s
   */
  static WeaponFactory factory(Consumer<Builder> consumer) {
    Map<WeaponType, Supplier<Weapon>> map = new HashMap<>();
    consumer.accept(map::put);
    return name -> map.get(name).get();
  }
}

public class App{
  public static void main(String[] args) {
    WeaponFactory factory = WeaponFactory.factory(builder -> {
      builder.add(WeaponType.SWORD, Sword::new);
      builder.add(WeaponType.AXE, Axe::new);
      builder.add(WeaponType.SPEAR, Spear::new);
      builder.add(WeaponType.BOW, Bow::new);
    });
    Weapon axe = factory.create(WeaponType.AXE);
    LOGGER.info(axe.toString());
  }
}
```

## 函数式接口
Java8中提供了很多内置的函数式接口，例如
* 断言型接口`Predicate`
* 函数型接口`Function`
* 消费型接口`Consumer`
* 供应型接口`Supplier`
* 比较型接口`Comparator`
* 流接口`Stream`： 其中包含了很多子函数是接口


参考
* [lambda表达式有何用](https://mp.weixin.qq.com/s/-PHOc6p-qKJBktle28AUgA)
