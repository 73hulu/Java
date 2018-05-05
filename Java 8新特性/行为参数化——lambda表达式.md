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

![lambda + Optional] (https://mmbiz.qpic.cn/mmbiz_jpg/KyXfCrME6UK27qRPAv9NJFPsU1I9dDdzqtSIbs0TY1U8BXANy4mzeSBIOrHJokILQD1xqNurBgMWKJH5GZGG3A/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)




参考
* [lambda表达式有何用](https://mp.weixin.qq.com/s/-PHOc6p-qKJBktle28AUgA)
