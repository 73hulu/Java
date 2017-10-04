# Float
来到浮点数的范畴了，虽然float不是经常用，但是还是要好好看看的，结构如下：

![Float_structure](http://ovn0i3kdg.bkt.clouddn.com/Float_structure.png)

### 常量
Float中有一些特殊的常量定义，如下：
```java
/**
 * A constant holding the positive infinity of type
 * {@code float}. It is equal to the value returned by
 * {@code Float.intBitsToFloat(0x7f800000)}.
 */
public static final float POSITIVE_INFINITY = 1.0f / 0.0f;

/**
 * A constant holding the negative infinity of type
 * {@code float}. It is equal to the value returned by
 * {@code Float.intBitsToFloat(0xff800000)}.
 */
public static final float NEGATIVE_INFINITY = -1.0f / 0.0f;

/**
* A constant holding a Not-a-Number (NaN) value of type
* {@code double}. It is equivalent to the value returned by
* {@code Double.longBitsToDouble(0x7ff8000000000000L)}.
*/
public static final float NaN = 0.0f / 0.0f;

/**
* A constant holding the largest positive finite value of type
* {@code float}, (2-2<sup>-23</sup>)&middot;2<sup>127</sup>.
* It is equal to the hexadecimal floating-point literal
* {@code 0x1.fffffeP+127f} and also equal to
* {@code Float.intBitsToFloat(0x7f7fffff)}.
*/
public static final float MAX_VALUE = 0x1.fffffeP+127f; // 3.4028235e+38f


/**
 * A constant holding the smallest positive normal value of type
 * {@code float}, 2<sup>-126</sup>.  It is equal to the
 * hexadecimal floating-point literal {@code 0x1.0p-126f} and also
 * equal to {@code Float.intBitsToFloat(0x00800000)}.
 *
 * @since 1.6
 */
public static final float MIN_NORMAL = 0x1.0p-126f; // 1.17549435E-38f


/**
* A constant holding the smallest positive nonzero value of type
* {@code float}, 2<sup>-149</sup>. It is equal to the
* hexadecimal floating-point literal {@code 0x0.000002P-126f}
* and also equal to {@code Float.intBitsToFloat(0x1)}.
*/
public static final float MIN_VALUE = 0x0.000002P-126f; // 1.4e-45f


/**
 * Maximum exponent a finite {@code float} variable may have.  It
 * is equal to the value returned by {@code
 * Math.getExponent(Float.MAX_VALUE)}.
 *
 * @since 1.6
 */
public static final int MAX_EXPONENT = 127;


/**
* Minimum exponent a normalized {@code float} variable may have.
* It is equal to the value returned by {@code
* Math.getExponent(Float.MIN_NORMAL)}.
*
* @since 1.6
*/
public static final int MIN_EXPONENT = -126;

```
INFINITY主要是为了解决除数为0的情况。NaN即"NOT A NUMBER"， 表示非数字，它与任何值都不相等， 甚至不等于自己，所以要判断一个数是否NAN需要用isNAN方法。VALUE定义了最大最小值，EXPONENT定义了最大最小指数。下面是一个测试程序：
```java
public static void main(String[] args) {
     float NAN1 = Float.NaN;
     float NAN2 = 0.0f / 0 ;
     System.out.println(Float.isNaN(NAN1)); //true
     System.out.println(Float.isNaN(NAN2)); //true
     System.out.println(NAN1 == NAN1); // false
     System.out.println(NAN1 == NAN2);//false


     double Inf1 = Double.POSITIVE_INFINITY;
     double Inf2 = Double.NEGATIVE_INFINITY;

     float Inf3 = Float.POSITIVE_INFINITY;
     float Inf4 = Float.NEGATIVE_INFINITY;



     System.out.println(Double.isInfinite(Inf1)); //true
     System.out.println(Float.isInfinite(Inf3)); //true
     System.out.println(Inf1 ==  Inf3); //true
     System.out.println(Inf2 == Inf4); //true

     System.out.println(Inf1 * 0); //NaN

     System.out.println(Inf1 + 1); //Infinity
     System.out.println(Inf1 * 0.4); //Infinity
     System.out.println(Inf1 / 0); //Infinity

 }
```
从上面的测试代码中可以得出结论：
1. double或者float判断是不是INFINITY都使用`isInfinite`方法。
2. double中的INFINITY与float中的INFINITY是相等的。
3. INFINITY乘以0得到NAN。
4. INFINITY做除了乘以0以外的任何四则运算，得到的结果仍然是INFINITY。


### public static Double valueOf(double d){...}
浮点型可不好做缓存，所以自动装箱的时候都是返回一个新new的对象。定义如下：
```java
public static Float valueOf(float f) {
    return new Float(f);
}
```

###  public static int hashCode(float value){..}
定义如下：
```java
public static int hashCode(float value) {
   return floatToIntBits(value);
}
```
调用的`floatToIntBits`方法定义如下：
```java
public static int floatToIntBits(float value) {
    int result = floatToRawIntBits(value);
    // Check for NaN based on values of bit fields, maximum
    // exponent and nonzero significand.
    if ( ((result & FloatConsts.EXP_BIT_MASK) ==
          FloatConsts.EXP_BIT_MASK) &&
         (result & FloatConsts.SIGNIF_BIT_MASK) != 0)
        result = 0x7fc00000;
    return result;
}
```
首先调用了`floatToRawIntBits`方法，该方法的定义如下：
```java
public static native int floatToRawIntBits(float value);
```
这个方法返回位代表的浮点数，具体的解释如下：
> 根据IEEE 754浮点“单一格式”位布局，返回指定浮点值的表示，保留非数字（NaN）值。它包括以下要点：
>
> If the argument is positive infinity, the result is 0x7f800000.
> If the argument is negative infinity, the result is 0xff800000.
> 如果参数是NaN，那么结果是整数，表示实际NaN值。从floatToIntBits方法不同，floatToRawIntBits不可折叠所有的位模式将NaN编码为一个“规范”NaN值.

好吧，读了一遍还是没有懂，后面的代码也不是很懂，在网上抄了一段结论：

该方法根据 IEEE 754 浮点“单一格式”位布局，返回指定浮点值的表示形式：第31位（掩码0x80000000选定的位）表示浮点数的符号，第30～23位（掩码0x7f800000选定的位）表示指数，第22～0位（掩码0x007fffff选定的位）表示浮点数的有效位数（有时也称为尾数）。如果参数为正无穷大，则结果为0x7f800000；如果参数为负无穷大，则结果为0xff800000；如果参数为NaN，则结果为0x7fc00000。

在所有情况下，结果都是一个整数，将其赋予intBitsToFloat(int)方法将生成一个浮点值，该浮点值与floatToIntBits的参数相同（而所有NaN值则会生成一个“规范”NaN值）。看下面的例子：
```java
float f = 123.456f;

int i = Float.floatToIntBits(f);

System.out.println(i);

```
将打印出
```java
1123477881
```

###  public boolean equals(Object obj) {...}
浮点数不太好比较大小，怎么办呢，前面说到的函数`floatToIntBits`又用到了：
```java
public boolean equals(Object obj) {
   return (obj instanceof Float)
          && (floatToIntBits(((Float)obj).value) == floatToIntBits(value));
}
```

从上面所有的例子可以看出来，Float这个类最重要的是`floatToIntBits`这个方法，而这个方法调用`floatToRawIntBits`这个本地实现的方法，将浮点数化成int型进行比较，被应用于`equals`、`hashCode`、`max`等方法的中。
