# Predicate

函数式接口。唯一的抽象方法是：`boolean test(T t)`，所以函数描述符是`T -> boolean`。主要用来进行过滤。

除此之外，基于Java 8 接口可以有默认方法的特性，`Predicate`接口含有其他的方法，具体接口结构如下：

![Predicate接口](https://ws3.sinaimg.cn/large/006tNc79gy1frzkfpj22uj30i006w3zg.jpg)


## boolean test(T t);
抽象接口，测试参数是否符合某种测试条件。

## default Predicate<T> and(Predicate<? super T> other){..}
```Java
default Predicate<T> and(Predicate<? super T> other) {
    Objects.requireNonNull(other);
    return (t) -> test(t) && other.test(t);
}
```
提供了逻辑与的`Predicate`判断。注意返回的是`Predicate`类型对象，所以return语句的断句是这样的`(t) -> (test(t) && other.test(t))`，由于该特性，所以可以进行链式运算。
例如：
```Java
Predicate<String> startWithJ = name -> name.startsWith("J");
Predicate<String> lengthLimit4 = name -> name.length() <= 4;
Predicate<String> endWithE = name -> name.endsWith("e");

List<String> names = new ArrayList<>(Arrays.asList("James", "Bao", "Jane", "You"));
names = names.stream().filter(startWithJ.and(lengthLimit4).and(endWithE)).collect(Collectors.toList());
System.out.println(Arrays.toString(names.toArray(new String[names.size()]))); // [Jane]
```

同样的道理，`Predicate`还提供了对于`or`逻辑、`xor`逻辑和`not`逻辑的运算。

## default Predicate<T> or(Predicate<? super T> other){..}

```Java
default Predicate<T> or(Predicate<? super T> other) {
    Objects.requireNonNull(other);
    return (t) -> test(t) || other.test(t);
}
```

## default Predicate<T> negate(){..}
```Java
default Predicate<T> negate(){
  return (t) -> !test();
}
```

## static <T> Predicate<T> isEqual(Object targetRef) {...}
返回一个判断对象是否相等的`Predicate`接口对象。
```Java
static <T> Predicate<T> isEqual(Object targetRef) {
    return (null == targetRef)
            ? Objects::isNull
            : object -> targetRef.equals(object);
}
```
