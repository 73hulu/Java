# Strings

`Strings`一看就是对于JDK`String`的补充，工具类，提供静态方法。

![Strings](https://ws3.sinaimg.cn/large/006tNbRwly1fvzp6ev5zmj30iq0ecq34.jpg)


## public final class Strings
类声明，被`final`修饰。无法被继承。

## 构造方法
由于是工具类，干脆隐藏构造方法，所以置为private，略。

## public static String nullToEmpty(@Nullable String string){..}
如果参数为null，则返回空串，否则返回原字符串。
```Java
public static String nullToEmpty(@Nullable String string) {
  return Platform.nullToEmpty(string);
}

// Platform
static String nullToEmpty(@Nullable String string) {
  return (string == null) ? "" : string;
}
```

## public static @Nullable String emptyToNull(@Nullable String string){..}
和`nullToEmpty`刚好相反：如果参数为空串则返回null，否则返回原字符串。
```Java
public static @Nullable String emptyToNull(@Nullable String string) {
  return Platform.emptyToNull(string);
}
```

## public static boolean isNullOrEmpty(@Nullable String string){..}
判别字符串是否为null或空串，实现很简单。略。

## public static String padStart(String string, int minLength, char padChar){..}
用于字符串的格式化填充，`minLength`指定最小长度，如果`string`本身长度大于`minLength`，则不执行填充，否则在`string`之前用`padChar`进行填充。

```Java
public static String padStart(String string, int minLength, char padChar){
  checkNotNull(string); // eager for GWT.
  if (string.length() >= minLength) {
    return string;
  }
  StringBuilder sb = new StringBuilder(minLength);
  for (int i = string.length(); i < minLength; i++) {
    sb.append(padChar);
  }
  sb.append(string);
  return sb.toString();
}
```
示例如下：
```Java
Strings.padStart("7", 3, '0'); // "007"
Strings.padStart("2010", 3, '0'); // "2010"
```

## public static String padEnd(String string, int minLength, char padChar){..}
和`padStart`的区别只在于一个开头填充，一个结尾填充。定义略，使用如下：
```Java
Strings.padEnd("4.", 5, '0'); // 4.000
Strings.padEnd("2010", 3, '!'); // "2010"
```
> `Ascii`中`truncate`方法有点类似于该方法，但是有点不太一样。

## public static String repeat(String string, int count){..}
顾名思义，用于快速创建子串周期重复的字符串。
```Java
public static String repeat(String string, int count) {
  checkNotNull(string); // eager for GWT.

  if (count <= 1) {
    checkArgument(count >= 0, "invalid count: %s", count);
    return (count == 0) ? "" : string;
  }

  // IF YOU MODIFY THE CODE HERE, you must update StringsRepeatBenchmark
  final int len = string.length();
  final long longSize = (long) len * (long) count;
  final int size = (int) longSize;
  if (size != longSize) {
    throw new ArrayIndexOutOfBoundsException("Required array size too large: " + longSize);
  }

  final char[] array = new char[size];
  string.getChars(0, len, array, 0);
  int n;
  for (n = len; n < size - n; n <<= 1) {
    System.arraycopy(array, 0, array, n, n);
  }
  System.arraycopy(array, 0, array, n, size - n);
  return new String(array);
}
```
这里使用`System.arraycopy`的方法创建字符串，这是`native`方法，比起`StringBuilder`等高级类来说要快很多。

## public static String commonPrefix(CharSequence a, CharSequence b){..}
寻找两个字符序列的公共前缀，定义如下：
```Java
public static String commonPrefix(CharSequence a, CharSequence b) {
  checkNotNull(a);
  checkNotNull(b);

  int maxPrefixLength = Math.min(a.length(), b.length());
  int p = 0;
  while (p < maxPrefixLength && a.charAt(p) == b.charAt(p)) {
    p++;
  }
  if (validSurrogatePairAt(a, p - 1) || validSurrogatePairAt(b, p - 1)) {
    p--;
  }
  return a.subSequence(0, p).toString();
}

@VisibleForTesting
static boolean validSurrogatePairAt(CharSequence string, int index) {
  return index >= 0
      && index <= (string.length() - 2)
      && Character.isHighSurrogate(string.charAt(index))
      && Character.isLowSurrogate(string.charAt(index + 1));
}
```
`commonPrefix`中一直很不理解的是中间的那个`if`分支，在`validSurrogatePairAt`中，前面两个对于`index`的而判断我都能理解（这里需要注意“前缀”的概念，字符串本身不能算作本身的前缀）。但是后面`isHighSurrogate`和`isLowSurrogate`是为什么我就不太能理解了，这个方法的本意是“确定给出的 char 值是否为一个高代理项代码单元（也称为前导代理项代码单元）。这类值并不表示它们本身的字符，而被用来表示 utf-16 编码中的增补字符。”emmmm 没太懂。


## public static String commonSuffix(CharSequence a, CharSequence b) {..}
查找两个字符序列的公共后缀，实现和`commonPrefix`差不多，略过。

## public static String lenientFormat(@Nullable String template, @Nullable Object @Nullable... args){..}
格式化字符串，将`template`中`%s`一次换成`args`中的参数，与`String.format`作用一样。
```Java
// TODO(diamondm) consider using Arrays.toString() for array parameters
  public static String lenientFormat(
      @Nullable String template, @Nullable Object @Nullable... args) {
    template = String.valueOf(template); // null -> "null"

    if (args == null) {
      args = new Object[] {"(Object[])null"};
    } else {
      for (int i = 0; i < args.length; i++) {
        args[i] = lenientToString(args[i]);
      }
    }

    // start substituting the arguments into the '%s' placeholders
    StringBuilder builder = new StringBuilder(template.length() + 16 * args.length);
    int templateStart = 0;
    int i = 0;
    while (i < args.length) {
      int placeholderStart = template.indexOf("%s", templateStart);
      if (placeholderStart == -1) {
        break;
      }
      builder.append(template, templateStart, placeholderStart);
      builder.append(args[i++]);
      templateStart = placeholderStart + 2;
    }
    builder.append(template, templateStart, template.length());

    // if we run out of placeholders, append the extra args in square braces
    if (i < args.length) {
      builder.append(" [");
      builder.append(args[i++]);
      while (i < args.length) {
        builder.append(", ");
        builder.append(args[i++]);
      }
      builder.append(']');
    }

    return builder.toString();
  }
```
