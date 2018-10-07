# StringJoiner


`StringJoiner`用来进行格式化的字符串拼接，可以指定分隔符、字符串前缀和字符串后缀。结构如下：

![StringJoiner](https://ws2.sinaimg.cn/large/006tNbRwly1fvwc2nyg21j30js0f074h.jpg)

## 构造函数
创建时指定分隔符、字符串前缀和后缀，缺省前缀和后缀为空字符串。
```Java
public StringJoiner(CharSequence delimiter) {
    this(delimiter, "", "");
}

public StringJoiner(CharSequence delimiter,
                        CharSequence prefix,
                        CharSequence suffix) {
    Objects.requireNonNull(prefix, "The prefix must not be null");
    Objects.requireNonNull(delimiter, "The delimiter must not be null");
    Objects.requireNonNull(suffix, "The suffix must not be null");
    // make defensive copies of arguments
    this.prefix = prefix.toString();
    this.delimiter = delimiter.toString();
    this.suffix = suffix.toString();
    this.emptyValue = this.prefix + this.suffix;
}
private final String prefix;
private final String delimiter;
private final String suffix;
```

## public StringJoiner setEmptyValue(CharSequence emptyValue) {..}
`emptyValue`是默认值，如果没有调用该方法进行设定，默认为`prefix + suffix`。
```Java
public StringJoiner setEmptyValue(CharSequence emptyValue) {
    this.emptyValue = Objects.requireNonNull(emptyValue,
        "The empty value must not be null").toString();
    return this;
}
private String emptyValue;
```
## public StringJoiner add(CharSequence newElement) {..}
往`StringJoiner`中添加元素时候调用该方法，大致分为两部分，首先添加分隔符，然后添加待拼接字符。定义如下：
```Java
public StringJoiner add(CharSequence newElement) {
    prepareBuilder().append(newElement);
    return this;
}

private StringBuilder prepareBuilder() {
    if (value != null) {
        value.append(delimiter);
    } else {
        value = new StringBuilder().append(prefix);
    }
    return value;
}
```
可以看到，`add`方法写的比较精巧。将分隔符和prefix的添加过程单独封装。当`StringJoiner`初始被使用的时候才进行内存分配，所谓懒加载。

## public String toString(){..}
使用`StringJoiner`获取结果值的方法。
```Java
@Override
public String toString() {
    if (value == null) {
        return emptyValue;
    } else {
        if (suffix.equals("")) {
            return value.toString();
        } else {
            int initialLength = value.length();
            String result = value.append(suffix).toString();
            // reset value to pre-append initialLength
            value.setLength(initialLength);
            return result;
        }
    }
}
private StringBuilder value;
```
从上面的代码可以看到，`StringJoiner`的底层结构`StringBuilder`中，保存了字符串前缀、分隔符和待拼接字符串。等到`toString`返回的时候才会将`suffix`拼接上。并且**得到最后结果后，通过设置stringBuilder长度的方式，将字符串后缀去除。**

## public StringJoiner merge(StringJoiner other) {..}
和其他的`StringJoiner`合并。注意该操作将舍弃`other`的`prefix`和`suffix`，保有待拼接子字符串和分隔符。
```Java
public StringJoiner merge(StringJoiner other) {
    Objects.requireNonNull(other);
    if (other.value != null) {
        final int length = other.value.length();
        // lock the length so that we can seize the data to be appended
        // before initiate copying to avoid interference, especially when
        // merge 'this'
        StringBuilder builder = prepareBuilder();
        builder.append(other.value, other.prefix.length(), length);
    }
    return this;
}
```

## public int length(){..}
返回内容的长度，这里将会返回"prefix + text + delimiter + suffix"的总长度，如果根本没有text（即add方法没有被调用过），那么将会返回默认值`emptyValue`的长度。虽然这样，但是`suffix`并没有真正被添加到`value`中，只有`toString`方法被调用的时候才会被添加。
```Java
public int length() {
    // Remember that we never actually append the suffix unless we return
    // the full (present) value or some sub-string or length of it, so that
    // we can add on more if we need to.
    return (value != null ? value.length() + suffix.length() :
            emptyValue.length());
}
```

> `StringJoiner`的作用就是格式化拼接字符串，优点在于指定前缀和后缀。同样效果的还有`Collectors`中`joining()`和`Google Guava`中的`Joiner`。
