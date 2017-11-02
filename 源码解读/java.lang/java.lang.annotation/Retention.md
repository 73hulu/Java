# Retention
指示具有注释类型的注释要保留多长时间，允许的值是`RetentionPolicy`的枚举值，即取值只能是`RetentionPolicy.SOURCE`、`RetentionPolicy.CLASS`、`RetentionPolicy.RUNTIME`中的一种。


```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Retention {
    /**
     * Returns the retention policy.
     * @return the retention policy
     */
    RetentionPolicy value();
}
```
