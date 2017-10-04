# Long
看过了最重要的Integer之后，Long就不怕了，其结构如下：
![Long_structure_2](http://ovn0i3kdg.bkt.clouddn.com/Long_structure_1.png)
![Long_structure_2](http://ovn0i3kdg.bkt.clouddn.com/Long_Structure_2.png)

看着虽然很多，但是很多内容直接调用了Integer的方法或者一样的套路，还是挑不同的记录。

###  public static Long valueOf(long l) {...}
自动装箱方法，定义如下：
```java
public static Long valueOf(long l) {
    final int offset = 128;
    if (l >= -128 && l <= 127) { // will cache
        return LongCache.cache[(int)l + offset];
    }
    return new Long(l);
}
```
没有Integer那么婆婆妈妈的，直接约定缓存范围是[-128, 127]，超出部分直接new新对象。

### public static String toString(long i){...}
```java
public static String toString(long i) {
    if (i == Long.MIN_VALUE)
        return "-9223372036854775808";
    int size = (i < 0) ? stringSize(-i) + 1 : stringSize(i);
    char[] buf = new char[size];
    getChars(i, size, buf);
    return new String(buf, true);
}
```
套路差不多一样，但是由于Long类型占的长度是Integer的两倍，所以不能用数组了（否则写到天荒地老），所以`stringSize`和`getChars`方法有所不同，其中`stringSize`方法定义如下：
```java
static int stringSize(long x) {
   long p = 10;
   for (int i=1; i<19; i++) {
       if (x < p)
           return i;
       p = 10*p;
   }
   return 19;
}
```
Long能表示的最大值是2的64次方，十进制表示的时候是19位数字。
其中`getChars`方法如下：
```java
static void getChars(long i, int index, char[] buf) {
    long q;
    int r;
    int charPos = index;
    char sign = 0;

    if (i < 0) {
        sign = '-';
        i = -i;
    }

    // Get 2 digits/iteration using longs until quotient fits into an int
    while (i > Integer.MAX_VALUE) {
        q = i / 100;
        // really: r = i - (q * 100);
        r = (int)(i - ((q << 6) + (q << 5) + (q << 2)));
        i = q;
        buf[--charPos] = Integer.DigitOnes[r];
        buf[--charPos] = Integer.DigitTens[r];
    }

    // Get 2 digits/iteration using ints
    int q2;
    int i2 = (int)i;
    while (i2 >= 65536) {
        q2 = i2 / 100;
        // really: r = i2 - (q * 100);
        r = i2 - ((q2 << 6) + (q2 << 5) + (q2 << 2));
        i2 = q2;
        buf[--charPos] = Integer.DigitOnes[r];
        buf[--charPos] = Integer.DigitTens[r];
    }

    // Fall thru to fast mode for smaller numbers
    // assert(i2 <= 65536, i2);
    for (;;) {
        q2 = (i2 * 52429) >>> (16+3);
        r = i2 - ((q2 << 3) + (q2 << 1));  // r = i2-(q2*10) ...
        buf[--charPos] = Integer.digits[r];
        i2 = q2;
        if (i2 == 0) break;
    }
    if (sign != 0) {
        buf[--charPos] = sign;
    }
}

```
这里的过程和Integer中的`getChars`方法一样，只是将过程分为了三个阶段，分为以`Integer.MAX_VALUE`和`65535`为界。


### public static int hashCode(long value){..}
定义如下：
```java
public static int hashCode(long value) {
    return (int)(value ^ (value >>> 32));
}
```
由于hashCode返回值是int类型，最多32位，而Long类型有64位，所以Long类型需要砍掉一半，这就是`value >>> 32`，但是为了不丢失这砍掉的一半的信息，会将值得高32位和低32位进行exclusive OR操作，这样就保证结果均会收到前后32位的影响，不丢失信息。如果直接将Long转为int，将会丢失高32位的信息。

> 为什么hashCode返回int型而不是long型？
>
> 因为在Java中，一个Array的最长长度是： Integer.MAX_VALUE
