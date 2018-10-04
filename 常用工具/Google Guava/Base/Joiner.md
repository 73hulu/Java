# Joiner

用来串联一些文字（例如数组、可迭代集合，map元素等）。通常可以返回`Joiner`本身或者一个最终结果的字符串。

![Joiner](https://ws1.sinaimg.cn/large/006tNc79ly1fvty3vqjzij30iy0qi74q.jpg)


## 构造函数
```Java
private final String separator;

private Joiner(String separator) {
this.separator = checkNotNull(separator);
}

private Joiner(Joiner prototype) {
this.separator = prototype.separator;
}
```
可以看到，类中变量只要是`seperator`，即拼接字符串时候的分隔符，该变量用`final`修饰，即`immutable`，这使得`Joiner`线程安全，可以用作`static final`常量。

> 官方解释： joiner instances are always immutable. The joiner configuration methods will always return a new Joiner, which you must use to get the desired semantics. This makes any Joiner thread safe, and usable as a static final constant.
> 这一引申出一个问题，什么样的变量适合作为常量？



## on
`on`是真正创建`Joiner`实例的静态方法，接收`seperator`参数，有两个重载：
```Java
public static Joiner on(String separator) {
   return new Joiner(separator);
 }

 /** Returns a joiner which automatically places {@code separator} between consecutive elements. */
 public static Joiner on(char separator) {
   return new Joiner(String.valueOf(separator));
 }
```

## join
join方法用来拼接元素，重载了过个方法，底层调用的是`append`方法
```Java
public final String join(Iterable<?> parts) {
  return join(parts.iterator());
}

public final String join(Iterator<?> parts) {
  return appendTo(new StringBuilder(), parts).toString();
}

public final String join(Object[] parts) {
  return join(Arrays.asList(parts));
}

public final String join(@Nullable Object first, @Nullable Object second, Object... rest) {
  return join(iterable(first, second, rest));
}
```

其中`appendTo`方法是整个`Joiner`类的核心：
```Java
public <A extends Appendable> A appendTo(A appendable, Iterator<?> parts) throws IOException {
  checkNotNull(appendable);
  if (parts.hasNext()) {
    appendable.append(toString(parts.next()));
    while (parts.hasNext()) {
      appendable.append(separator);
      appendable.append(toString(parts.next()));
    }
  }
  return appendable;
}
```
过程比较简单。

## public Joiner skipNulls(){...}
在看文档的时候发现了这样的用法：
```Java
Joiner joiner = Joiner.on("; ").skipNulls();
 . . .
return joiner.join("Harry", null, "Ron", "Hermione"); // Harry; Ron; Hermione
```
当时就觉得奇怪，`on`中调用了`Joiner`的构造方式，对final变量赋值，而且`Joiner`类中只有这一个成员变量，那么`skipNulls`的这个功能是如何实现的呢？

看到源码发现，原来`skipNulls`方法返回了一个新的`Joiner`，其中对`appendTo`方法做了重新定义，相当于实现了一个`Joiner`的匿名子类。定义如下：
```Java
/**
 * Returns a joiner with the same behavior as this joiner, except automatically skipping over any
 * provided null elements.
 */
public Joiner skipNulls() {
  return new Joiner(this) {
    @Override
    public <A extends Appendable> A appendTo(A appendable, Iterator<?> parts) throws IOException {
      checkNotNull(appendable, "appendable");
      checkNotNull(parts, "parts");
      while (parts.hasNext()) {
        Object part = parts.next();
        if (part != null) {
          appendable.append(Joiner.this.toString(part));
          break;
        }
      }
      while (parts.hasNext()) {
        Object part = parts.next();
        if (part != null) {
          appendable.append(separator);
          appendable.append(Joiner.this.toString(part));
        }
      }
      return appendable;
    }

    @Override
    public Joiner useForNull(String nullText) {
      throw new UnsupportedOperationException("already specified skipNulls");
    }

    @Override
    public MapJoiner withKeyValueSeparator(String kvs) {
      throw new UnsupportedOperationException("can't use .skipNulls() with maps");
    }
  };
}
```

注意到，用这种方式实现的`Joiner`不能使用`useForNull`和`withKeyValueSeparator`方法。


这里需要明白，`Joiner`实例是不可修改的，`skipNulls`方法将返回一个新实例的引用。所以下面这种写法将抛出无法达到预期效果：
```Java
// Bad! Do not do this!
Joiner joiner = Joiner.on(',');
joiner.skipNulls(); // does nothing!
return joiner.join("wrong", null, "wrong"); // 这里不会将null值过滤
```
## public Joiner useForNull(final String nullText) {..}
```Java
/**
 * Returns a joiner with the same behavior as this one, except automatically substituting {@code
 * nullText} for any provided null elements.
 */
public Joiner useForNull(final String nullText) {
  checkNotNull(nullText);
  return new Joiner(this) {
    @Override
    CharSequence toString(@Nullable Object part) {
      return (part == null) ? nullText : Joiner.this.toString(part);
    }

    @Override
    public Joiner useForNull(String nullText) {
      throw new UnsupportedOperationException("already specified useForNull");
    }

    @Override
    public Joiner skipNulls() {
      throw new UnsupportedOperationException("already specified useForNull");
    }
  };
}
```

## withKeyValueSeparator
用来返回`MapJoiner`实例，这个实例用来实现`Map`实例的字符串拼接。这个类的实现比较简单，这里不再赘述，下面是他的使用示例：

```Java
MapJoiner j = Joiner.on(";").withKeyValueSeparator(":");
assertEquals("", j.join(ImmutableMultimap.of().entries()));
assertEquals("", j.join(ImmutableMultimap.of().entries().iterator()));
assertEquals(":", j.join(ImmutableMultimap.of("", "").entries()));
assertEquals(":", j.join(ImmutableMultimap.of("", "").entries().iterator()));
assertEquals("1:a;1:b", j.join(ImmutableMultimap.of("1", "a", "1", "b").entries()));
assertEquals("1:a;1:b", j.join(ImmutableMultimap.of("1", "a", "1", "b").entries().iterator()));
```
