# String
好吧 终于到了重头戏了 String一定要好好看！结构如下：
![String](http://ovn0i3kdg.bkt.clouddn.com/String_structure_1.png)
![String](http://ovn0i3kdg.bkt.clouddn.com/string_structure_2.png)

方法虽然多，但是大部分都是重载。

## public final class String implements java.io.Serializable, Comparable<String>, CharSequence
首先看这个类声明，`String`类被`final`修饰，不可继承。类实现了`Serializable`、`Comparable`接口和`CharSequence`接口，最后这接口的结构如下

![CharSequence](http://ovn0i3kdg.bkt.clouddn.com/CharSequence_structure.png)

之后我们会在String中看到接口方法的实现。

##  private final char value[];
字符数组用来存储字符串中的字符，这就是String内部的实现。特别需要注意的是，这里用`final`修饰，也就是说，一旦String的实例被创建，即value被填充，那么不可再更改，如果需要更改，那将是创建一个新对象，用新的内容赋值。这就是所谓的**字符串永久性**。

## private int hash;
String和之前遇到的类不同的一点在于，对象属性保存了其哈希值，默认为0。

## private static final ObjectStreamField[] serialPersistentFields
```java
private static final ObjectStreamField[] serialPersistentFields =
        new ObjectStreamField[0];
```

String类中还定义了一个属性，从名字上看起来跟序列化有关吧，可能比`serialVersionUID`有更多的用处，这里不做过多了解了，更详细的情况参考：http://www.infoq.com/cn/articles/cf-java-object-serialization-rmi/

## public static final Comparator&lt;String&gt; CASE_INSENSITIVE_ORDER
这个静态常量CASE_INSENSITIVE_ORDER是一个比较器，定义如下：
```java
public static final Comparator<String> CASE_INSENSITIVE_ORDER
                                         = new CaseInsensitiveComparator();
private static class CaseInsensitiveComparator
        implements Comparator<String>, java.io.Serializable {
    // use serialVersionUID from JDK 1.2.2 for interoperability
    private static final long serialVersionUID = 8575799808933029326L;

    public int compare(String s1, String s2) {
        int n1 = s1.length();
        int n2 = s2.length();
        int min = Math.min(n1, n2);
        for (int i = 0; i < min; i++) {
            char c1 = s1.charAt(i);
            char c2 = s2.charAt(i);
            if (c1 != c2) {
                c1 = Character.toUpperCase(c1);
                c2 = Character.toUpperCase(c2);
                if (c1 != c2) {
                    c1 = Character.toLowerCase(c1);
                    c2 = Character.toLowerCase(c2);
                    if (c1 != c2) {
                        // No overflow because of numeric promotion
                        return c1 - c2;
                    }
                }
            }
        }
        return n1 - n2;
    }
```
这里定义了内部类CaseInsensitiveComparator并实现了`Comparator<String>`中`compare(String s1, String s2)`接口,这里是按照字典序进行比较。

首先确定了也比较的范围min，然后就逐个字符比较呗，分别转为大小和小写后，如果还都不相等，就返回两个char字符的差值。如果在可比较范围之内都相等，那么就返回长度的差值。

刚看到这个源码的时候我很困惑，为什么要分别进行大小写两次转换呢？看到后面才明白这个方法被应用于比较方法`compareToIgnoreCase`中，意思就是忽略大小写的比较。

> 看到这个发现一个有趣的现象，String类本身被final，属性中除了`hash`外都被final修饰，真的是一旦建立就无法改变。

## 构造函数
`String`最多的就是构造函数，数了一下有16个，其中两个已经废弃了。那就按照顺序一个个来学习下吧。

###  public String() {..}
空参数表，定义如下：
```java
  public String() {
      this.value = "".value;
  }
```
把空字符串的值赋值给当前的value。 有一个问题，这个空字符串是存放在哪里么？是堆中还是常量池？学习到后面会知道，用双引号创建的字符串对象是存放在常量池中的。

### public String(String original){...}
接收String类型的参数用来构造另一个String对象，定义如下：
```java
public String(String original) {
    this.value = original.value;
    this.hash = original.hash;
}
```

有一道面试题是这样的：`String str = new String（"hello world"）；` 一共创建了几个对象？

答案是2个对象，"hello world"用双引号的方式创建了一个对象，然后以这个对象为参数，又new了一个对象。

> String 用双引号就可以创建对象，背后的原理就是常量池了，具体的在`intern`方法中有说明。


### public String(char value[]){...}
接收一个字符数组作为参数，定义如下：
```java
public String(char value[]) {
    this.value = Arrays.copyOf(value, value.length);
}
```
这个地方调用了`java.util.Arrays`中的静态方法`copyOf`，该方法定义如下：
```java
public static char[] copyOf(char[] original, int newLength) {
   char[] copy = new char[newLength];
   System.arraycopy(original, 0, copy, 0,
                    Math.min(original.length, newLength));
   return copy;
}
```
而这个方法里面呢，又调用了`java.lang.System`中的`arraycopy`方法，这是一个native方法，定义如下：
```java
public static native void arraycopy(Object src,  int  srcPos,
                                        Object dest, int destPos,
                                        int length);

```
好吧，不追究了，了解到这里就好。

##### public String(char value[], int offset, int count){..}
接收参数，就是拷贝出一个字符数组的一部分，定义如下:
```java
public String(char value[], int offset, int count) {
     if (offset < 0) {
         throw new StringIndexOutOfBoundsException(offset);
     }
     if (count <= 0) {
         if (count < 0) {
             throw new StringIndexOutOfBoundsException(count);
         }
         if (offset <= value.length) {
             this.value = "".value;
             return;
         }
     }
     // Note: offset or count might be near -1>>>1.
     if (offset > value.length - count) {
         throw new StringIndexOutOfBoundsException(offset + count);
     }
     this.value = Arrays.copyOfRange(value, offset, offset+count);
 }
```
先是做了参数检查以防止越界，最后调用的是`Arrays`类的`copyOfRange`方法，该方法定义如下：
```java
public static char[] copyOfRange(char[] original, int from, int to) {
    int newLength = to - from;
    if (newLength < 0)
        throw new IllegalArgumentException(from + " > " + to);
    char[] copy = new char[newLength];
    System.arraycopy(original, from, copy, 0,
                     Math.min(original.length - from, newLength));
    return copy;
}
```
和上面的copyOf差不多过程，多了参数检查。

### public String(byte bytes[], int offset, int length, String charsetName) throws UnsupportedEncodingException{...}
接收四个参数，第一个是byte数组，最后一个指定编码格式，定义如下：
```java
public String(byte bytes[], int offset, int length, String charsetName)
            throws UnsupportedEncodingException {
    if (charsetName == null)
        throw new NullPointerException("charsetName");
    checkBounds(bytes, offset, length);
    this.value = StringCoding.decode(charsetName, bytes, offset, length);
}
```
中间调用的`checkBounds`用来做越界检查，定义如下：
```java
private static void checkBounds(byte[] bytes, int offset, int length) {
    if (length < 0)
        throw new StringIndexOutOfBoundsException(length);
    if (offset < 0)
        throw new StringIndexOutOfBoundsException(offset);
    if (offset > bytes.length - length)
        throw new StringIndexOutOfBoundsException(offset + length);
}
```

### public String(byte bytes[], int offset, int length, Charset charset) {...}
这个方法和上面的方法差不多，但是最后一个参数是`Charset`类型，这样就避免了抛出`UnsupportedEncodingException`异常，定义如下：
```java
public String(byte bytes[], int offset, int length, Charset charset){
    if (charset == null)
        throw new NullPointerException("charset");
    checkBounds(bytes, offset, length);
    this.value =  StringCoding.decode(charset, bytes, offset, length);
}
```

### public String(byte bytes[], String charsetName){..}
这个函数省去了offset和length，实际上是默认了拷贝这个bytes数组，定义如下：
```java
public String(byte bytes[], String charsetName)
            throws UnsupportedEncodingException {
    this(bytes, 0, bytes.length, charsetName);
}
```

### public String(byte bytes[], Charset charset) {..}
同样的，默认拷贝整个bytes数组，定义如下：
```java
public String(byte bytes[], Charset charset) {
    this(bytes, 0, bytes.length, charset);
}
```

### public String(byte bytes[], int offset, int length) {..}
这个方法没有接受指定编码格式的参数，就按照默认的“ISO-8859-1”格式进行编码。
```java
public String(byte bytes[], int offset, int length){
  checkBounds(bytes, offset, length);
  this.value = StringCoding.decode(bytes, offset, length);
}
```
### public String(byte bytes[]) {..}
使用默认字符编码，拷贝整个bytes数组，定义如下：
```java
public String(byte bytes[]) {
    this(bytes, 0, bytes.length);
}
```
### public String(StringBuffer buffer){...}和 public String(StringBuilder builder){...}
`StringBuffer`和`StringBuilder`本身非常相似，区别在于前者是线程安全的，而后者不是。这一点在上面两个方法的定义中也可以体现：
```java
public String(StringBuffer buffer) {
    synchronized(buffer) {
        this.value = Arrays.copyOf(buffer.getValue(), buffer.length());
    }
}
```

```java
public String(StringBuilder builder) {
  this.value = Arrays.copyOf(builder.getValue(), builder.length());
}
```
可以看到，前者使用了`synchronized`关键词，是线程安全的，而后者并没有。

### String(char[] value, boolean share){...}
这里第二个参数的总是接受true，注意到之前同样有一个单接受字符数组的构造函数，但是不同的是，那个构造函数重新拷贝了一份数组再对value进行赋值，此时实例变量value和形参value指向的就是不同的两个地址，而在这个方法中，直接将形参的值赋值给实例变量，即两者的指向是相同的，这就是所谓的"share"。
```java
String(char[] value, boolean share) {
   // assert share : "unshared not supported";
   this.value = value;
}
```

## public int length() {...}
方法用于得到String的长度，实现很简单，返回字符数组的长度即可：
```java
public int length() {
    return value.length;
}
```
## public boolean isEmpty() {...}
用于判断字符串是否为空，实现如下：
```java
public boolean isEmpty() {
   return value.length == 0;
}
```

## public char charAt(int index){...}
得到索引index上的字符，先要进行越界判断，然后返回字符数组下标为index上的字符，index的大小范围是[0, value.length - 1]，实现如下：
```java
public char charAt(int index) {
  if ((index < 0) || (index >= value.length)) {
      throw new StringIndexOutOfBoundsException(index);
  }
  return value[index];
}
```
> 和`charAt`方法类似的还有`codePointAt(..)`、`codePointBefore(...)`、`codePointCount(...)`之类的，很少用到，先不看了。

## void getChars(char dst[], int dstBegin){...}
给定的这个方法用来拷贝字符数组的一部分，指定起点。实现如下：
```java

void getChars(char dst[], int dstBegin) {
    System.arraycopy(value, 0, dst, dstBegin, value.length);
}
```

## public void getChars(int srcBegin, int srcEnd, char dst[], int dstBegin){...}
这个就厉害了，不光可以指定原字符数组的起点和终点，还可以指定目的字符数组的起点。当然给定这么多的参数，方法一开始先要做参数检查：
```java
public void getChars(int srcBegin, int srcEnd, char dst[], int dstBegin) {
    if (srcBegin < 0) {
        throw new StringIndexOutOfBoundsException(srcBegin);
    }
    if (srcEnd > value.length) {
        throw new StringIndexOutOfBoundsException(srcEnd);
    }
    if (srcBegin > srcEnd) {
        throw new StringIndexOutOfBoundsException(srcEnd - srcBegin);
    }
    System.arraycopy(value, srcBegin, dst, dstBegin, srcEnd - srcBegin);
}
```

## public byte[] getBytes(String charsetName)throws UnsupportedEncodingException {...} 和 public byte[] getBytes(Charset charset) {...} 和  public byte[] getBytes(){...}
三个方法目的都是拷贝出byte数组。参数决定编码而已，很简单不具体说了。

## public boolean equals(Object anObject){...}
有朋友面试的时候被问过手写String的equals方法，所以好好看一眼吧。

```java
public boolean equals(Object anObject) {
    if (this == anObject) {
        return true;
    }
    if (anObject instanceof String) {
        String anotherString = (String)anObject;
        int n = value.length;
        if (n == anotherString.value.length) {
            char v1[] = value;
            char v2[] = anotherString.value;
            int i = 0;
            while (n-- != 0) {
                if (v1[i] != v2[i])
                    return false;
                i++;
            }
            return true;
        }
    }
    return false;
}
```
这里套路是这么走的：先判断是不是同一个引用，然后判断是不是String类型的实例【记住了，继承Object类的equals方法的参数是Object类型】，都OK了之后就应该判断内容是不是一致了。怎么判断呢？先判断长度，然后从前往后逐个字符比较，一旦不容就返回false。很简单，可以注意下这个比较的顺序安排，越轻松不费事的判断越要放到前面。

## public int hashCode(){...}
重申那句话：**重写equals方法一定也要重写hashCode方法**，看看String是怎么定义`hashCode`方法的：
```java
public int hashCode() {
    int h = hash;
    if (h == 0 && value.length > 0) {
        char val[] = value;

        for (int i = 0; i < value.length; i++) {
            h = 31 * h + val[i];
        }
        hash = h;
    }
    return h;
}
```
用数学表达式来说就是这样的：`hashCode = s[0]*31^(n-1) + s[1]*31^(n-2) + ... + s[n-1]`其中，n为字符串的长度，s[i]是各个位置上的字符。空字符串的hash值为0。

> 这个有一个问题，字符串在创建的时候，其属性`hash`默认为0，之后也没有看到什么地方给赋值了，只有在这个`hashCode`方法中给hash属性赋值了，难道这个方法如果一直不调用，hash属性就一直等于0? 在源码中打断点试验了以下，确实是这样的。没调用`hashCode`方法之前`hash`属性一直是0。

## public int compareTo(String anotherString){..}
两个字符串的比较，要么返回开始不相同的位置上的字符的差值，要么返回两者的长度差值，定义如下：
```java
public int compareTo(String anotherString) {
   int len1 = value.length;
   int len2 = anotherString.value.length;
   int lim = Math.min(len1, len2);
   char v1[] = value;
   char v2[] = anotherString.value;

   int k = 0;
   while (k < lim) {
       char c1 = v1[k];
       char c2 = v2[k];
       if (c1 != c2) {
           return c1 - c2;
       }
       k++;
   }
   return len1 - len2;
}
```
## public int compareToIgnoreCase(String str){...}
也是字符串的比较，但是这次忽略了两个字符串的大小写，怎么实现呢？用到了前面说的比较器
```java
public int compareToIgnoreCase(String str) {
    return CASE_INSENSITIVE_ORDER.compare(this, str);
}
```
## regionMatches方法
用来比较两个字符串部分区域是否一样，实现思路很简单，逐字符比较。后面的方法比前面多一个参数，意思是是否选择忽略大小写，如果不忽略，和前面的方法效果是一样的，两个函数的实现过程如下：
```java
public boolean regionMatches(int toffset, String other, int ooffset,
            int len) {
    char ta[] = value;
    int to = toffset;
    char pa[] = other.value;
    int po = ooffset;
    // Note: toffset, ooffset, or len might be near -1>>>1.
    if ((ooffset < 0) || (toffset < 0)
            || (toffset > (long)value.length - len)
            || (ooffset > (long)other.value.length - len)) {
        return false;
    }
    while (len-- > 0) {
        if (ta[to++] != pa[po++]) {
            return false;
        }
    }
    return true;
}
```
```java
public boolean regionMatches(boolean ignoreCase, int toffset,
            String other, int ooffset, int len) {
    char ta[] = value;
    int to = toffset;
    char pa[] = other.value;
    int po = ooffset;
    // Note: toffset, ooffset, or len might be near -1>>>1.
    if ((ooffset < 0) || (toffset < 0)
            || (toffset > (long)value.length - len)
            || (ooffset > (long)other.value.length - len)) {
        return false;
    }
    while (len-- > 0) {
        char c1 = ta[to++];
        char c2 = pa[po++];
        if (c1 == c2) {
            continue;
        }
        if (ignoreCase) {
            // If characters don't match but case may be ignored,
            // try converting both characters to uppercase.
            // If the results match, then the comparison scan should
            // continue.
            char u1 = Character.toUpperCase(c1);
            char u2 = Character.toUpperCase(c2);
            if (u1 == u2) {
                continue;
            }
            // Unfortunately, conversion to uppercase does not work properly
            // for the Georgian alphabet, which has strange rules about case
            // conversion.  So we need to make one last check before
            // exiting.
            if (Character.toLowerCase(u1) == Character.toLowerCase(u2)) {
                continue;
            }
        }
        return false;
    }
    return true;
}
```

## startsWith方法
用来判断字符串是不是以特定的字符串开头，后者调用前者。注意这里"开头"的意思可不一定是从0开始的，前者offset指定了偏移位置，后者则默认从0开始，前者的定义如下：
```java
public boolean startsWith(String prefix, int toffset) {
    char ta[] = value;
    int to = toffset;
    char pa[] = prefix.value;
    int po = 0;
    int pc = prefix.value.length;
    // Note: toffset might be near -1>>>1.
    if ((toffset < 0) || (toffset > value.length - pc)) {
        return false;
    }
    while (--pc >= 0) {
        if (ta[to++] != pa[po++]) {
            return false;
        }
    }
    return true;
}
```

## public boolean endsWith(String suffix){...}
聪明的我已经猜到了，这个肯定是调用了有两个参数的`startsWith`方法，事实上就是这么做的。不多说了。

## indexOf 和 lastIndexOf方法
套路差不多的四个函数，重载的方法之间，单个参数的调用了两个参数的方法。所以只选择一个来看下：
```java
public int indexOf(int ch, int fromIndex) {
   final int max = value.length;
   if (fromIndex < 0) {
       fromIndex = 0;
   } else if (fromIndex >= max) {
       // Note: fromIndex might be near -1>>>1.
       return -1;
   }

   if (ch < Character.MIN_SUPPLEMENTARY_CODE_POINT) {
       // handle most cases here (ch is a BMP code point or a
       // negative value (invalid code point))
       final char[] value = this.value;
       for (int i = fromIndex; i < max; i++) {
           if (value[i] == ch) {
               return i;
           }
       }
       return -1;
   } else {
       return indexOfSupplementary(ch, fromIndex);
   }
}
```
可以看到，类型检查方法决定：如果指定的起点小于0则默认从0开始，如果超出字符串上就返回-1表示找不到。然后在ch合法的情况系一个个比较字符，找到就返回下标，注意的起点是fromIndex，包含这个位置。如果字符不合法就调用`indexOfSupplementary`，这个方法不重要，不看了。

`lastIndexOf`方法差不多，不同的地方就是从后往前比较。

## indexOf和lastIndexOf方法
重载了一堆，最终调用的是它：
```java
static int indexOf(char[] source, int sourceOffset, int sourceCount,
            char[] target, int targetOffset, int targetCount,
        int fromIndex) {
    if (fromIndex >= sourceCount) {
        return (targetCount == 0 ? sourceCount : -1);
    }
    if (fromIndex < 0) {
        fromIndex = 0;
    }
    if (targetCount == 0) {
        return fromIndex;
    }

    char first = target[targetOffset];
    int max = sourceOffset + (sourceCount - targetCount);

    for (int i = sourceOffset + fromIndex; i <= max; i++) {
        /* Look for first character. */
        if (source[i] != first) {
            while (++i <= max && source[i] != first);
        }

        /* Found first character, now look at the rest of v2 */
        if (i <= max) {
            int j = i + 1;
            int end = j + targetCount - 1;
            for (int k = targetOffset + 1; j < end && source[j]
                    == target[k]; j++, k++);

            if (j == end) {
                /* Found whole string. */
                return i - sourceOffset;
            }
        }
    }
    return -1;
}
```
实现过程很简单，不多说了。

## substring方法

取得子串，最后一个方法是实现了`CharSequence`接口的方法，直接调用了第二个方法。实际上第一个也是调用了第二个方法，所以来看第二个方法的定义：
```java
public String substring(int beginIndex, int endIndex) {
   if (beginIndex < 0) {
       throw new StringIndexOutOfBoundsException(beginIndex);
   }
   if (endIndex > value.length) {
       throw new StringIndexOutOfBoundsException(endIndex);
   }
   int subLen = endIndex - beginIndex;
   if (subLen < 0) {
       throw new StringIndexOutOfBoundsException(subLen);
   }
   return ((beginIndex == 0) && (endIndex == value.length)) ? this
           : new String(value, beginIndex, subLen);
}
```
注意看的是在一些列边界检查之后，最后为了优化效果，做了一个判断，当取得的子串就是字符串本身的时候直接返回了本身的引用，而不是再去new一个新的字符串。这种优化可以学习下。


##  public String concat(String str){...}
在本身后面再拼接字符串，之前说过本身的属性value被final修饰，是不可改变的，所以本身不能拼接，只能往一个新的字符串数组中赋值，之后再new一个新的String。定义如下：
```java
public String concat(String str) {
   int otherLen = str.length();
   if (otherLen == 0) {
       return this;
   }
   int len = value.length;
   char buf[] = Arrays.copyOf(value, len + otherLen);
   str.getChars(buf, len);
   return new String(buf, true);
}
```
## public String replace(char oldChar, char newChar){...}
用newChar替换字符串中的oldChar，实现如下：
```java
public String replace(char oldChar, char newChar) {
    if (oldChar != newChar) {
        int len = value.length;
        int i = -1;
        char[] val = value; /* avoid getfield opcode */

        while (++i < len) {
            if (val[i] == oldChar) {
                break;
            }
        }
        if (i < len) {
            char buf[] = new char[len];
            for (int j = 0; j < i; j++) {
                buf[j] = val[j];
            }
            while (i < len) {
                char c = val[i];
                buf[i] = (c == oldChar) ? newChar : c;
                i++;
            }
            return new String(buf, true);
        }
    }
    return this;
}
```
由过程很容易看到，原来字符串本身不会有变化，方法将会返回一个替换了之后的字符串数组的副本。还可以看到，这种替换是全局的。

## public boolean matches(String regex){...}
参数是正则表达式，判断字符串中是否有满足正则表达式的子串。实现如下：
```java
public boolean matches(String regex) {
   return Pattern.matches(regex, this);
}
```
这里直接调用的是`Pattern`的静态方法`matches`，等看到这个类再说吧。

##  public boolean contains(CharSequence s){...}
判断字符串是不是含有某个子串，那好办，前面不是有indexOf方法么， 有就返回开始位置，没有就返回-1。这里只需要判断返回值是不是大于-1就行。不多说了。

## replaceFirst、replaceAll和replace方法
```java
public String replaceFirst(String regex, String replacement) {
   return Pattern.compile(regex).matcher(this).replaceFirst(replacement);
}
```

```java
public String replaceAll(String regex, String replacement) {
    return Pattern.compile(regex).matcher(this).replaceAll(replacement);
}
```

```java
public String replace(CharSequence target, CharSequence replacement) {
      return Pattern.compile(target.toString(), Pattern.LITERAL).matcher(
            this).replaceAll(Matcher.quoteReplacement(replacement.toString()));
}
```

需要注意的是，这个方法的第一个参数是正则表达式，而不是字符串，有道笔试题就是用`.`作为第一个参数，问你结果是什么？`.`在正则中表示任何字符，所以当然是全部替换啦。

## split方法

后者调用前者，设置第二个参数为0。前者方法中第二个参数的含义是，定义如下：
```java
public String[] split(String regex, int limit) {
    /* fastpath if the regex is a
     (1)one-char String and this character is not one of the
        RegEx's meta characters ".$|()[{^?*+\\", or
     (2)two-char String and the first char is the backslash and
        the second is not the ascii digit or ascii letter.
     */
    char ch = 0;
    if (((regex.value.length == 1 &&
         ".$|()[{^?*+\\".indexOf(ch = regex.charAt(0)) == -1) ||
         (regex.length() == 2 &&
          regex.charAt(0) == '\\' &&
          (((ch = regex.charAt(1))-'0')|('9'-ch)) < 0 &&
          ((ch-'a')|('z'-ch)) < 0 &&
          ((ch-'A')|('Z'-ch)) < 0)) &&
        (ch < Character.MIN_HIGH_SURROGATE ||
         ch > Character.MAX_LOW_SURROGATE))
    {
        int off = 0;
        int next = 0;
        boolean limited = limit > 0;
        ArrayList<String> list = new ArrayList<>();
        while ((next = indexOf(ch, off)) != -1) {
            if (!limited || list.size() < limit - 1) {
                list.add(substring(off, next));
                off = next + 1;
            } else {    // last one
                //assert (list.size() == limit - 1);
                list.add(substring(off, value.length));
                off = value.length;
                break;
            }
        }
        // If no match was found, return this
        if (off == 0)
            return new String[]{this};

        // Add remaining segment
        if (!limited || list.size() < limit)
            list.add(substring(off, value.length));

        // Construct result
        int resultSize = list.size();
        if (limit == 0) {
            while (resultSize > 0 && list.get(resultSize - 1).length() == 0) {
                resultSize--;
            }
        }
        String[] result = new String[resultSize];
        return list.subList(0, resultSize).toArray(result);
    }
    return Pattern.compile(regex).split(this, limit);
}

```
这里有一个快速的路径，需要满足下面几个条件：

①如果这个分隔符是单个字符串并且这个字符串是`.$|()[{^?*+\`这中间的某一个    

或者

②如果这个分隔符是两个字符，并且这两个字符中第一个字符是反斜杠"\"，第二个字符不是数字也不是字母也不是unicode的范围内的字符（?）

以上两种情况，满足任何一种都能够进入“快速通道”，快速通道怎么处理呢？就是匹配分隔符然后将被分割得到的子串放到list表中。那么方法的第二个参数limit是做什么的呢？
官方的解释是：
> limit 参数控制模式应用的次数，因此影响所得数组的长度。如果该限制 n 大于 0，则模式将被最多应用 n - 1 次，数组的长度将不会大于 n，而且数组的最后一项将包含所有超出最后匹配的定界符的输入。如果 n 为非正，那么模式将被应用尽可能多的次数，而且数组可以是任何长度。如果 n 为 0，那么模式将被应用尽可能多的次数，数组可以是任何长度，并且结尾空字符串将被丢弃。

为什么是这样么，我们结合一个例子来看一下：
```java
public static void main(String[] args) {
     String str = "boo:and:foo";
     System.out.println(Arrays.toString(str.split(":", 2))); ①
     System.out.println(Arrays.toString(str.split(":", 5)));②
     System.out.println(Arrays.toString( str.split(":", -2)));③
     System.out.println(Arrays.toString(str.split("o", 5)));④
     System.out.println(Arrays.toString(str.split("o", -2)));⑤
     System.out.println(Arrays.toString(str.split("o", 0)));⑥
 }
```
以上程序的输出结果是：
```java
[boo, and:foo]
[boo, and, foo]
[boo, and, foo]
[b, , :and:f, , ]
[b, , :and:f, , ]
[b, , :and:f]
```
逐条分析：

① split是":"，limit是2，由于分隔符满足条件②，所以进入"快速通道"。在while第一次循环中，取得next是3，进入if判断，满足第二个条件，往list中存放子串boo，并将off置为next的下一个位置，即4。这时候list的长度是1。进入下一次循环，取得next的大小是7，进入if判断，两个条件都不满足，这时候进入else块，将剩下的子串一股脑地都放到list中，off也变成了原字符串的总长度，退出循环。所以最终产生的数组为[boo, and:foo]。

② 与①不同的是，此时limit是5，在本例中list的大小最大是3，所以不会超过5-1,所以不会进入else块，而是很顺利的一直取得放到list中，最后一次分割后，退出循环，剩下的尾巴也会放到list中。

③ 当limit是负数的时，在while循环中的if判断里，会一直满足第一个条件，永远不会进入else块，一直不断取得子串，这个过程和②差不多，最后一次分割后，退出循环，剩下的尾巴也会放到list中。

④ split是"o"，limit是5， 由于分隔符满足条件②，所以进入"快速通道"。在while第一次循环中，取得next是1，进入if块，执行substring(0, 1)，取得"b"，将off置为2，下一次取得next为2，再执行substring(2, 2)，取得空字符串，将off置为3，下一次取得next为9, 执行substring(3, 9) ，取得“:and:f”，将off置为10，下一次取得next为10，执行substring(10, 10),取得空字符串，将off置为11。下一次取得next为11,执行substring(11,11)，取得空字符串；下一次取得next为-1，循环结束。

⑤ 这个过程和④没有区别，只是if判断满足的条件不同而已。

⑥ 前面几个过程都和一样，不一样的地方出现在`if (limit == 0){...}`这个判断的地方，这时候，会从后往前遍历list，碰到空字符串舍去，直到碰到一个非空字符串，所以结果是[b, , :and:f]，比起⑤的结果，少了后面的两个空字符串。

有了上面的例子，就很能理解官方解释的那段话是什么意思了，不用背下来啊，对源码有个印象就行。


##  join 方法
这个方法用来将多个字符串用特定的分隔符拼接起来，比如`String message = String.join("-", "Java", "is", "cool");`得到的message是“Java-is-cool”。
前者接受一个不定长参数，定义如下：
```Java
public static String join(CharSequence delimiter, CharSequence... elements) {
    Objects.requireNonNull(delimiter);
    Objects.requireNonNull(elements);
    // Number of elements not likely worth Arrays.stream overhead.
    StringJoiner joiner = new StringJoiner(delimiter);
    for (CharSequence cs: elements) {
        joiner.add(cs);
    }
    return joiner.toString();
}
```
这里有几个类很陌生，`Objects`和`StringJoiner`都是`java.util`中的类，学到的时候再说。总之，不定长参数可以被解析成数组的，遍历添加到`StringJoiner`实例中，最后返回字符串即可。

后者接受一个迭代器Iterable，实现过程仍旧是遍历
```java
public static String join(CharSequence delimiter,
            Iterable<? extends CharSequence> elements) {
    Objects.requireNonNull(delimiter);
    Objects.requireNonNull(elements);
    StringJoiner joiner = new StringJoiner(delimiter);
    for (CharSequence cs: elements) {
        joiner.add(cs);
    }
    return joiner.toString();
}
```
过程一模一样，不说了。

##  toLowerCase 和 toUpperCase 方法
用来转换大小写的，源码实现不太重要，先不看了。

## public String trim(){...}
这个方法返回一个去掉头尾空白格的副本字符串，实现过程很简单：
```java
public String trim() {
   int len = value.length;
   int st = 0;
   char[] val = value;    /* avoid getfield opcode */

   while ((st < len) && (val[st] <= ' ')) {
       st++;
   }
   while ((st < len) && (val[len - 1] <= ' ')) {
       len--;
   }
   return ((st > 0) || (len < value.length)) ? substring(st, len) : this;
}
```
这里说去掉的是“空格”好像不太严谨，源码告诉我们，去掉的字符实际上是ascii码小于等空格的字符，那哪些字符是小于这个空格的呢？参考http://www.asciima.com/， 发现是一些行开始符、换行符这种的。所以严格来说，`trim`方法是不仅仅去掉空格（空白的ascii码是32），而是一些看不见的字符。

## public String toString(){..}

当然返回this就行了啊~

## public char[] toCharArray(){...}
返回一个字符数组，诶，String类不是有一个字符数组的属性么？返回这个属性？当然不行，属性私有的，况且不可操作，当然是用`System.arraycopy`来复制一个副本啊。定义如下：
```java
public char[] toCharArray() {
    // Cannot use Arrays.copyOf because of class initialization order issues
    char result[] = new char[value.length];
    System.arraycopy(value, 0, result, 0, value.length);
    return result;
}
```

##   format
格式化，都是调用了`java.util.Format`中的方法，各自定义如下：
```java
public static String format(String format, Object... args) {
    return new Formatter().format(format, args).toString();
}
```

```java
public static String format(Locale l, String format, Object... args) {
    return new Formatter(l).format(format, args).toString();
}
```
就是用“%s”作为占位符，然后在第二个参数中指定参数。注意，由于占位符是用百分号写的，所以如果这个字符串中本来就有百分号的话怎么办呢？用转义，即"%%"。

## valueOf方法
这个方法重载太多了并且实现很简单，所以就放到一起说了.

| 方法参数  | 内容    |
| :------------- | :------------- |
|Object obj   |  return (obj == null) ? "null" : obj.toString(); |
|char data[]   | return new String(data);  |
|char data[], int offset, int count   |  return new String(data, offset, count); |
|boolean b   |  return b ? "true" : "false"; |
|char c   |  char data[] = {c}; return new String(data, true); |
|int i   |  Integer.toString(i); |
|long l   | return Long.toString(l);  |
|float f   | return Float.toString(f);  |
|double d   |  return Double.toString(d); |


## public native String intern();
最厉害的来了！无数题的考点啊！【敲黑板！！！】而且是native方法，不追踪了，有精力可以参考http://www.importnew.com/14142.html 研究下。

这个方法有一段注释我觉得有必要贴上来：
```java
/**
 * Returns a canonical representation for the string object.
 * <p>
 * A pool of strings, initially empty, is maintained privately by the
 * class {@code String}.
 * <p>
 * When the intern method is invoked, if the pool already contains a
 * string equal to this {@code String} object as determined by
 * the {@link #equals(Object)} method, then the string from the pool is
 * returned. Otherwise, this {@code String} object is added to the
 * pool and a reference to this {@code String} object is returned.
 * <p>
 * It follows that for any two strings {@code s} and {@code t},
 * {@code s.intern() == t.intern()} is {@code true}
 * if and only if {@code s.equals(t)} is {@code true}.
 * <p>
 * All literal strings and string-valued constant expressions are
 * interned. String literals are defined in section 3.10.5 of the
 * <cite>The Java&trade; Language Specification</cite>.
 *
 * @return  a string that has the same contents as this string, but is
 *          guaranteed to be from a pool of unique strings.
 */
```
讲的很明白了，String维护这一个常量池，最开始是空的。当`intern`方法被触发的时候，会先常量池中根据`equals`方法先寻找是不是存在当前字符串, 就会直接返回当前字符串. 如果常量池中没有此字符串, 会将此字符串放入常量池中后, 再返回。那什么时候创建的对象放到常量池中，什么时候创建的对象放到堆中呢？

以下两种情况创建的字符串对象会放到常量池中：
1. 直接使用双引号声明出来的String对象会直接存储在常量池中。
2. 直接使用intern方法。intern 方法会从字符串常量池中查询当前字符串是否存在，若不存在就会将当前字符串放入常量池中。

诶，到这里就想起来一道题目，问`String hello = "dog";  String lili = "dog";`那么`hello == lili`的值是什么？有了上面的知识储备，答案很显然是true，因为helo是用双引号的方式创建的字符串，对象放在常量池中，当仍用双引号创建对象，所以会从常量池中找，发现“dog”这个值已经被创建过了，所以返回引用。所以啊hello和lili指向的是常量池中的同一个对象，结果当然是true。

注意啊，`intern`这个方法是一个实例方法，这个方法最后返回的是常量池中的引用，那么问题来了例如：
```java
String s = new String("1");
s.intern();
```
执行之后，s引用指向的到底是堆地址呢还是常量池中的地址呢？堆地址！此时s指向的是堆中的地址，而该地址中存储的string的内容指向了常量池。有点绕，可以打开下面链接，里面的图画的很明白了。

JDK1.6和1.7版本对`intern`方法的解释不一样，归根到底是由于1.7调整了常量池的位置，http://www.importnew.com/14142.html 这篇博文讲的很详细了。我就不再抄一遍了。

看到链接的博客评论里有一个实例，我模仿写了下面的代码（环境1.8）：
```java
String string=new String("ttt" ) ;①
string.intern();②
String adString="ttt"; ③
System.out.println(string==adString); // false
String s5 = new String("aa") + new String("a");④
s5.intern(); ⑤
String s6 = "aaa"; ⑥
System.out.println(s5 == s6); //true
```
为什么打印出`false` 和`true`呢。
对于①，最终会创建出两个对象：常量池中的"ttt"和堆中的"ttt"。②的作用是去常量池中寻找是否存在"ttt"，结果是存在。③用双引号创建，去常量池中寻找"ttt"，找到了，所以adString引用的是常量池中的地址。所以string和adString分别指向了堆和常量池，当然不相等。

对于④，最终创建了三个对象：常量池中的“aa”、常量池中的"a"和堆中的"aaa"。当⑤去常量池中寻找”aaa"的时候发现没有，但是堆中有啊，所以直接引用了堆中的地址。当⑥用双引号创建对象的时候在常量池中找到了这个引用的地址，所以最终s5和s6引用的对象是同一个，所以相等返回true。
