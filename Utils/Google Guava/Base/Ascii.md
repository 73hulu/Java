# Ascii

包含处理ASCII字符（取值范围为0x00 through 0x7F，即取值为[0, 127)）的一些静态方法。结构如下：

![Ascii](https://ws2.sinaimg.cn/large/006tNc79ly1fvts6vpwpnj30j40iq74j.jpg)

其中常量部分为一些常见字符的ASCII值，这里不再赘述。

> ASCII码查询可以参考 http://ascii.911cha.com/

下面展开介绍下其中的静态方法。


## toLowerCase方法
用来进行大小写的转换，重载了三个方法
### public static String toLowerCase(String string){..}
将字符串转为小写形式。
```Java
public static String toLowerCase(String string) {
  int length = string.length();
  for (int i = 0; i < length; i++) {
    // 将找到第一个大写字母开始再进行转化，如果入参本身是全小写，则返回原来的参数
    if (isUpperCase(string.charAt(i))) {
      char[] chars = string.toCharArray();
      for (; i < length; i++) {
        char c = chars[i];
        if (isUpperCase(c)) {
          // 英文大小写字母之间的异或结果是Ox20
          chars[i] = (char) (c ^ CASE_MASK);
        }
      }
      return String.valueOf(chars);
    }
  }
  return string;
}

/** A bit mask which selects the bit encoding ASCII character case. */
private static final char CASE_MASK = 0x20;
```
特别注意其中大小写转换的方式非常秒，这里使用了大小写的掩码。比如A（ASCII为Ox41），a（ASCII为Ox61），它们异或的结果为Ox20。利用这个结论，很快进行大小写的抓换。JDK中`String`类的`toLowerCase`方法比较复杂，还考虑到了Local的区别。

### public static String toLowerCase(CharSequence chars){..}
这里的入参是`CharSequence`类型，我们常常用的`String`只是其中的一种实现。`CharSequence`的底层其实就是`char[]`，所以使用遍历+字符转化的方法即可实现大小写转换。

```java
public static String toLowerCase(CharSequence chars) {
  if (chars instanceof String) {
    return toLowerCase((String) chars);
  }
  char[] newChars = new char[chars.length()];
  for (int i = 0; i < newChars.length; i++) {
    newChars[i] = toLowerCase(chars.charAt(i));
  }
  return String.valueOf(newChars);
}
```

### public static char toLowerCase(char c){..}
针对于单个字符的大小写转换，很简单的实现。
```Java
public static char toLowerCase(char c) {
  return isUpperCase(c) ? (char) (c ^ CASE_MASK) : c;
}
```

## toUpperCase
转化为大写，同样有三个重载，不再赘述。

## public static boolean isLowerCase(char c){..}
判断是否为字母小写，只用判断其值范围是否在[a, z]即可。
```Java
public static boolean isLowerCase(char c) {
  // Note: This was benchmarked against the alternate expression "(char)(c - 'a') < 26" (Nov '13)
  // and found to perform at least as well, or better.
  return (c >= 'a') && (c <= 'z');
}
```

## public static boolean isUpperCase(char c){..}
判断是否为字母大写，判断是否在[A, Z]即可。

## public static String truncate(CharSequence seq, int maxLength, String truncationIndicator){..}
这个方法主要实现了截取字符串和拼接字符串的作用。`seq`是待截取的字符串。`maxLength`用来限制字符串的最大长度，`truncationLength`是待拼接的字符串。

这里需要注意的限制是，`maxLength`的长度必须不小于`truncationIndicator`的长度，即`seq`中至少有一个字符被截取到。

如果`seq`的长度小于`maxLength`，那么就不进行截取拼接了，直接返回原字符串。

```Java
public static String truncate(CharSequence seq, int maxLength, String truncationIndicator) {
  checkNotNull(seq);

  // length to truncate the sequence to, not including the truncation indicator
  int truncationLength = maxLength - truncationIndicator.length();

  // in this worst case, this allows a maxLength equal to the length of the truncationIndicator,
  // meaning that a string will be truncated to just the truncation indicator itself
  checkArgument(
      truncationLength >= 0,
      "maxLength (%s) must be >= length of the truncation indicator (%s)",
      maxLength,
      truncationIndicator.length());

  if (seq.length() <= maxLength) {
    String string = seq.toString();
    if (string.length() <= maxLength) {
      return string;
    }
    // if the length of the toString() result was > maxLength for some reason, truncate that
    seq = string;
  }

  return new StringBuilder(maxLength)
      .append(seq, 0, truncationLength)
      .append(truncationIndicator)
      .toString();
}
```
总结以下过程：在检查完参数合法性之后，如果发现`seq`长度本身小于`maxLength`，那么不进行任何操作就返回原字符串，否则进行截取和拼接。测试例子如下：
```Java
Ascii.truncate("foobar", 7, "..."); // returns "foobar"
Ascii.truncate("foobar", 5, "..."); // returns "fo..."
```

## public static boolean equalsIgnoreCase(CharSequence s1, CharSequence s2){..}
不计大小写比较两个字符串是否相等。注意需要考虑到非字母字符的情况，如果出现非字符字符，那么就返回false。

```Java
public static boolean equalsIgnoreCase(CharSequence s1, CharSequence s2) {
  // Calling length() is the null pointer check (so do it before we can exit early).
  int length = s1.length();
  if (s1 == s2) {
    return true;
  }
  if (length != s2.length()) {
    return false;
  }
  for (int i = 0; i < length; i++) {
    char c1 = s1.charAt(i);
    char c2 = s2.charAt(i);
    if (c1 == c2) {
      continue;
    }
    int alphaIndex = getAlphaIndex(c1);
    // This was also benchmarked using '&' to avoid branching (but always evaluate the rhs),
    // however this showed no obvious improvement.
    if (alphaIndex < 26 && alphaIndex == getAlphaIndex(c2)) {
      continue;
    }
    return false;
  }
  return true;
}

// 注意这里取得字符在字母表中索引的方法。
/**
 * Returns the non-negative index value of the alpha character {@code c}, regardless of case. Ie,
 * 'a'/'A' returns 0 and 'z'/'Z' returns 25. Non-alpha characters return a value of 26 or greater.
 */
private static int getAlphaIndex(char c) {
  // Fold upper-case ASCII to lower-case and make zero-indexed and unsigned (by casting to char).
  return (char) ((c | CASE_MASK) - 'a');
}
```
例如对于小写字母a（ASCII码为Ox41）则 `c | CASE_MASK`的结果为"ox61"，`getAlphaIndex`的结果为32，这是数字0的ASCII码。对于大写字母A（ASCII码为97），`c | CASE_MASK`的结果为97， `getAlphaIndex`的结果同样为32。所以"a/A"的结果是0，'z/Z'的结果为25。
