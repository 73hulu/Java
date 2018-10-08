# Functions

与接口[`google.guava.base.Function`](./Function.md)有关的提供静态方法的工具类，结构如下：

![Functions](https://ws4.sinaimg.cn/large/006tNbRwly1fw0r9mjov7j30j60jigly.jpg)


## public final class Functions {..}
类生命，被final修饰，不能被继承。

## 构造函数
既然是提供静态方法的工具类，那么最好不要提供实例化的方法，所以讲构造函数私有化。略。


## 不同的Function接口实现类
`Functions`剩下的方法都是用来实例化一类`Function`。具体来说，有以下几种实现：

| 实例类别 | 函数描述符 |
| :------------- | :------------- |
| ToStringFunction       | T -> String      |
|IdentityFunction   | T -> T  |
|FunctionForMapNoDefault   | T -> R, if  (T, R)∈ Map or throw exception|
|ForMapWithDefault   |T -> R, if  (T, R)∈ Map or defaultValue|
|FunctionComposition   | T -> U   |
|PredicateFunction   |  T -> Boolean |
|ConstantFunction   |  T -> T |
|SupplierFunction   | T -> R  |


### public static Function<Object, String> toStringFunction(){..}
实例化`ToStringFunction`的实例，定义如下：
```Java
public static Function<Object, String> toStringFunction() {
  return ToStringFunction.INSTANCE;
}

// toStringFunction
// enum singleton pattern
private enum ToStringFunction implements Function<Object, String> {
 INSTANCE;

 @Override
 public String apply(Object o) {
   checkNotNull(o); // eager for GWT.
   return o.toString();
 }

 @Override
 public String toString() {
   return "Functions.toStringFunction()";
 }
}
```
可以看到，`ToStringFunction`专门用来提供`T -> String`转化的，调用对象的`toString`方法。并且`ToStringFunction`使用枚举来创建单例。

### public static <E> Function<E, E> identity()
实例化`IdentityFunction`，定义如下：
```Java
public static <E> Function<E, E> identity() {
  return (Function<E, E>) IdentityFunction.INSTANCE;
}

// IdentityFunction
// enum singleton pattern
private enum IdentityFunction implements Function<Object, Object> {
  INSTANCE;

  @Override
  public @Nullable Object apply(@Nullable Object o) {
    return o;
  }

  @Override
  public String toString() {
    return "Functions.identity()";
  }
}
```
`IdentityFunction`同样使用枚举类实现单例。

## public static <K, V> Function<K, V> forMap(Map<K, V> map){..}
返回`FunctionForMapNoDefault`实例，故名思意，这个`Function`实例用于无默认值的的Map类型。对于给定的map和key，将返回对应的value值。注意如果map中不包含key，将会抛出异常定义如下：
```Java
public static <K, V> Function<K, V> forMap(Map<K, V> map) {
  return new FunctionForMapNoDefault<>(map);
}

// FunctionForMapNoDefault
private static class FunctionForMapNoDefault<K, V> implements Function<K, V>, Serializable {
  final Map<K, V> map;

  FunctionForMapNoDefault(Map<K, V> map) {
    this.map = checkNotNull(map);
  }

  @Override
  public V apply(@Nullable K key) {
    V result = map.get(key);
    checkArgument(result != null || map.containsKey(key), "Key '%s' not present in map", key);
    return result;
  }

  @Override
  public boolean equals(@Nullable Object o) {
    if (o instanceof FunctionForMapNoDefault) {
      FunctionForMapNoDefault<?, ?> that = (FunctionForMapNoDefault<?, ?>) o;
      return map.equals(that.map);
    }
    return false;
  }

  @Override
  public int hashCode() {
    return map.hashCode();
  }

  @Override
  public String toString() {
    return "Functions.forMap(" + map + ")";
  }

  private static final long serialVersionUID = 0;
}
```
## public static <K, V> Function<K, V> forMap(Map<K, ? extends V> map, @Nullable V defaultValue){..}
返回`ForMapWithDefault`实例。与前一个`forMap`相比的区别在于：当map中不包含key时，上一个`forMap`将抛出异常，而该重载则返回预先指定的默认值。
```Java
public static <K, V> Function<K, V> forMap(Map<K, ? extends V> map, @Nullable V defaultValue) {
  return new ForMapWithDefault<>(map, defaultValue);
}

// ForMapWithDefault
private static class ForMapWithDefault<K, V> implements Function<K, V>, Serializable {
  final Map<K, ? extends V> map;
  final @Nullable V defaultValue;

  ForMapWithDefault(Map<K, ? extends V> map, @Nullable V defaultValue) {
    this.map = checkNotNull(map);
    this.defaultValue = defaultValue;
  }

  @Override
  public V apply(@Nullable K key) {
    V result = map.get(key);
    return (result != null || map.containsKey(key)) ? result : defaultValue;
  }

  @Override
  public boolean equals(@Nullable Object o) {
    if (o instanceof ForMapWithDefault) {
      ForMapWithDefault<?, ?> that = (ForMapWithDefault<?, ?>) o;
      return map.equals(that.map) && Objects.equal(defaultValue, that.defaultValue);
    }
    return false;
  }

  @Override
  public int hashCode() {
    return Objects.hashCode(map, defaultValue);
  }

  @Override
  public String toString() {
    // TODO(cpovirk): maybe remove "defaultValue=" to make this look like the method call does
    return "Functions.forMap(" + map + ", defaultValue=" + defaultValue + ")";
  }

  private static final long serialVersionUID = 0;
}
```


### public static <A, B, C> Function<A, C> compose(Function<B, C> g, Function<A, ? extends B> f){..}
这个方法就有意思了，两个参数都是`Function`实例。它的用法类似于JDK8中`g.compose(f)`或者`g.andThen(f)`的用法。
```Java
public static <A, B, C> Function<A, C> compose(Function<B, C> g, Function<A, ? extends B> f) {
  return new FunctionComposition<>(g, f);
}

// FunctionComposition
private static class FunctionComposition<A, B, C> implements Function<A, C>, Serializable {
 private final Function<B, C> g;
 private final Function<A, ? extends B> f;

 public FunctionComposition(Function<B, C> g, Function<A, ? extends B> f) {
   this.g = checkNotNull(g);
   this.f = checkNotNull(f);
 }

 @Override
 public C apply(@Nullable A a) {
   return g.apply(f.apply(a));
 }

 @Override
 public boolean equals(@Nullable Object obj) {
   if (obj instanceof FunctionComposition) {
     FunctionComposition<?, ?, ?> that = (FunctionComposition<?, ?, ?>) obj;
     return f.equals(that.f) && g.equals(that.g);
   }
   return false;
 }

 @Override
 public int hashCode() {
   return f.hashCode() ^ g.hashCode();
 }

 @Override
 public String toString() {
   // TODO(cpovirk): maybe make this look like the method call does ("Functions.compose(...)")
   return g + "(" + f + ")";
 }

 private static final long serialVersionUID = 0;
}
```
### public static <T> Function<T, Boolean> forPredicate(Predicate<T> predicate) {..}
接受的参数是`Predicate`类型，这里完成的是T->Boolean的转换。
```Java
public static <T> Function<T, Boolean> forPredicate(Predicate<T> predicate) {
  return new PredicateFunction<T>(predicate);
}

// PredicateFunction
/** @see Functions#forPredicate */
private static class PredicateFunction<T> implements Function<T, Boolean>, Serializable {
  private final Predicate<T> predicate;

  private PredicateFunction(Predicate<T> predicate) {
    this.predicate = checkNotNull(predicate);
  }

  @Override
  public Boolean apply(@Nullable T t) {
    return predicate.apply(t);
  }

  @Override
  public boolean equals(@Nullable Object obj) {
    if (obj instanceof PredicateFunction) {
      PredicateFunction<?> that = (PredicateFunction<?>) obj;
      return predicate.equals(that.predicate);
    }
    return false;
  }

  @Override
  public int hashCode() {
    return predicate.hashCode();
  }

  @Override
  public String toString() {
    return "Functions.forPredicate(" + predicate + ")";
  }

  private static final long serialVersionUID = 0;
}
```

### public static <E> Function<Object, E> constant(@Nullable E value){..}
```Java
public static <E> Function<Object, E> constant(@Nullable E value) {
  return new ConstantFunction<E>(value);
}

private static class ConstantFunction<E> implements Function<Object, E>, Serializable {
  private final @Nullable E value;

  public ConstantFunction(@Nullable E value) {
    this.value = value;
  }

  @Override
  public E apply(@Nullable Object from) {
    return value;
  }

  @Override
  public boolean equals(@Nullable Object obj) {
    if (obj instanceof ConstantFunction) {
      ConstantFunction<?> that = (ConstantFunction<?>) obj;
      return Objects.equal(value, that.value);
    }
    return false;
  }

  @Override
  public int hashCode() {
    return (value == null) ? 0 : value.hashCode();
  }

  @Override
  public String toString() {
    return "Functions.constant(" + value + ")";
  }

  private static final long serialVersionUID = 0;
}
```

### public static <E> Function<Object, E> constant(@Nullable E value) {...}
返回`ConstantFunction`的实例，这个实例和`IdentityFunction`有点类似，永远返回自身。在JDK8中可以用lambda表达式`o -> value`来替代。
```Java
public static <E> Function<Object, E> constant(@Nullable E value) {
  return new ConstantFunction<E>(value);
}

private static class ConstantFunction<E> implements Function<Object, E>, Serializable {
  private final @Nullable E value;

  public ConstantFunction(@Nullable E value) {
    this.value = value;
  }

  @Override
  public E apply(@Nullable Object from) {
    return value;
  }

  @Override
  public boolean equals(@Nullable Object obj) {
    if (obj instanceof ConstantFunction) {
      ConstantFunction<?> that = (ConstantFunction<?>) obj;
      return Objects.equal(value, that.value);
    }
    return false;
  }

  @Override
  public int hashCode() {
    return (value == null) ? 0 : value.hashCode();
  }

  @Override
  public String toString() {
    return "Functions.constant(" + value + ")";
  }

  private static final long serialVersionUID = 0;
}
```

### public static <T> Function<Object, T> forSupplier(Supplier<T> supplier){...}
接收`Supplier`类型的参数，返回`SupplierFunction`实例。在JDK8中可以使用`o -> supplier.get()`来替代。
```Java
public static <T> Function<Object, T> forSupplier(Supplier<T> supplier) {
  return new SupplierFunction<T>(supplier);
}

/** @see Functions#forSupplier */
private static class SupplierFunction<T> implements Function<Object, T>, Serializable {

  private final Supplier<T> supplier;

  private SupplierFunction(Supplier<T> supplier) {
    this.supplier = checkNotNull(supplier);
  }

  @Override
  public T apply(@Nullable Object input) {
    return supplier.get();
  }

  @Override
  public boolean equals(@Nullable Object obj) {
    if (obj instanceof SupplierFunction) {
      SupplierFunction<?> that = (SupplierFunction<?>) obj;
      return this.supplier.equals(that.supplier);
    }
    return false;
  }

  @Override
  public int hashCode() {
    return supplier.hashCode();
  }

  @Override
  public String toString() {
    return "Functions.forSupplier(" + supplier + ")";
  }

  private static final long serialVersionUID = 0;
}
```
