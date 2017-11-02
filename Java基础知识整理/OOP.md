# OOP

面向对象的三大特征是:
1. 封装：解决了数据的安全性问题
2. 继承：解决了代码的重用问题
3. 多态：解决了程序扩展问题

> 三种特征要牢记！


### 封装
封装性，从字面来看就是把一些信息包装起来，专业一点讲，封装是指利用抽象数据类型将数据和基本数据的操作都封装到一起，使其构成一个不可分割的独立实体。这个独立的实体被叫做“类”。

Java中一个`.java`文件中可以有多个类，但是只能有一个用`public`文件修饰的类，并且需要与文件名同名。一个类中可以包含另外一个类，这就是所谓的内部类和外部类。Java是纯面向对象的语言，即没有全局变量和全局方法，所有方法和变量都在类中。

封装的目的在于隐藏。数据被保护在抽象数据类型的内部，**尽可能地隐藏内部的细节，只保留一些对外接口使之与外部发生联系**。系统的其他对象只能通过包裹在数据外面的已经授权的操作来与这个封装的对象进行交流和交互。

注意上面高亮的这句话，其中“内部细节”，我们指的是“属性”，而“对外接口”是指方法。我们需要保证“属性”的隐秘性和“方法”的开放性。那么，怎么保证呢？用可访问性修饰符。Java中有四种可访问性修饰符：private、protected、（default）和private，访问权限依次减少。这四个修饰符都可以用来修饰属性和方法。一般地，我们将属性都设置为private，那么外部对象如何访问这个属性？每个属性都有get和set方法，而这些方法都是开放的。

属性，有时候也叫做变量，可以有三种分类：实例变量、类变量和局部变量。它们分别有不同的生命周期和作用范围。其中实例变量属于一个对象。什么是对象？对象就是一个类的实例化。比如“人类”是一个类，而“Bob”这个人就是这个类的一个实例。外部访问实例变量必须基于特定的实例。如何得到这个实例？使用构造函数。

构造函数是特殊的方法。特殊在哪里？
①方法名必须和类名相同

②无返回值，void也不行。如果有返回值，虽然同名，那么它也只是一个普通的方法而不是一个构造函数。

③总是伴随着new操作一起调用，在实例化的时候被（系统）自动调用，且只运行一次，而普通方法是在程序执行的时候被（开发人员)调用，且可以被调用很多次。

④不能被继承，因此不能被覆盖，但是能够被重载。重载的构造函数之间的调用关键字是`this`。

⑤如果开发人员没有显式地给出一个构造函数（无论有没有参数），系统会自动生成一个无参构造函数（可访问性修饰符与类的可访问性修饰符一样）。但是！！！如果定义了一个构造函数（无论又没有参数），系统不会生成一个构造函数。所以开发人员最好每次都**指定一个无参的构造函数**，以免造成严重后果。

⑥继承关系中，子类可以通过`super`来调用父类的构造方法。注意，当父类没有提供无参构造函数时，子类的构造函数就必须**显式**调用父类的构造函数。如果父类提供了无参构造函数，那么此时子类就不必要显式调用构造函数（此时系统默认调用了）。初始化时，先调用父类的构造函数，然后再调用子类的构造函数。  

对于类的实例化，先大致这么多。

### 继承
继承是指从已经有的类中派生出新的类，关键字是`extends`。子类继承了父类，子类就可以继承父类的属性和方法。

但是远远没有那么简单，继承是有条件的。

首先，不是什么类都可以被继承的。如果**类A对类B没有可见性，或者类A是终类，即被final修饰**，那么B就不能继承A。

再者，并不是所有的属性和方法都可以被继承的。子类只能继承父类**非私有，即public和protected，且非final**的属性和方法。另外，构造方法是万万不能继承的。

最后，经过上面两个条件的筛选之后，得到的属性和方法不一定能被继承。为什么呢？因为存在隐藏和方法覆盖。

##### 方法覆盖
方法覆盖也就是我们所说的“重写”。这里的方法是指：可能被继承（还记得能被继承的条件么）的**实例方法**，注意，不包含静态方法！！！什么情况下能构成重写呢？子类定义的方法需要满足“**两同一大一小**”原则：
①方法名相同，参数类型相同，即方法签名相同。
②子类访问权限应该大于父类方法的访问权限。比如父类方法的可见性是protected，那么子类必须是public。如果父类是public，那么子类必须是public。
③子类抛出的异常应该小于父类方法抛出的异常。

如果子类中的某个方法符合上面两个条件，那么子类的方法将会覆盖父类中的方法，而不会继承这个方法。但是如果不满足的话，那个子类和父类中的两个方法是完全没有关系的两个方法，子类可以继承父类方法。

例如B继承了A，A的实例方法test()在B中被重写了。C是B的子类，那么C就无法通过super().super().test()来访问A的test()方法，因为已经被覆盖了。

例如下面这段程序的执行结果是什么？
```java
public class Test {

  static class Super{
    public int f(){
      return 1;
    }
  }
  public  static class SubSuper extends Super{
    public float f(){
      return 2f;
    }
  }
  public static void main(String[] args) {
    Super s = new SubSuper();
    s.f();

  }
}
```
这段程序会产生编译错误。因为首先`SubSuper`继承自`Super`类，其中`SubSuper`的`f`方法签名与父类的`f`方法签名不一致，所以不能覆盖，`SubSuper`类将集成父类的`f`方法，这时候两者共存，但是不能构成重载（因为参数列表一直，不能将返回值作为依据），所以这时候会有编译错误。

> 方法签名是指方法名和返回类型，不包含返回类型。方法签名能唯一确定一个方法
> 注意“重写”和“重载”的区别：重写要求方法签名完全一致，重载要求方法名一样，但是参数列表不一样。


#### 隐藏
“隐藏”针对的对象是能够被继承的**数据域（包括实例变量和类变量）**和**静态方法**。如果子类中存在同名的数据域或静态方法，那么父类中的数据域或静态方法将被隐藏。

所以隐藏和覆盖有什么区别呢？当然有区别！“覆盖”是原来的没了，消失了，不能访问了。而隐藏说明其实还存在，还有方法可以访问到，怎么访问？可以通过super()调用或者父类类型的引用变量来访问。

诶，记住“父类类型的引用可以用来范围被隐藏的变量和静态方法”，记住这句话。这时候经典的考题来了。看看下面这个例子:
```java

//Animal.java
public class Animal {
    public String news = "Animal's news";
    public  static String message = "Animal's message";

    public static String smile() {
        return "smile from Animal";
    }
    public String getNews() {
        return news;
    }
    public String getMessage(){
        return message;
    }
}


//Tiger.java
public class Tiger extends Animal{
    public String news = "Tiger's news";
    public static String message = "Tiger's message";
    public String somethingNew = "Something new from tiger";

    public static String smile() {
        return "smile form Tiger";
    }
    public String getNews() {
        return news;
    }
    public String print(){
        return "The news is " + news + " and the message is " + message;
    }
}

//Test.java
public class Test {
    public static void main(String[] args) {
        Animal x = new Tiger();

        System.out.println("(1) x.news is " + x.news); //(1) x.news is Animal's news
        System.out.println("(2) x.message is " + x.message); //(2) x.message is Animal's message
//		System.out.println("(3) x.somethingNew is " + x.somethingNews);


        System.out.println("(3)x.smile() is " + x.smile()); //(3)x.smile() is smile from Animal
        System.out.println("(4)((Tiger)x).smile() is " + ((Tiger)x).smile()); //(4)((Tiger)x).smile() is smile form Tiger
        System.out.println("(5) x.getNews() is " + x.getNews()); //(5) x.getNews() is Tiger's news
        System.out.println("(6) x.getMessage() is " + x.getMessage());//(6) x.getMessage() is Animal's message
//		System.out.println("(7) x.print() is " + x.print());
    }
}
```
程序运行结果已经标注在上面，为什么会是这样的结果。来分析一下。

首先`x`是一个引用，`Animal`类型的引用，而实际的类是`Tiger`，所以它被声明为父类引用，实际运行的类型是子类。前面我们说到方法覆盖和隐藏，这里考的就是这个知识点。

首先`news`属性是父类`Animal`的实例属性，而子类中有同名属性，符合继承条件，所以子类隐藏了父类的这个实例属性，但是x是父类类型的引用，所以`x.news`访问的是父类的实例变量。

接着`message`是父类的静态属性，而子类中有同名属性，符合继承条件，所以子类隐藏了父类的这个静态属性，没有被继承，但是x是父类类型的引用，所以`x.message`访问的是父类的类变量。

接着`smile`是父类的静态方法，而子类中有同名的静态方法，符合继承条件，所以子类隐藏了父类的这个静态方法，没有被继承，但是x是父类类型的引用，所以`x.smile`访问的是父类的静态方法。

接着`getNews`是父类的实例方法， 符合继承条件，而子类中有同名的静态方法， 所以子类的方法覆盖了父类的方法，注意是覆盖，没有被继承，所以这时候`x.getNews`方法的是子类的实例方法。

最后`getMessage`是父类的实例方法，符合继承条件，而子类中不具有同名方法，所以这个方法是实打实的被继承了，这时候`x.getMessage`访问的是父类的实例方法，而其中访问的是父类的`message`类变量。


实际上，上面这个例子就是一个多态的实现。当我们看到类似于`Animal x = new Tiger()`这种初始化方式的时候，即声明类型和运行类型不一样的时候，要特别小心，记住以下结论：
* 使用父类类型的引用来访问**实例方法**的时候，要注意方法覆盖的可能性，变量所引用对象的是实际类在运行时决定使用该方法的哪种实现。
* 使用父类类型的引用来访问数据域或静态方法的时候，引用变量所声明的类型在编译的时候就决定使用哪个数据域或静态方法。


话题重新回到继承。Java不支持多继承，但是可以用接口和内部类来实现多继承的“效果”，这种方式叫做组合。使用接口与组合的方式比采用继承的方式具有更好的可扩展性，这一点在设计模式中最能体现。两种方式的不同在这里就不多说了。

在继承关系中，经常考察this和super关键字的区别。
this指代当前类的实例对象（注意是对象，这也就是为什么类方法中不能有this关键字的原因），super指代的是父类的实例对象。this可以用来访问本类中的实例变量，而多个重载的构造函数之间的调用是通过`this()`的形式进行的。子类可以通过`super()`形式对父类的构造方法进行调用，当存在隐藏和方法覆盖的情况时，可以通过`super`关键字进行调用，类似于`this`。

特别注意的是，子类的构造方法可能会由系统隐式调用父类的构造方法，如果开发人员自己显式调用，即用`super`调用，那么`super()`必须要放在构造方法的第一句，否则编译报错。

### 多态
多态是OOP中重要的性质，表示的是同一个操作作用于不同的对象时，会有不同的表现，比如同样是执行“+”操作，对两个整数来说就是数字加和，对于字符串就是字符串连接。在Java中，多态主要有两种表现方式：
1. 方法重载——编译时多态
即overload。在**同一个类**中有很多同名但参数列表不同的方法。该使用那个方法是在**编译**的时候就决定了的，是一种编译时的多态，可以看做是同一个类的方法的多态性。
2. 方法覆盖——运行时多态
即Override。Java语言具有下面这个特点：
 > 为父类对象设计的任何代码可应用于子类。“每一个子类的实例都是父类的实例，反之不成立”。

所以一个基类的引用变量不仅可以指向基类的实例对象，也可以指向子类的实例对象（就像上面例子中的`Animal`类型引用变量）。它所调用的方法在运行期间才动态绑定（绑定是指将一个方法调用和一个方法主题连接到一起），就是引用变量所指向的具体实例对象的方法，也就是内存里正在运行的那个对象的方法，而不是引用变量的类型中定义的方法。因为是运行时才决定的，所以也叫做运行时多态。
> 注意方法才有多态的概念，变量是没有多态的概念的。

OOP的题目除了上面的知识点之外，还经常和初始化顺序一起考察。例如：
```java
public class Baset {

    private String baseName = "base";
    public static String baseMessage = "baseMessage";
    //构造方法
    public  Baset() {
        // TODO Auto-generated constructor stub
        callName();
    }

    //成员方法
    public void callName() {
        System.out.println("baseName : " + baseName);
        System.out.println("baseMessage : " + baseMessage);
    }

    //内部静态类
    static class Sub extends Baset{
        //静态字段
        private String subName = "sub";
        public static String subMessage = "subMessage";

        public Sub(){
            callName();
        }
        public void callName() {
            System.out.println("subName：" + subName);
            System.out.println("subMessage：" + subMessage);

        }

    }
    public static void main(String[] args) {
        Baset base = new Sub();
    }
}
```
上面程序的输出结果是：
```
subName：null
subMessage：subMessage
subName：sub
subMessage：subMessage
```
让我们来分析一下这个过程。
首先，当`Baset`这个类被编译通知后，会在相应目录下生成两个.class文件，一个是Baset.class，另一个是Baset$Sub.class文件，这时候类加载器将这个两个.class文件加载到内存。

接着，main方法中执行 `Baset base = new Sub()`方法，发现`Sub`继承了`Baset`，类，所以会先初始化其静态成员，此题中只有`baseMessage`这个静态变量，然后执行子类的静态成员初始化，同样是`subMessage`，注意，此时子类的静态是隐藏了父类的静态变量，这种隐藏是编译的时候就决定了的。然后执行Sub的构造函数，构造函数实际上是省略了对父类构造方法的调用，实际上`Sub`的完整的构造方式应该是这样的：
```java
public Sub(){
    super();//调用父类的构造方法
    subName = "sub"; //子类实例属性的初始化
    callName();
}
```
而父类的构造方法实际上是这样的：
```java
public  Baset() {
    // TODO Auto-generated constructor stub
    baseName = "base";// 父类实例变量的输出化
    callName();
}
```
所以在调用`Sub`构造函数的时候，首先调用的`Baset`的构造函数，初始化了`baseName`这个实例变量，然后执行`callName`方法，注意了，此时调用的是子类的callName方法， 因为存在方法覆盖。而子类的callName方法中将打印`subName`和`subMessage`。此时，`subName`还是默认值null，而`subMessage`已经被初始化为`subMessage`。之后回到`Sub`类的构造函数，进行实例变量`subName`初始化，接着调用callName方法，此时可以打印出`subName`和`subMessage`的值。



有一道题目是如何取得父类的类名。

我们知道利用反射可以取得类名，方法是`getClass().getName()`，那么是不是可以利用`super.getClass().getName()`来取得父类的类名呢？就像下面这样：
```java

//A.java
class A{}

//Test.java
public class Test extends A {

  public void test(){
      System.out.println(super.getClass().getName());
  }

  public static void main(String[] args) {
      new Test().test(); //Test
  }
}
```
打印的结果是Test，为什么不是A。原因在于`getClass`这个方法在Object中被定义成final和native，子类无法重写，而所有类都是Object的子类，而Object的getClass方法的解释是返回运行时候的类，所以上面的程序中getClass会一直返回运行的类，即`Test`，所以得不到其父类A的类名。那么有其他办法么？有，通过反射机制`getSuperclass`。
```java
public class Test extends A {

    public void test(){
        System.out.println(super.getClass().getSuperclass().getName());
    }

    public static void main(String[] args) {
        new Test().test(); //A
    }
}
```
所以下一步就是学习反射机制了。


> OOP这部分要好好理解，有很多题目要刷才能查漏补缺
