# Target

指示注释类型适用的上下文。 注释类型可能适用的声明上下文和类型上下文在JLS 9.6.4.1中指定，并且在源代码中用java.lang.annotation.ElementType的枚举常量表示。
如果注解类型T中不存在@Target元注释，则类型T的注释可以被写为除了`TYPE_ELEMENT`声明之外的任何声明的修饰符。

该注释定义如下：
```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Target {
    /**
     * Returns an array of the kinds of elements an annotation type
     * can be applied to.
     * @return an array of the kinds of elements an annotation type
     * can be applied to
     */
    ElementType[] value();
}
```

例如，如果一个注释的`@Target`限定该注释可以修饰的类型是`ElementType.ANNOTATION_TYPE`，那么该注释就是”元注释”能够修饰其他注释。

如果注释需要注释多种类型，可以用下面这种写法。
```java
@Target({ElementType.FIELD, ElementType.METHOD, ElementType.FIELD})
public @interface Bogus {
   ...
}
```

该注解类有一个方法，`value`方法用来返回给注解能修饰的类型数组。
