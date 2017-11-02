# Documented


指示具有类型的注释默认由javadoc和类似工具来记录。 这种类型应该用于注释注释影响客户使用注释元素的类型的声明。 如果一个类型声明使用Documented进行注释，则其注释将成为注释元素的公共API的一部分。

该类型定义如下：
```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Documented {
}
```
