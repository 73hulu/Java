# String高效编程

## 编译器对String的优化
字符串在程序中被频繁使用，所以Java专门针对String类做了很多的性能优化，比如字符串常量池，还有一个措施是编译器对String操作的优化，看下面这段程序：
```java
public static void main(String[] arge) {

        //1
        String str1 = new String("1234");
        String str2 = new String("1234");
        System.out.println("new String()==：" + (str1 == str2));

        //2
        String str3 = "1234";
        String str4 = "1234";
        System.out.println("常量字符串==：" + (str3 == str4));

        //3
        String str5 = "1234";
        String str6 = "12" + "34";
        System.out.println("常量表达式==：" + (str5 == str6));

        //4
        String str7 = "1234";
        String str8 = "12" + 34;
        System.out.println("字符串和数字相加的表达式==：" + (str7 == str8));

        //5
        String str9 = "12true";
        String str10 = "12" + true;
        System.out.println("字符串和Boolen相加表达式==：" + (str9 == str10));

        //6
        final String val = "34";
        String str11 = "1234";
        String str12 = "12" + val;
        System.out.println("字符串和常量相加的表达式==：" + (str11 == str12));

        //7
        String str13 = "1234";
        String str14 = "12" + getVal();
        System.out.println("字符串和函数得来的常量相加表达式==：" + (str13 == str14));
    }
    private static String getVal()
    {
        return "34";
    }

```
运行结果为：
```Java
new String()==：false
常量字符串==：true
常量表达式==：true
字符串和数字相加的表达式==：true
字符串和Boolen相加表达式==：true
字符串和常量相加的表达式==：true
字符串和函数得来的常量相加表达式==：false
```
有了字符串的基础了解，很好理解第一条打印结果和第二句。但是后面几条的结果为什么是这样的呢。看第三句子，编译时编译器发现能够计算出"12"+"34"的值，它是个常量，就按照第二个例子一样处理，最终str5和str6都指向了同一个内存地址。所以==比较结果为true；第四至第六条打印语句也是这个道理。最后一句，译器发现str14值是要调用函数才能计算出来的，是要在运行时才能确定结果的，所以编译器就设置为运行时执行到`String str14="12" + getVal();`时 要重新分配内存空间，导致str13和str1是指向两个不同的内存地址，所以==比较结果为false。

## 少用语法糖，用StringBuilder拼接字符串
Java最甜的语法糖莫过于String类的"+"运算了，常用来拼接字符串，但是如果在大量的拼接需求下，还是建议使用`StringBuilder`来拼接字符串。实际上，“+”的底层实现也是利用了`StringBuilder`(至少在JDK1.6之后是这样的)。看下面这个例子：
```Java
public class Test {
    public void test() {

        String str = "";

        for (int i = 0; i < 1000; i++) {
            str = str + i;
        }
    }
}
```
使用命令`javap -v Test`进行反编译，编译得到下面的字节码：
```java
public void test();
  Code:
   Stack=3, Locals=3, Args_size=1
   0:  ldc #15; //String
   2:  astore_1
   3:  iconst_0
   4:  istore_2
   5:  goto   30
   8:  new #17; //class java/lang/StringBuilder
   11: dup
   12: aload_1
   13: invokestatic  #19; //Method java/lang/String.valueOf:(Ljava/lang/Object;)Ljava/lang/String;
   16: invokespecial #25; //Method java/lang/StringBuilder."<init>":(Ljava/lang/String;)V
   19: iload_2
   20: invokevirtual #28; //Method java/lang/StringBuilder.append:(I)Ljava/lang/StringBuilder;
   23: invokevirtual #32; //Method java/lang/StringBuilder.toString:()Ljava/lang/String;
   26: astore_1
   27: iinc   2, 1
   30: iload_2
   31: sipush 1000
   34: if_icmplt  8
   37: return
  LineNumberTable:
   line 15: 0
   line 17: 3
   line 18: 8
   line 17: 27
   line 20: 37

  LocalVariableTable:
   Start  Length  Slot  Name   Signature
   0      38      0    this       LTest;
   3      35      1    str       Ljava/lang/String;
   5      32      2    i       I

}
```
从上面代码的注释部分可以看出确实是用`StringBuilder`实现的，虽然编译器帮我们做了这些事，但是在实际编码的时候，建议还是使用`StringBuilder`，毕竟这样可以加深你对性能的理解，而且不依赖特定的编译器优化。


### String为什么要设计成不可变的
String是不可变类(immutable)，因为被final。但是，final只能让指向不可变，但是并不能保证内容不可变，所以我们还是有办法改变final对象的内容的，能用这种办法改变String的内容么？

不能，因为String并未提供对内容修改的公共方法，外部根本无法对内部进行修改。

明白了不可变的实现原理，现在来想一想原因，为什么要将String设计成不可变的。这个问题在知乎https://www.zhihu.com/question/31345592 进行了讨论。

不可变的好处，首先是安全。安全可以体现在很多方面。
1. String是常用的类，我们可能经常用于HashMap和HashSet，作为键值。如果是可变的，我们任意去修改值，破坏了HashSet键值的唯一性。所以千万不要用可变类型做HashMap和HashSet键值。
2. 在并发场景下，多个线程同时读一个资源，是不会引发竞态条件的。只有对资源做写操作才有危险。不可变对象不能被写，所以线程安全。


String是几乎每个类都会使用的类，特别是作为Hashmap之类的集合的key值时候，mutable的String有非常大的风险。而且一旦发生，非常难发现。


参考
* [String高效编程优化（Java）](http://blog.csdn.net/bianlians/article/details/51644592)
* [Java编程优化之旅（二） String类型知多少](http://blog.csdn.net/guodongxiaren/article/details/22511427)
* [Java编译器对String的优化](http://www.cnblogs.com/chybin/p/5503885.html)
