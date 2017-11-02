# Arrays

Arrays类是Java集合框架的一部分，提供了对数组的处理方法（如查找和排序）和与线性表的转换方法。

该类的方法列表太长了，多是一些重载的方法所以这里就不截图了。下面只列出一些常用的方法。


### 构造方法
构造方法是私有的，目的是为了覆盖默认的公有的构造方法，不让外部调用。
```Java
private Arrays() {}
```

### sort方法
重载了14个sort方法，支持对不同基本数据类型数组（eg. int[], double[], float[], long[]）和范围（[left, right]，无这两个参数的表示对全数组进行排序）。下面就一个典型例子进行讲解。

```Java
public static void sort(int[] a, int fromIndex, int toIndex) {
    rangeCheck(a.length, fromIndex, toIndex);
    DualPivotQuicksort.sort(a, fromIndex, toIndex - 1, null, 0, 0);
}
```
这是对元素为int类型的数组进行部分排序的sort方法。方法首先对范围参数进行了检查，`rangeCheck`方法定义如下：
```Java
private static void rangeCheck(int arrayLength, int fromIndex, int toIndex) {
    if (fromIndex > toIndex) {
        throw new IllegalArgumentException(
                "fromIndex(" + fromIndex + ") > toIndex(" + toIndex + ")");
    }
    if (fromIndex < 0) {
        throw new ArrayIndexOutOfBoundsException(fromIndex);
    }
    if (toIndex > arrayLength) {
        throw new ArrayIndexOutOfBoundsException(toIndex);
    }
}
```
当范围参数不合法的时候会抛出`IllegalArgumentException`或`ArrayIndexOutOfBoundsException`。

接下来就是真正排序的时候了。在1.7之前，`Arrays.sort`使用的是我们普通的快速排序方法。但是在1.7之后使用的是`DualPivotQuicksort`中的`sort`方法。该类名翻译过来就是“双轴快速排序”，顾名思义即基于两个轴来进行比较，而普通的快排是选择一个点来作为轴点。该方法的具体实现比较复杂，请参考`DualPivotQuicksort`类的说明。

###
