# Java初始化加载顺序

之前学了ClassLoader之后，知道经过委托、加载过程之后，类就开始初始化了，一个类中有那么多的变量方法，再加上继承关系，总得有个顺序吧，这篇博文就是来初始化时候的加载顺序了。

在此之前，先要有些概念要弄明白。


## Java三种变量类型

之前我一股脑的把所有的变量都叫“成员变量”，好吧，概念不清楚闹了很多笑话。今天来明确下，下次遇到的时候要说的准确。

Java是纯面向对象的语言，所有的变量方法都需要在某个类的范畴内，这样一来就没有什么全局变量的概念了，而变量在类中的不同位置决定了它属于什么变量

比如下面这个实例：
```java
public class Test {
    //类变量（静态变量）
    private static int count = 0;


    //成员变量（实例变量）
    private int i = 1;

    public static void main(String[] args) {
        //局部变量
        int j = 1;
    }
}
```
Java有三种变量：类变量、成员变量和局部变量，当然它们各自可能都有别名，你怎么习惯怎么叫，但是要区分开。它们最显著的区别就是位置和修饰符的区别啊，真是位置决定人生啊，人生有什么大不同呢，下面来总结一下：

### 类变量（静态变量）
* 类变量也叫静态变量，在类中用关键字`static`声明，但是必须在构造方法和语句块之外。
* 无论一类创建了多少个对象，类只拥有类变量的一份拷贝。那也就是说，类变量被类的所有实例共享了。咦，在某种程序上来说，这是不是就是全局变量的意思了。
* 静态变量除了被声明为常量之外很少被使用。常量是指生命为public/private，final和static类型的变量。常量，顾名思义，一旦初始化时候就不能改变了。
* 静态变量被存储在静态存储区，在JVM中称为”方法区”的地方，这是线程不隔离的地方。经常被声明为常量，很少单独使用static声明变量。
* 静态变量在程序开始的时候创建，在结束的时候销毁。注意了，生命周期是“程序”！
* 与实例变量具有相似的可见性，但是为了对类的使用者可见，大多数静态变量声明为public类型。
* 默认值与实例变量相似，数值型的默认为0，布尔型默认为false，引用型默认为null。变量的值可以在声明时候就定义，也可以在构造方法中指定，还可以在静态语句块中初始化（因为类变量比静态语句块先初始化，所以在静态语句块中可以取到该变量而不会抛出空指针异常）
- 静态变量可以通过: `ClassName.VariableName`的方式访问【其实也可以通过对象访问，但是这样做没有意义，并且会抛出警告】
- 类变量被声明为public static final类型时，类名称必须使用大写字母。如果静态变量不是public和final类型，其命名方法与实例变量以及局部变量的命名方式一致。
- **静态变量不能被非静态方法使用**

### 成员变量（实例变量）
* 实例变量声明在一个类中，但在方法、构造方法和语句块之外。
* 当一个对象被实例化之后，每个实例变量的值就跟着定了。
* 实例变量在对象创建，在对象销毁的时候销毁。注意，成员变量的生命周期是对象。
* 实例变量是跟随对象分配在堆上的。
* 实例变量的值应该至少被一个方法、构造方法或者语句块引用，使得外部能够通过这些方式获取实例变量信息。
* 实例变量可以声明在使用前或者使用后。【？没看懂这句话的意思】
* 访问修饰符可以修饰实例变量。
* 实例变量对类中的方法、构造方法或语句块是可见的，但一般情况下应该把实例变量设为private，通过set和get方法来进行对实例变量的操作。
* 实例变量具有默认值，数值型的默认为0，布尔型的默认为false，引用型的默认为null。变量的值可以在声明时候指定，也可以在构造方法中指定。
* 实例变量可以直接通过变量名访问。但是一般用get方法得到，因为通常将实例变量设置为了private。

### 局部变量
* 局部变量声明在方法、构造方法或者语句块中
* 局部变量在方法、构造方法、或者语句块被执行的时候被创建，当他们执行完之后，变量将被销毁。注意，生命周期是方法或语句块。
* **访问修饰符不能修饰局部变量**
* 局部变量是分配在栈上的。
* 局部变量没有默认值，所以局部变量被声明后，只有初始化后才可以使用。

## 静态代码块和非静态代码块
代码块分为静态和非静态的，写法分别是`static{}`和`{}`，前者会在类加载的时执行且只被执行一次，一般用来初始化静态变量和调用静态方法。为什么只有一次，因为类加载在虚拟机的生命周期中类只被加载一次，类加载的原则是延迟加载，即能少加载就少加载，因为虚拟机的空间有限。

下面是一个测试程序：
```java
public class Test{
    public static void main(String[] args) {
        try {
            Class.forName("JavaLangJarTest.TestClass");
            Class.forName("JavaLangJarTest.TestClass");
            System.out.println(TestClass.x);
        }catch (Exception e){
            e.getStackTrace();
        }
    }
}


class TestClass {
   public static int x = 100;

   public final static int y = 200;

    public TestClass() {
        System.out.println("Test的构造函数执行");
    }
    static {
        System.out.println("static语句块执行");
    }
    public static void display(){
        System.out.println("静态方法被执行");
    }

    public void display_1(){
        System.out.println("实例方法被执行");
    }
}
```
发现只打印出一条`static语句块执行`，说明类只被加载了一次，无论你new了多少个实例。

除此之外，静态代码块还在哪些时机被加载【其实就是为类的加载时机】：
  1. 显示加载的时候，比如用`Class.forName()`， 或`ClassName className;`
  2. 实例化一个对象的时候，比如将上面main方法中内容改成`TestClass testClass = new TestClass();`或`TestClass testClass = Class.forName("JavaLangJarTest.TestClass").newInstance();`。
  3. 调用类的静态方法的时候，比如将上面main方法中内容改成`TestClass.display();`
  4. 调用类的静态变量的时候，比如将上面main方法中内容改成`System.out.println(TestClass.x);`
但是在这个例子中需要注意两点：
    * 调用了类的静态常量的时候，是不会加载了类的，也就不会执行静态代码块了。比如讲上面的main内容改成`System.out.println(TestClass.y);`，发现只打印出了200。这是JVM的规定，当访问类的静态变量时，如果编译器能计算就不会加载类，算不出来的时候会加载了类。
    * 在用Class.forName()的时候我们也可以选择不要加载类，比如将`Class.forName("Test")`改成`Class.forName("Test",false,StaticBlockTest.class.getClassLoader())，`会发现没有任何输出，即类没有被加载，静态代码块也没有被执行。



> 类加载的时机有哪些呢？
1. 第一次创建对象的时要加载类。
2. 调用静态方法时候要加载类，访问静态属性时候会加载类。
3. 加载子类的时候会加载父类。
4. 创建对象引用**不会**加载类。
5. 子类调动父类的静态方法时：
  1. 当子类没有覆盖父类的静态方法时，只加载父类，不加载子类。
  2. 当子类有覆盖父类的静态方法时，既加载父类，也加载子类。
6. 访问静态变量，如果编译器可以计算出常量的值，则不会加载类，例如`public static final int a = 123`；否则会加载类，例如`public static final int a = Math.PI`;


### 构造函数

## 初始化顺序
首先我们看下在非继承关系中，即单个类中，各种变量、方法和代码块的执行顺序，这种例子在网上有很多写的很好的例子，这里直接用下别人的例子：
```java
public class InitialOrderTest {   

    // 静态变量   
    public static String staticField = "类变量";   
    // 变量   
    public String field = "成员变量";   

    // 静态初始化块   
    static {   
        System.out.println(staticField);   
        System.out.println("静态初始化块");   
    }   

    // 初始化块   
    {   
        System.out.println(field);   
        System.out.println("非静态初始化块");   
    }   

    // 构造器   
    public InitialOrderTest() {   
        System.out.println("构造器");   
    }   

    public static void main(String[] args) {   
        new InitialOrderTest();   
    }   
}

```
结果输出：
```java
静态变量
静态初始化块
成员变量
非静态初始化块
构造器
```

如果存在继承关系，那么加载顺序又是怎样的，下面是一个测试例子：
```java
class Parent {   
    // 静态变量   
    public static String p_StaticField = "父类--静态变量";   
    // 变量   
    public String p_Field = "父类--变量";   

    // 静态初始化块   
    static {   
        System.out.println(p_StaticField);   
        System.out.println("父类--静态初始化块");   
    }   

    // 初始化块   
    {   
        System.out.println(p_Field);   
        System.out.println("父类--初始化块");   
    }   

    // 构造器   
    public Parent() {   
        System.out.println("父类--构造器");   
    }   
}   

public class SubClass extends Parent {   
    // 静态变量   
    public static String s_StaticField = "子类--静态变量";   
    // 变量   
    public String s_Field = "子类--变量";   
    // 静态初始化块   
    static {   
        System.out.println(s_StaticField);   
        System.out.println("子类--静态初始化块");   
    }   
    // 初始化块   
    {   
        System.out.println(s_Field);   
        System.out.println("子类--初始化块");   
    }   

    // 构造器   
    public SubClass() {   
        System.out.println("子类--构造器");   
    }   

    // 程序入口   
    public static void main(String[] args) {   
        new SubClass();   
    }   
}  
```
结果打印出：
```java
父类--静态变量
父类--静态初始化块
子类--静态变量
子类--静态初始化块
父类--变量
父类--初始化块
父类--构造器
子类--变量
子类--初始化块
子类--构造器
```
注意到，在实例化一个子类之前，需要将父类和子类全部加载进来，所谓加载进来就是要给给静态变量分配空间并执行静态代码块，然后才开始实例化，因为存在继承关系，所以先实例化父这时候给父类变量分配空间并执行父类的非静态初始化块，然后调用父类的构造函数实例化出一个父类对象，接着才是子类一系列的事情。。。

到了这里，实际上还有一个问题，类加载的时候，静态变量是不是永远都在静态代码块之前呢？成员变量是不是永远都在非静态初始块之前呢？下面是一个测试程序：
```java
public class TestOrder {   
    // 静态变量   
    public static TestA a = new TestA();   

    // 静态初始化块   
    static {   
        System.out.println("静态初始化块");   
    }   

    // 静态变量   
    public static TestB b = new TestB();   

    public static void main(String[] args) {   
        new TestOrder();   
    }   
}   

class TestA {   
    public TestA() {   
        System.out.println("Test--A");   
    }   
}   

class TestB {   
    public TestB() {   
        System.out.println("Test--B");   
    }   
}  
```
打印结果是：
```java
Test--A
静态初始化块
Test--B
```
可见，静态变量和静态代码块，成员变量和非静态代码块，他们之间的执行顺序决定于谁先声明谁先声明。


## 总结
到了这里可以总结一下了，Java程序的初始化一般遵循3个原则（优先级依次递减）：
1. 静态对象（变量）优于非静态对象（变量）初始化。其中，静态对象（变量）只初始化一次，而非静态对象（变量）可能会多次初始化。
2. 父类优先于子类进行初始化。
3. 按照变量的定义顺序进行初始化，即使变量定义散布于方法之中，他们依然在任何方法（包括构造方法）被调用之前被初始化。



参考 :
1. [Java 变量类型](http://www.cnblogs.com/zx3707/p/5662378.html)
2. [Java static作用及加载顺序](http://blog.csdn.net/u010442302/article/details/52052091)
3. [java中static{}语句块详解](http://blog.csdn.net/u010442302/article/details/51365857)
