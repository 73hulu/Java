# Math
该类封装了基础的数学计算。结构如下：

![Math](http://ovn0i3kdg.bkt.clouddn.com/Math_1.png)
![Math](http://ovn0i3kdg.bkt.clouddn.com/Math_2.png)

全部都是静态方法，而且大部分的方法都是借助了`StrictMath`中的方法，并且实现并不重要，当成工具类会用即可。

类中有两个字符：
1. static double E
这就是double值，该值是比任何其他更近到e，自然对数的基础上。
2. static double PI
这就是双值，该值是比任何其他更接近到pi，一个圆的圆周比其直径。

下面是该类中常用方法的整理：

| 方法   | 描述    |
| :------------- | :------------- |
| static double abs(double a)  【还有一系列重载方法】     | 此方法返回一个double值的绝对值.       |
| static double acos(double a)   | 此方法返回一个值的反余弦值，返回的角度范围从0.0到pi.  |
| static double asin(double a)   |   此方法返回一个值的反正弦，返回的角度范围在-pi/2到pi/2.|
| static double atan(double a)   |此方法返回一个值的反正切值，返回的角度范围在-pi/2到pi/2.|
|static double atan2(double y, double x)    | 此方法返回角度theta（x，y）从转换的矩形坐标到极坐标（r，θ）.  |
|static double cos(double a)    | 此方法返回一个角的三角余弦.  |
|static double sin(double a)    | 此方法返回一个double值的双曲正弦.  |
|static double tan(double a)    |  此方法返回一个角的三角函数正切值 |
|static double toDegrees(double angrad)    |  这种方法大致相等的角度，以度为单位的角度转换成弧度测量. |
| static double toRadians(double angdeg)   |  此方法转换一个角度，以度为单位大致相等的角弧度测量. |
|static double cbrt(double a)    | 此方法返回一个double值的立方根.  |
|static double pow(double a, double b)    |  此方法返回的第一个参数的值提升到第二个参数的幂 |
|static double exp(double a)    |  此方法返回欧拉数e的一个double值的次幂. |
|  static double	expm1(double x)  |  此方法返回 e^x -1. |
|static double log(double a)    |   此方法返回一个double值的自然对数（以e为底）.|
|static double log10(double a)    | 此方法返回一个double值以10为底.  |
|**static double floor(double a) **   |  此方法返回最大的（最接近正无穷大）double值小于或相等于参数，并相等于一个整数. |  
|**static double ceil(double a) **   |  此方法返回最小的（最接近负无穷大）double值，大于或等于参数，并等于一个整数. |
|**static double rint(double a) **   |  此方法返回的double值，值的参数是最接近的，相等于一个整数. |
|**static long round(double a) **   |  此方法返回的参数最接近的long. |
|**static int round(float a) **   |  此方法返回的参数最接近的整数. |
|static double max(double a, double b)    【还有一系列重载的方法】|  此方法返回两个double值较大的那一个. |
|static double min(double a, double b) 【还有一系列重载的方法】| 此方法返回的两个较小的double值.  |
|**static double random()  **  |  该方法返回一个无符号的double值，大于或等于0.0且小于1.0. |

这些方法在使用的时候要特别注意返回值大部分情况下都是double。特别是加粗的几个方法，每次都记不清楚。这里另外拎出来讲讲

1. Math.floor()
`<=`的概念，向下取整，返回值是double类型。
```java
Math.floor(2.2) = 2.0;  
Math.floor(-2.2) = -3.0;  
Math.floor(2.5) = 2.0;  
Math.floor(-2.5) = -3.0;  
Math.floor(2.7) = 2.0;  
Math.floor(-2.7) = -3.0;
```
2. Math.ceil()
`>=`的概念，向上取整，返回值是double类型。
```java
Math.ceil(2.2) = 3.0;  
Math.ceil(-2.2) = -2.0;  
Math.ceil(2.5) = 3.0;  
Math.ceil(-2.5) = -2.0;  
Math.ceil(2.7) = 3.0;  
Math.ceil(-2.7) = -2.0;
```
3. Math.round()
正宗的四舍五入，返回值是int类型。
参数是负数的时候不太好理解，可以利用公式`Math.round(x) = Math.floor(x + 0.5) `来记忆。
```java
Math.round(2.2) = 2；  
Math.round(-2.2) = -2；  
Math.round(2.5) = 3；  
Math.round(-2.5) = -2；  
Math.round(2.7) = 3；  
Math.round(-2.7) = -3  
```
4. Math.rint()
返回最接近该值的那个整数，注意！如果存在两个这样的整数，则返回其中的**偶数**。
```java
Math.rint(2.2) = 2.0；  
Math.rint(-2.2) = -2.0；  
Math.rint(2.7) = 3.0；  
Math.rint(-2.7) = -3.0；  
Math.rint(2.5) = 2.0；  
Math.rint(-2.5) = -2.0；  
Math.rint(3.5) = 4.0；  
Math.rint(-3.5) = -4.0；
```

5. static double random()
返回一个**[0.0, 1.0)** 之间的double值，注意区间和返回值类型！！！可以利用它来返回一某个区间（左开右闭）的随机浮点数，如果要返回整型，千万别忘记类型转化。
例如，返回[a,b)之间的随机整数， 则`int r = (int)(Math.random() * (b - a) + a)`。

  `random`方法还可以用来生成字符。例如生成a~z之间的任意字符： ` (char)('a' + Math.random() * ('z' - 'a' + 1) )`；随机生成一个ch1-ch2之间的任意字符：`(char)(ch1 + Math.random() * (ch2 - ch1 + 1))`;

  随机数一般可以使用两种方法来实现：一是使用`Math.random()`方法，二是使用`java.util.Random`对象，两者有什么关系？
  我们看下`random`方法的实现：
  ```java
  public static double random() {
      return RandomNumberGeneratorHolder.randomNumberGenerator.nextDouble();
  }
  ```
  追溯`nextDouble`发现它就是`Random`类的实例方法`nextDouble`！

  所以一般情况下，大部分程序员都是用`Math.random`，比较方便。

  > 有关`Random`类，请查看相关博客。

参考
* [java中Math.random()与java.util.random()的区别](http://blog.csdn.net/huangbiao86/article/details/6433964)
