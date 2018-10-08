# CaseFormat

这个类主要针对全字母组成的字符串的大小写格式转化，结构如下：
![CaseFormat](https://ws2.sinaimg.cn/large/006tNc79ly1fvtwo9eonbj30iw0h63ys.jpg)

注意到，`CaseFormat`是一个枚举类，其中包含五种枚举类型：

| 形式 | 含义 |
| :------------- | :------------- |
| LOWER_CAMEL | Java 变量命名形式，即常见驼峰式, e.g., "lowerCamel".|
|LOWER_HYPHEN   |用连接符连接的变量命名形式，e.g., "lower-hyphen".   |
|LOWER_UNDERSCORE   |  C++变量命名形式, e.g., "lower_underscore". |
|UPPER_CAMEL   |  Java and C++ 类命名形式, e.g., "UpperCamel". |
|UPPER_UNDERSCORE   | Java and C++ 常量命名形式, e.g., "UPPER_UNDERSCORE". |

这里值得学习的一点在于，`CaseFormat`枚举类中包含了抽象方法`normalizeWord`和`convert`，这种写法在之前的学习和实践中还没有使用过。

> 开发者在`CharMatcher`类中皮了一下，放置了一个小彩蛋。

## 构造方法
```Java
CaseFormat(CharMatcher wordBoundary, String wordSeparator) {
  this.wordBoundary = wordBoundary;
  this.wordSeparator = wordSeparator;
}

private final CharMatcher wordBoundary;
private final String wordSeparator;
```
枚举类中包含两个变量，`wordBoundary`是`CharMatcher`实例，这个类是`Predicate`的子类，主要的作用就是用来判别某种特性的。

## public final String to(CaseFormat format, String str){...}
将字符串str按照某种格式转化。
```Java
public final String to(CaseFormat format, String str) {
 checkNotNull(format);
 checkNotNull(str);
 return (format == this) ? str : convert(format, str);
}

/** Enum values can override for performance reasons. */
String convert(CaseFormat format, String s) {
  // deal with camel conversion
  StringBuilder out = null;
  int i = 0;
  int j = -1;
  while ((j = wordBoundary.indexIn(s, ++j)) != -1) {
    if (i == 0) {
      // include some extra space for separators
      out = new StringBuilder(s.length() + 4 * wordSeparator.length());
      out.append(format.normalizeFirstWord(s.substring(i, j)));
    } else {
      out.append(format.normalizeWord(s.substring(i, j)));
    }
    out.append(format.wordSeparator);
    i = j + wordSeparator.length();
  }
  return (i == 0)
      ? format.normalizeFirstWord(s)
      : out.append(format.normalizeWord(s.substring(i))).toString();
}
```


## private String normalizeFirstWord(String word){..}
按照当前方式实现字符串首字母的格式化。
```Java
private String normalizeFirstWord(String word) {
  return (this == LOWER_CAMEL) ? Ascii.toLowerCase(word) : normalizeWord(word);
}
```
## abstract String normalizeWord(String word);
**枚举类中包含抽象方法， 此时枚举类并不需要加上`abstract`修饰词。但是枚举类的实例需要实现该方法。**



## private static String firstCharOnlyToUpper(String word){..}
将字符串转化为只有首字母大写的形式。
```Java
private static String firstCharOnlyToUpper(String word) {
return (word.isEmpty())
    ? word
    : Ascii.toUpperCase(word.charAt(0)) + Ascii.toLowerCase(word.substring(1));
}
```

## 五种`CaseFormat`的实现
本文开头列出了五种实现。

### LOWER_HYPHEN
```Java
/** Hyphenated variable naming convention, e.g., "lower-hyphen". */
LOWER_HYPHEN(CharMatcher.is('-'), "-") {
  @Override
  String normalizeWord(String word) {
    return Ascii.toLowerCase(word);
  }

  @Override
  String convert(CaseFormat format, String s) {
    if (format == LOWER_UNDERSCORE) {
      return s.replace('-', '_');
    }
    if (format == UPPER_UNDERSCORE) {
      return Ascii.toUpperCase(s.replace('-', '_'));
    }
    return super.convert(format, s);
  }
}
```

### LOWER_UNDERSCORE
```Java
/** C++ variable naming convention, e.g., "lower_underscore". */
LOWER_UNDERSCORE(CharMatcher.is('_'), "_") {
  @Override
  String normalizeWord(String word) {
    return Ascii.toLowerCase(word);
  }

  @Override
  String convert(CaseFormat format, String s) {
    if (format == LOWER_HYPHEN) {
      return s.replace('_', '-');
    }
    if (format == UPPER_UNDERSCORE) {
      return Ascii.toUpperCase(s);
    }
    return super.convert(format, s);
  }
}
```

### LOWER_CAMEL
```Java
/** Java variable naming convention, e.g., "lowerCamel". */
LOWER_CAMEL(CharMatcher.inRange('A', 'Z'), "") {
  @Override
  String normalizeWord(String word) {
    return firstCharOnlyToUpper(word);
  }
}
```

### UPPER_CAMEL
```Java
/** Java and C++ class naming convention, e.g., "UpperCamel". */
UPPER_CAMEL(CharMatcher.inRange('A', 'Z'), "") {
  @Override
  String normalizeWord(String word) {
    return firstCharOnlyToUpper(word);
  }
}
```

### UPPER_UNDERSCORE
```Java
/** Java and C++ constant naming convention, e.g., "UPPER_UNDERSCORE". */
UPPER_UNDERSCORE(CharMatcher.is('_'), "_") {
  @Override
  String normalizeWord(String word) {
    return Ascii.toUpperCase(word);
  }

  @Override
  String convert(CaseFormat format, String s) {
    if (format == LOWER_HYPHEN) {
      return Ascii.toLowerCase(s.replace('_', '-'));
    }
    if (format == LOWER_UNDERSCORE) {
      return Ascii.toLowerCase(s);
    }
    return super.convert(format, s);
  }
}
```
