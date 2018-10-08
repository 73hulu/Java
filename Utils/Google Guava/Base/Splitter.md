# Splitter
`Splitter`提供了与`Joiner`相反的操作，即分割字符串。在JDK中`String`提供实例方法`split`同样用于分割字符串，但是与该方法相比，`Splitter`提供了更加丰富的定制服务。


类结构如下：

![Splitter](https://ws3.sinaimg.cn/large/006tNc79ly1fvubk11n3hj30iw0si0t9.jpg)


## 构造方法
```Java
private final CharMatcher trimmer;
private final boolean omitEmptyStrings;
private final Strategy strategy;
private final int limit;

private Splitter(Strategy strategy) {
  this(strategy, false, CharMatcher.none(), Integer.MAX_VALUE);
}

private Splitter(Strategy strategy, boolean omitEmptyStrings, CharMatcher trimmer, int limit) {
  this.strategy = strategy;
  this.omitEmptyStrings = omitEmptyStrings;
  this.trimmer = trimmer;
  this.limit = limit;
}
```
与`Joiner`类似，`Splitter`的构造方法同样是私有的，那么一定有一个静态方法提供类的实例化。在构造方法中可以看到，主要对四个参数进行赋值：
* trimmer：结果集中字符串两侧去除该匹配子串，常用的是两边去空格。
* omitEmptyStrings： 是否在结构中省略空白字符串，默认为false，即不省略，在后面的`omitEmptyStrings`方法中可以改写该参数。
* strategy: 分割策略。`Strategy`类型，是`Splitter`的内部结构，定义如下：
```Java
private interface Strategy {
  Iterator<String> iterator(Splitter splitter, CharSequence toSplit);
}
```
抽象方法的其中一个参数居然是`Splitter`类型，这难道不是构成了递归？？？ 有趣，看下文如何构造吧。
* limit：整型，猜得出大约是用来限制结果的长度。默认`Integer.MAX_VALUE`，所以默认情况下会显示所有结果，与`String`中`split`中第二个参数为负数的时候含义一致。


这里注意到，四个变量都是`final`修饰，所以`Splitter`和`Joiner`一样都可以作为常量使用。

## on方法
这就是用来实例化`Splitter`的静态方法，支持单个字符、字符串和正则表达式作为分隔符。有四个重载：

### public static Splitter on(char separator) {..}
以单个字符进行分割。定义如下：
```Java
public static Splitter on(char separator) {
  return on(CharMatcher.is(separator));
}
```
构造一个`CharMatcher`实例，调用下一个重载方法。

### public static Splitter on(final CharMatcher separatorMatcher) {..}
以`CharMatcher`实例为参数，返回`Splitter`的实例。在`Splitter`的构造方法中可以看到，至少需要提供`Strategy`的是作为构造函数的参数。而`Strategy`的参数又以`Splitter`为参数，这貌似形成了递归，那么`on`方法如何结果这个问题？

```Java
public static Splitter on(final CharMatcher separatorMatcher) {
  checkNotNull(separatorMatcher);

  return new Splitter(
      new Strategy() {
        @Override
        public SplittingIterator iterator(Splitter splitter, final CharSequence toSplit) {
          return new SplittingIterator(splitter, toSplit) {
            @Override
            int separatorStart(int start) {
              return separatorMatcher.indexIn(toSplit, start);
            }

            @Override
            int separatorEnd(int separatorPosition) {
              return separatorPosition + 1;
            }
          };
        }
      });
}
```
`Strategy`的抽象方法`iterator`需要返回一个迭代器，这里返回的是一个迭代器的子类`SplittingIterator`的子类。定义如下：

```Java
private abstract static class SplittingIterator extends AbstractIterator<String> {
  final CharSequence toSplit;
  final CharMatcher trimmer;
  final boolean omitEmptyStrings;

  /**
   * Returns the first index in {@code toSplit} at or after {@code start} that contains the
   * separator.
   */
  abstract int separatorStart(int start);

  /**
   * Returns the first index in {@code toSplit} after {@code separatorPosition} that does not
   * contain a separator. This method is only invoked after a call to {@code separatorStart}.
   */
  abstract int separatorEnd(int separatorPosition);

  int offset = 0;
  int limit;

  protected SplittingIterator(Splitter splitter, CharSequence toSplit) {
    this.trimmer = splitter.trimmer;
    this.omitEmptyStrings = splitter.omitEmptyStrings;
    this.limit = splitter.limit;
    this.toSplit = toSplit;
  }

  @Override
  protected String computeNext() {
    /*
     * The returned string will be from the end of the last match to the beginning of the next
     * one. nextStart is the start position of the returned substring, while offset is the place
     * to start looking for a separator.
     */
    int nextStart = offset;
    while (offset != -1) {
      int start = nextStart;
      int end;

      int separatorPosition = separatorStart(offset);
      if (separatorPosition == -1) {
        end = toSplit.length();
        offset = -1;
      } else {
        end = separatorPosition;
        offset = separatorEnd(separatorPosition);
      }
      if (offset == nextStart) {
        /*
         * This occurs when some pattern has an empty match, even if it doesn't match the empty
         * string -- for example, if it requires lookahead or the like. The offset must be
         * increased to look for separators beyond this point, without changing the start position
         * of the next returned substring -- so nextStart stays the same.
         */
        offset++;
        if (offset > toSplit.length()) {
          offset = -1;
        }
        continue;
      }

      while (start < end && trimmer.matches(toSplit.charAt(start))) {
        start++;
      }
      while (end > start && trimmer.matches(toSplit.charAt(end - 1))) {
        end--;
      }

      if (omitEmptyStrings && start == end) {
        // Don't include the (unused) separator in next split string.
        nextStart = offset;
        continue;
      }

      if (limit == 1) {
        // The limit has been reached, return the rest of the string as the
        // final item. This is tested after empty string removal so that
        // empty strings do not count towards the limit.
        end = toSplit.length();
        offset = -1;
        // Since we may have changed the end, we need to trim it again.
        while (end > start && trimmer.matches(toSplit.charAt(end - 1))) {
          end--;
        }
      } else {
        limit--;
      }

      return toSplit.subSequence(start, end).toString();
    }
    return endOfData();
  }
}
```
很神奇不是？！这里可以好好体会一下迭代器设计模式的设计思想。

另外，对于`CharMatcher`这个类需要好好理解一下，它的一些设计实际上类似于正则，例如
```Java
Splitter.on(CharMatcher.anyOf(";,")).split("foo,;bar,quux") // 返回["foo", "", "bar", "quux"]
```
这里`CharMatcher.anyOf`就相当于正则中的“|”符号。

### public static Splitter on(final String separator){..}
使用`String`类型作为分割符，定义如下：
```Java
public static Splitter on(final String separator) {
  checkArgument(separator.length() != 0, "The separator may not be the empty string.");
  if (separator.length() == 1) {
    return Splitter.on(separator.charAt(0));
  }
  return new Splitter(
      new Strategy() {
        @Override
        public SplittingIterator iterator(Splitter splitter, CharSequence toSplit) {
          return new SplittingIterator(splitter, toSplit) {
            @Override
            public int separatorStart(int start) {
              int separatorLength = separator.length();

              positions:
              for (int p = start, last = toSplit.length() - separatorLength; p <= last; p++) {
                for (int i = 0; i < separatorLength; i++) {
                  if (toSplit.charAt(i + p) != separator.charAt(i)) {
                    continue positions;
                  }
                }
                return p;
              }
              return -1;
            }

            @Override
            public int separatorEnd(int separatorPosition) {
              return separatorPosition + separator.length();
            }
          };
        }
      });
}
```
这里特别注意空格，例如：
```Java
Splitter.on(", ").split("foo, bar,baz")// 将返回 ["foo", "bar,baz"]
```

### public static Splitter on(Pattern separatorPattern) {..}
支持正则表达式作为分隔符，其实现机制是调用了JDK中`Pattern`和`Matcher`的匹配方法。
```java
@GwtIncompatible // java.util.regex
public static Splitter on(Pattern separatorPattern) {
  return on(new JdkPattern(separatorPattern));
}

private static Splitter on(final CommonPattern separatorPattern) {
  checkArgument(
      !separatorPattern.matcher("").matches(),
      "The pattern may not match the empty string: %s",
      separatorPattern);

  return new Splitter(
      new Strategy() {
        @Override
        public SplittingIterator iterator(final Splitter splitter, CharSequence toSplit) {
          final CommonMatcher matcher = separatorPattern.matcher(toSplit);
          return new SplittingIterator(splitter, toSplit) {
            @Override
            public int separatorStart(int start) {
              return matcher.find(start) ? matcher.start() : -1;
            }

            @Override
            public int separatorEnd(int separatorPosition) {
              return matcher.end();
            }
          };
        }
      });
}
```
例如：
```Java
// 在windows或linux中，根据换行符分割文件内容
Splitter.on(Pattern.compile("\r?\n")).split(entireFile)
```

## public static Splitter fixedLength(final int length){..}
用于限定**每个被分割得到的元素的长度**，注意这里指每个元素的长度，而不是元素的个数。定义如下：
```Java
public static Splitter fixedLength(final int length) {
  checkArgument(length > 0, "The length may not be less than 1");

  return new Splitter(
      new Strategy() {
        @Override
        public SplittingIterator iterator(final Splitter splitter, CharSequence toSplit) {
          return new SplittingIterator(splitter, toSplit) {
            @Override
            public int separatorStart(int start) {
              int nextChunkStart = start + length;
              return (nextChunkStart < toSplit.length() ? nextChunkStart : -1);
            }

            @Override
            public int separatorEnd(int separatorPosition) {
              return separatorPosition;
            }
          };
        }
      });
}
```
例如：
```java
 Splitter.fixedLength(2).split("abcde"); // 将放回["ab", "cd", "e"]
```

那么，如果是限制结果元素的个数的话，如何实现呢？那就是下一个方法`limit`的功能。


## public Splitter limit(int limit){..}
用来限制结果元素的个数。定义如下：
```Java
public Splitter limit(int limit) {
  checkArgument(limit > 0, "must be greater than zero: %s", limit);
  return new Splitter(strategy, omitEmptyStrings, trimmer, limit);
}
```
需要注意的是，分割结果并不会抛弃元素，会将“本来应该抛弃”的元素合并到结果集合的最后一个元素中，即:
```Java
Splitter.on(',').limit(3).split("a,b,c,d"); // 将返回 ["a", "b", "c,d"]
```
如果在`Splitter`的定制中排除了空串，这些空串是不会被计算在内的，例如：
```Java
Splitter.on(',').limit(3).omitEmptyStrings().split("a,,,b,,,c,d"); // 将返回 ["a", "b", "c,d"]
```
如果使用了`trimResults()`方法，那么结果集中元素还是会执行`trimResults`方法的，例如：
```Java
Splitter.on(',').limit(3).trimResults().split(" a , b , c , d "); // 将返回 ["a", "b", "c , d"]
```


## public Splitter omitEmptyStrings() {..}
```java
public Splitter omitEmptyStrings() {
  return new Splitter(strategy, true, trimmer, limit);
}
```
用于去除结果中的空字符串。

## trimResults方法
结果集中字符串去除两侧匹配子串。无参的`trimResults`方法是去空格，有参数方法指定匹配子串。
```Java
public Splitter trimResults() {
  return trimResults(CharMatcher.whitespace());
}

// TODO(kevinb): throw if a trimmer was already specified!
public Splitter trimResults(CharMatcher trimmer) {
  checkNotNull(trimmer);
  return new Splitter(strategy, omitEmptyStrings, trimmer, limit);
}
```

## public Iterable<String> split(final CharSequence sequence){..}
之前的所有方法都在创建和自定义`Splitter`特性，`split`真正执行字符串分割的方法。定义如下：
```Java
public Iterable<String> split(final CharSequence sequence) {
  checkNotNull(sequence);

  return new Iterable<String>() {
    @Override
    public Iterator<String> iterator() {
      return splittingIterator(sequence);
    }

    @Override
    public String toString() {
      return Joiner.on(", ")
          .appendTo(new StringBuilder().append('['), this)
          .append(']')
          .toString();
    }
  };
}
```
而实际上，它返回的是一个迭代器：
```Java
private Iterator<String> splittingIterator(CharSequence sequence) {
  return strategy.iterator(this, sequence);
}
```

如何返回一个序列呢？很好办，有了迭代器，创建一个容器即可，这就是下面`splitToList`方法。

##  public List<String> splitToList(CharSequence sequence){..}
和`split`方法不同的是，该方法以序列的形式返回结果。定义如下：
```Java
@Beta
public List<String> splitToList(CharSequence sequence) {
  checkNotNull(sequence);

  Iterator<String> iterator = splittingIterator(sequence);
  List<String> result = new ArrayList<>();

  while (iterator.hasNext()) {
    result.add(iterator.next());
  }

  return Collections.unmodifiableList(result);
}
```
实现过程很简单，但是注意返回的序列是一个不可变序列。
## withKeyValueSeparator
针对`Map`实例的分割。暂且不提。
