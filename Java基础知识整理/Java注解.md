# Java注解
Java注解(Annotation)是JDK1.5引入的功能，也叫做元数据，增加代码的可读性，给谁读呢？人和机器。和类、接口、枚举在同一个层次。它可以声明在包、类、方法、局部变量、方法参数之前，用来对它们进行说明，注释。

Annotation的作用包括：
1. 编写文档：通过代码里的元数据生成文档。【生成doc】
2. 代码分析：通过代码里标识的元数据对代码进行分析【使用反射】
3. 编译检查：通过代码里标识的元数据让编译器能够实现基本的编译检查【`Override`】

Java api中注解相关的内容都在包`java.lang.annotation`中，其结构如下：
![java.lang.annotaion](http://ovn0i3kdg.bkt.clouddn.com/java.lang.annotaion.png)
具体内容可以参考源码解读。

### Java元注解
什么是“元注解”呢？就是那些可以用来修饰其他注释的注释，即元注解的`@Target`的value值一定包含`ElementType.ANNOTATION_TYPE`。

Java提供了四种元注解:`@Documented`、`@Target`、`@Retention`和`@Inherited`。具体的作用参考源码解读中相关内容。

下面就来学习一下如何自定义一个注解。


自定义注解并应用的过程分为下面三个过程：定义注解类、应用注解类、反射，如下图：

![注解应用结构](http://ovn0i3kdg.bkt.clouddn.com/%E6%B3%A8%E8%A7%A3%E7%9A%84%E5%BA%94%E7%94%A8%E7%BB%93%E6%9E%84%E5%9B%BE.jpg)

例如：
```java

@Documented
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
public @interface MyAnnotaion{

}
```
上面代码定义了名为`MyAnnotaion`的注解，注解应用于类、接口和方法，并且在整个运行期间内都存在。


接着将这个注解应用于某个类。如：
```java
@MyAnnotaion
public Class AnnotationnUse{

}
```

我们在main方法中测试以下这个类都应用了哪些注解：
```java
package cn.gacl.annotation;
@MyAnnotation
//这里是将新创建好的注解类MyAnnotation标记到AnnotaionTest类上
public class AnnotationUse {
    public static void main(String[] args) {
        // 这里是检查Annotation类是否有注解，这里需要使用反射才能完成对Annotation类的检查
        if (AnnotationUse.class.isAnnotationPresent(MyAnnotation.class)) {
            /*
             * MyAnnotation是一个类，这个类的实例对象annotation是通过反射得到的，这个实例对象是如何创建的呢？
             * 一旦在某个类上使用了@MyAnnotation，那么这个MyAnnotation类的实例对象annotation就会被创建出来了
             * 假设很多人考驾照，教练在有些学员身上贴一些绿牌子、黄牌子，贴绿牌子的表示送礼送得比较多的，
             * 贴黄牌子的学员表示送礼送得比较少的，不贴牌子的学员表示没有送过礼的，通过这个牌子就可以标识出不同的学员
             * 教官在考核时一看，哦，这个学员是有牌子的，是送过礼给他的，优先让有牌子的学员过，此时这个牌子就是一个注解
             * 一个牌子就是一个注解的实例对象，实实在在存在的牌子就是一个实实在在的注解对象，把牌子拿下来(去掉注解)注解对象就不存在了
             */
            MyAnnotation annotation = (MyAnnotation) AnnotationUse.class
                    .getAnnotation(MyAnnotation.class);
            System.out.println(annotation);// 打印MyAnnotation对象，这里输出的结果为：@cn.itcast.day2.MyAnnotation()
        }
    }
}
```
程序执行结果是：`@JavaLangJarTest.anotationTest.MyAnotation()
`

注解可以看做是特殊的类，既然是类，那么就可以为类添加属性和方法。
* 添加属性
语法为：`类型 属性名()`。这个方法跟我们平常定义属性的办法不同，这种写法来看，注解更像是一种特殊的接口，注解中属性的定义和接口中方法的定义一样，而应用了注解的类可以认为是实现了这个特殊的接口。

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.METHOD})
public @interface MyAnotation {

    // 定义基本属性
    // @return
    String color();
}

@MyAnotation(color = "red") //应用注解，并定义属性，如果定义color，编译会报错，因为属性的值是必须的，有没有办法可以不写呢？可以，指定默认值
public class AnotationUse {

    public static void main(String[] args) {
        if (AnotationUse.class.isAnnotationPresent(MyAnotation.class)){

            MyAnotation annotation = (MyAnotation) AnotationUse.class.getAnnotation(MyAnotation.class);
            //用反射方式获得注解对应的实例对象后，在通过该对象调用属性对应的方法
            System.out.println(annotation.color());
        }
    }
}
```
* 为属性值指定默认值
属性如果没有默认值，那么在应用注解的时候，必须指定属性的值，否则编译不通过。指定默认值的语法是:`类型 属性值() default 默认值`。
```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.METHOD})
public @interface MyAnotation {

    // 定义基本属性
    // @return
    String color() default "blue";
}
@MyAnotation //注解里面指定了默认值，所以这里就不用定义color了
public class AnotationUse {

    public static void main(String[] args) {
        if (AnotationUse.class.isAnnotationPresent(MyAnotation.class)){

            MyAnotation annotation = (MyAnotation) AnotationUse.class.getAnnotation(MyAnotation.class);
            System.out.println(annotation.color()); //blue
        }
    }
}
```
* value属性
如果一个注解中有一个名称为value的属性，且你只想设置value属性(即其他属性都采用默认值或者你只有一个value属性)，那么可以省略掉“value=”部分。例如：`@SuppressWarnings("deprecation")`。
```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.METHOD})
public @interface MyAnotation {

    String color() default "blue";

    String value();
}

@MyAnotation("aaa") //这里可以省略"value = "
public class AnotationUse {

    public static void main(String[] args) {
        if (AnotationUse.class.isAnnotationPresent(MyAnotation.class)){

            MyAnotation annotation = (MyAnotation) AnotationUse.class.getAnnotation(MyAnotation.class);
            System.out.println(annotation.value()); //aaa
        }
    }
}
```

* 数组类型的属性
  * 增加数组类型属性： `int[] arrayAttr() default {1, 2, 3}`。
  * 应用数组类型属性：`@MyAnnotation(arrayAttr = {2, 4, 5})`。
  * 如果数组属性只有一个值，这时候属性部分可以省略大括号。`@MyAnnotation(arrayAttr = 2)`，表示数组只有一个值，值为2。
* 枚举类型的属性
  - 增加枚举类型的类型的属性：`EumTrafficLamp lamp() default EnumTrafficLamp.RED`.
  - 应用枚举类型的属性：`@MyAnnotation(lamp=EumTrafficLamp.GREEN)`
- 注解类型的属性
```java

//MetaAnnotation 注解为元注解
public @interface MetaAnnotation{
  String value(); //设置有一个唯一的属性value
}
```
为注解添加一个注解类型的属性，并制定注解类型的缺省值：`MetaAnnotation annotaionArttr() default @MetaAnnotation("xdp")`。

综合的例子参考https://www.cnblogs.com/xdp-gacl/p/3622275.html



### Java内建注解
Java提供了三种内建注解。
1. `@Override`：当我们想要复写父类中的方法时，我们需要使用该注解去告知编译器我们想要复写这个方法。这样一来当父类中的方法移除或者发生更改时编译器将提示错误信息。源码如下：
```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {
}
```
2. `@Deprecated`：当我们希望编译器知道某一方法不建议使用时，我们应该使用这个注解。Java在javadoc中推荐使用该注解，我们应该提供为什么该方法不推荐使用以及替代的方法。源码如下：
```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(value={CONSTRUCTOR, FIELD, LOCAL_VARIABLE, METHOD, PACKAGE, PARAMETER, TYPE})
public @interface Deprecated {
}
```
3. `@SuppressWarnings`：这个仅仅是告诉编译器忽略特定的警告信息，例如在泛型中使用原生数据类型。它的保留策略是SOURCE（译者注：在源文件中有效）并且被编译器丢弃。源码如下：
```java
@Target({TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE})
@Retention(RetentionPolicy.SOURCE)
public @interface SuppressWarnings {
    String[] value();
}
```
内部有一个String数组，主要接受值有：
  * `deprecation`：使用了不赞成使用的类或方法时的警告；
  * `unchecked`：执行了未检查的转换时的警告，例如当使用集合时没有用泛型 (Generics) 来指定集合保存的类型;
  * `fallthrough`：当 Switch 程序块直接通往下一种情况而没有 Break 时的警告;
  * `path`：在类路径、源文件路径等中有不存在的路径时的警告;
  * `serial`：当在可序列化的类上缺少 serialVersionUID 定义时的警告;
  * `finally`：任何 finally 子句不能正常完成时的警告;
  * `all`：关于以上所有情况的警告。

参考：
* [深入理解Java注解类型(@Annotation)](http://blog.csdn.net/javazejian/article/details/71860633?t=1495827334875)
* [Java注解教程及自定义注解](http://www.importnew.com/17413.html)
* [Java中三种简单注解介绍和代码实例](http://www.jb51.net/article/55370.htm)
* [Java基础加强总结(一)——注解(Annotation)](https://www.cnblogs.com/xdp-gacl/p/3622275.html)
