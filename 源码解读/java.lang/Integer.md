# Integer
终于到了最重要的一个基本数值包装类了，还是先看下结构：
![Integer_structure_1](http://ovn0i3kdg.bkt.clouddn.com/Integer_structure_1.png)
![Integer_structure_2](http://ovn0i3kdg.bkt.clouddn.com/Integer_structure_2.png)

这么多的方法我一屏都截不下来，它之所以重要，并不是因为方法多，而是Byte、Short、Long中很多方法都是直接调用Integer中的方法的，这一点在之前看Byte和Short类的时候应该深有体会。

任务很艰巨，还是挑重点来吧，条条都是重点啊！

### public static Integer valueOf(int i) {...} 和 public static Integer valueOf(String s) throws NumberFormatException{..} 和 public static Integer valueOf(String s, int radix) throws NumberFormatException{..}
三种valueOf方法，当然还是第一种方法最重要了，其定义如下：
```java
public static Integer valueOf(int i) {
   if (i >= IntegerCache.low && i <= IntegerCache.high)
       return IntegerCache.cache[i + (-IntegerCache.low)];
   return new Integer(i);
}
```
又是熟悉的配方：缓存。但是这次的缓存类没有那么简单了。复杂在什么地方，可以看到，之前Byte和Short直接规定了范围是[-128, 127]，但是Integer中没有直接这么做的，而是用内部缓存类的静态属性low和high来限定范围。内部静态缓存类`IntegerCache`的定义如下：
```java

private static class IntegerCache {
   static final int low = -128;
   static final int high;
   static final Integer cache[];

   static {
       // high value may be configured by property
       int h = 127;
       String integerCacheHighPropValue =
           sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
       if (integerCacheHighPropValue != null) {
           try {
               int i = parseInt(integerCacheHighPropValue);
               i = Math.max(i, 127);
               // Maximum array size is Integer.MAX_VALUE
               h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
           } catch( NumberFormatException nfe) {
               // If the property cannot be parsed into an int, ignore it.
           }
       }
       high = h;

       cache = new Integer[(high - low) + 1];
       int j = low;
       for(int k = 0; k < cache.length; k++)
           cache[k] = new Integer(j++);

       // range [-128, 127] must be interned (JLS7 5.1.7)
       assert IntegerCache.high >= 127;
   }

   private IntegerCache() {}
}
```
可见Integer只限定了下界为-128,上界只设定了默认值为127但是可以被修改。如何修改呢？可以看到代码`String integerCacheHighPropValue = sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");`，这是是通过读取VM参数赋值给`integerCacheHighPropValue`，之后取127与该值得最大值赋值给i,但是这个值与-128的距离不能超过Integer能表示的范围。我们可以通过VM参数 `-XX:AutoBoxCacheMax=` 可以配置缓存的最大值。

但是呢，一般来说好像没人这么无聊去设置这个参数，所以一般情况下，Integer的缓存区间还是[-128,127]， 超出这个范围的每次都是返回新new的对象。


### toXxxString(..)
这个真的有好几种转换为String的方法，最厉害的也是最基本的是这种：
```java
public static String toString(int i) {
    if (i == Integer.MIN_VALUE) ①
        return "-2147483648";
    int size = (i < 0) ? stringSize(-i) + 1 : stringSize(i); ②
    char[] buf = new char[size];③
    getChars(i, size, buf);④
    return new String(buf, true);⑥
}
```
① 先判断这个数字是不是最小值，如果是则直接返回字符串，省去了下面的步骤。
② 再判断这个值的长度，怎么做的呢？如果是负数，就将这个数取反，得到`stringSize`方法的结果加上1【因为有一个负号，所以长度加1】，如果是正数则直接返回方法的结果。那么 `stringSize`方法是怎么实现的呢，定义如下：
```java
static int stringSize(int x) {
    for (int i=0; ; i++)
        if (x <= sizeTable[i])
            return i+1;
}

final static int [] sizeTable = { 9, 99, 999, 9999, 99999, 999999, 9999999,
                                    99999999, 999999999, Integer.MAX_VALUE };
````
这个方法巧妙地借助了一个数组，这个数组存储了数组下标加1后表示的位数能够存储的最大正数值。如`sizeTable[0] = 9`，表示位数为0 + 1 = 1的整数的最大值为9，通过比较x的值得到x的位数。厉害了，要是我来设计的话，我可能要用递归的方法每次除以10直到商为0，虽然复杂度一样，但是这样太消耗栈空间了。学习了！

③ 构造该数字长度的字符数组。

④ 依次填充字符数组，这里利用到了一个方法是`getChars`，定义如下：
```java
static void getChars(int i, int index, char[] buf) {
      int q, r;
      int charPos = index;
      char sign = 0;

      if (i < 0) {
          sign = '-';
          i = -i;
      }
      // Generate two digits per iteration
      while (i >= 65536) { ①
          q = i / 100;
      // really: r = i - (q * 100);
          r = i - ((q << 6) + (q << 5) + (q << 2));
          i = q;
          buf [--charPos] = DigitOnes[r];
          buf [--charPos] = DigitTens[r];
      }

      // Fall thru to fast mode for smaller numbers
      // assert(i <= 65536, i);
      for (;;) { ②
          q = (i * 52429) >>> (16+3);
          r = i - ((q << 3) + (q << 1));  // r = i-(q*10) ...
          buf [--charPos] = digits [r];
          i = q;
          if (i == 0) break;
      }
      if (sign != 0) {
          buf [--charPos] = sign;
      }
  }
```
① 当i >= 65536的时候每一次从后往前获取两个最低位数。使用移位操作快速计算出`q*100`（因为`2^6+2^5+2^2=64+32+4=100`)，此时r的值就是后两位。现在需要构造两个数组，要求只根据r的值就能快速得到个位和十位上的字符。怎么设计？

假如现在r = 65，个位上是 5，要得到个位上的5，这时候不管十位是多少个位上一定是5，所以数组DigitOnes的 05，15，25，35，45，55，65，75，85，95位置上都是 5，这样不管是25，还是35 都能得到个位上的5。在来看看如何得到十位上的数，还是65，十位是6，所以DigitTens 的60，61，62，63，64，……69 位置上都是6。
```JAVA
final static char [] DigitTens = {
      '0', '0', '0', '0', '0', '0', '0', '0', '0', '0',
      '1', '1', '1', '1', '1', '1', '1', '1', '1', '1',
      '2', '2', '2', '2', '2', '2', '2', '2', '2', '2',
      '3', '3', '3', '3', '3', '3', '3', '3', '3', '3',
      '4', '4', '4', '4', '4', '4', '4', '4', '4', '4',
      '5', '5', '5', '5', '5', '5', '5', '5', '5', '5',
      '6', '6', '6', '6', '6', '6', '6', '6', '6', '6',
      '7', '7', '7', '7', '7', '7', '7', '7', '7', '7',
      '8', '8', '8', '8', '8', '8', '8', '8', '8', '8',
      '9', '9', '9', '9', '9', '9', '9', '9', '9', '9',
      } ;

  final static char [] DigitOnes = {
      '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
      '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
      '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
      '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
      '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
      '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
      '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
      '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
      '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
      '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
      } ;
````

② 当i < 65535的时候，只取一位的数字的char；`q = (i * 52429) >>> (16+3);`这段代码其实就是`q=i/10` ,其中 `(double)52429/(1<<19)=0.10000038146972656`也就是在int型的时候计算一个数的十分之1的精度是够的，可以看出开发者的这种优化意识是非常强的。同样是这个时候需要借助的是digits数组，其内容如下：
```java
final static char[] digits = {
    '0' , '1' , '2' , '3' , '4' , '5' ,
    '6' , '7' , '8' , '9' , 'a' , 'b' ,
    'c' , 'd' , 'e' , 'f' , 'g' , 'h' ,
    'i' , 'j' , 'k' , 'l' , 'm' , 'n' ,
    'o' , 'p' , 'q' , 'r' , 's' , 't' ,
    'u' , 'v' , 'w' , 'x' , 'y' , 'z'
};
````

> 当时我就觉得奇怪了，digits这个数组中为什么还有字母呢？看上面的过程应该用不到字母才对，后来接着看别的方法的时候才知道这个数组被非10进制的toString方法复用了，大写的服！

最后判断如果这个数是负数，将数组的第一个元素置为‘-’。

> 有一个问题，为什么用65535作为分界线呢，为什么不一直两位两位取值或者干脆一位一位取值呢？或者两位取值一定快，如果奇数位的时候判断一下做下处理不就行了么? 一位一位取值胜在精度。开发者做了两全的办法，大数范围内，求快，小数范围内，求准，至于为什么是65535，我才因为Integer是32位的，按照该概率来讲，平分才有最好的效果，所以取16位。猜测而已。

以上方法输出的是数字十进制表示的字符串，如果需要的不是十进制怎么办？比如想要得到100这个数字的二进制表示。重载了支持指定进制的toString方法，定义如下：
```java
  public static String toString(int i, int radix) {
      if (radix < Character.MIN_RADIX || radix > Character.MAX_RADIX)
          radix = 10;

      /* Use the faster version */
      if (radix == 10) {
          return toString(i);
      }

      char buf[] = new char[33];
      boolean negative = (i < 0);
      int charPos = 32;

      if (!negative) {
          i = -i;
      }

      while (i <= -radix) {
          buf[charPos--] = digits[-(i % radix)];
          i = i / radix;
      }
      buf[charPos] = digits[-i];

      if (negative) {
          buf[--charPos] = '-';
      }

      return new String(buf, charPos, (33 - charPos));
  }

  public String toString() {
    return toString(value);
  }
```
首先限定了最终的进制的值，即小于2大于36的一律作为10进制输出，如果是十进制，则调用无参方法。否则从低位开始计算该进制下的值，这里同样借助了`digits`数组。但是这里有一个问题，为什么要将数字转为负数后再进行除法运算？

其他的一些方法并不是那么常用，这里也不做介绍了，有兴趣看源码吧。


### parseInt
有好几种重载的`paraseInt`方法，区别在于进制、有无符号，最重要的是下面这种：
```java
public static int parseInt(String s, int radix)
                throws NumberFormatException
{
    /*
     * WARNING: This method may be invoked early during VM initialization
     * before IntegerCache is initialized. Care must be taken to not use
     * the valueOf method.
     */

    if (s == null) {
        throw new NumberFormatException("null");
    }

    if (radix < Character.MIN_RADIX) {
        throw new NumberFormatException("radix " + radix +
                                        " less than Character.MIN_RADIX");
    }

    if (radix > Character.MAX_RADIX) {
        throw new NumberFormatException("radix " + radix +
                                        " greater than Character.MAX_RADIX");
    }

    int result = 0;
    boolean negative = false;
    int i = 0, len = s.length();
    int limit = -Integer.MAX_VALUE;
    int multmin;
    int digit;

    if (len > 0) {
        char firstChar = s.charAt(0);
        if (firstChar < '0') { // Possible leading "+" or "-"
            if (firstChar == '-') {
                negative = true;
                limit = Integer.MIN_VALUE;
            } else if (firstChar != '+')
                throw NumberFormatException.forInputString(s);

            if (len == 1) // Cannot have lone "+" or "-"
                throw NumberFormatException.forInputString(s);
            i++;
        }
        multmin = limit / radix;
        while (i < len) {
            // Accumulating negatively avoids surprises near MAX_VALUE
            digit = Character.digit(s.charAt(i++),radix);
            if (digit < 0) {
                throw NumberFormatException.forInputString(s);
            }
            if (result < multmin) {
                throw NumberFormatException.forInputString(s);
            }
            result *= radix;
            if (result < limit + digit) {
                throw NumberFormatException.forInputString(s);
            }
            result -= digit;
        }
    } else {
        throw NumberFormatException.forInputString(s);
    }
    return negative ? result : -result;
}
```
具体的暂时看不懂也不想看，过两天再来看吧。


### public static int hashCode(int value){..}
定义如下：
```java
public static int hashCode(int value) {
    return value;
}
```
简单地返回了value的值而已。

### public static int compare(int x, int y){...}
这次不是返回差值了，定义如下：
```java
public static int compare(int x, int y) {
    return (x < y) ? -1 : ((x == y) ? 0 : 1);
}
```
主调较小的时候返回-1， 相等时候返回0，主调较大的时候返回1。

### public static Integer decode(String nm) throws NumberFormatException
作用是解码字符串转化为正数。这个字符串可以是十进制、十六进制和八进制
```java
public static Integer decode(String nm) throws NumberFormatException {
      int radix = 10;
      int index = 0;
      boolean negative = false;
      Integer result;

      if (nm.length() == 0)
          throw new NumberFormatException("Zero length string");
      char firstChar = nm.charAt(0);
      // Handle sign, if present
      if (firstChar == '-') {
          negative = true;
          index++;
      } else if (firstChar == '+')
          index++;

      // Handle radix specifier, if present
      if (nm.startsWith("0x", index) || nm.startsWith("0X", index)) {
          index += 2;
          radix = 16;
      }
      else if (nm.startsWith("#", index)) {
          index ++;
          radix = 16;
      }
      else if (nm.startsWith("0", index) && nm.length() > 1 + index) {
          index ++;
          radix = 8;
      }

      if (nm.startsWith("-", index) || nm.startsWith("+", index))
          throw new NumberFormatException("Sign character in wrong position");

      try {
          result = Integer.valueOf(nm.substring(index), radix);
          result = negative ? Integer.valueOf(-result.intValue()) : result;
      } catch (NumberFormatException e) {
          // If number is Integer.MIN_VALUE, we'll end up here. The next line
          // handles this case, and causes any genuine format error to be
          // rethrown.
          String constant = negative ? ("-" + nm.substring(index))
                                     : nm.substring(index);
          result = Integer.valueOf(constant, radix);
      }
      return result;
  }
```
重要的是前面判断进制的过程，`0x `和`#`开头的是十六进制，`0`开头的是八进制，默认十进制。
