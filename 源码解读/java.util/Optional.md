# Optional
Java8中引入这个类的作用就是为了解决NPE问题，可以说是非常有用了。

注意，这是一个类，而不是一个接口。
![Optional](https://ws4.sinaimg.cn/large/006tKfTcgy1fs4ywpamifj30pw0lqq6x.jpg)

## public final class Optional<T>
类声明，被final修饰，不能被继承和修改。

## 构造方法
重载了两个构造方法，并且都是**私有**的。
```Java
private Optional() {
    this.value = null;
}

private Optional(T value) {
    this.value = Objects.requireNonNull(value);
}
```
其中的`value`为该类的私有属性`private final T value;`，同样被`final`修饰，只能赋值一次。

构造方法是私有的，我们如何取得`Optional`的对象呢？接着往下读。

## public static<T> Optional<T> empty() {..}
用来获取一个空的`Optional`对象，定义如下：
```Java
public static<T> Optional<T> empty() {
    @SuppressWarnings("unchecked")
    Optional<T> t = (Optional<T>) EMPTY;
    return t;
}
private static final Optional<?> EMPTY = new Optional<>();
```
可见，该方法返回的是在类加载之初就创建的静态常量`EMPTY`，此时`value`的值为`null`。

## public static <T> Optional<T> of(T value){..}
这个方法用来获取指定value的`Optional`对象。
```Java
public static <T> Optional<T> of(T value) {
    return new Optional<>(value);
}
```
可见，这个方法调用的是私有的构造方法，而这个构造方法中又调用了`Objects.requireNonNull`方法。如果此时参数`value=null`，则会抛出NPE。

所以至此我们大概也明白了，`empty`方法是专门用来获取`value=null`的`Optional`对象，而`of`是专门用来获取非`null`的指定value的`Optional`对象。

## public static <T> Optional<T> ofNullable(T value) {..}
上面的`of`方法在参数为null的时候会抛出NPE，`ofNullable`允许参数为null。实际上，底层分别调用了`empty`和`of`方法。
```Java
public static <T> Optional<T> ofNullable(T value) {
    return value == null ? empty() : of(value);
}
```

## public T get() {..}
用来获取非null的value值，如果value本身为空就会抛出NPE。
```Java
public T get() {
    if (value == null) {
        throw new NoSuchElementException("No value present");
    }
    return value;
}
```

## public boolean isPresent() {..}
查看value是否存在，“存在”的含义就是“是否为null”。
```Java
public boolean isPresent() {
    return value != null;
}
```
## public void ifPresent(Consumer<? super T> consumer) {..}
这就是我们经常用到的方法了，只有当value存在的时候才进行数据消费，所以不会抛出异常。
```Java
public void ifPresent(Consumer<? super T> consumer) {
    if (value != null)
        consumer.accept(value);
}
```

## public Optional<T> filter(Predicate<? super T> predicate){..}
该方法对于`Optional`进行过滤，只有当value满足非null且满足某种断言时候，才返回自己本身，否则将返回空`Optional`。
```Java
public Optional<T> filter(Predicate<? super T> predicate) {
    Objects.requireNonNull(predicate);
    if (!isPresent())
        return this;
    else
        return predicate.test(value) ? this : empty();
}
```

##  public&lt;U&gt; Optional&lt;U&gt; map(Function<? super T, ? extends U> mapper) {..}  和  public&lt;U&gt; Optional&lt;U&gt; flatMap(Function&lt;? super T, Optional&lt;U&gt;&gt; mapper){..}
当value存在时，对齐进行某种处理，返回`Optional`对象，其value值与函数式接口处理结果有关。
```Java
public<U> Optional<U> map(Function<? super T, ? extends U> mapper) {
    Objects.requireNonNull(mapper);
    if (!isPresent())
        return empty();
    else {
        return Optional.ofNullable(mapper.apply(value));
    }
}

public<U> Optional<U> flatMap(Function<? super T, Optional<U>> mapper) {
    Objects.requireNonNull(mapper);
    if (!isPresent())
        return empty();
    else {
        return Objects.requireNonNull(mapper.apply(value));
    }
}
```
这里需要注意下`map`和`flatMap`两者参数的不同，前者需要的参数是函数描述符为(T -> R)的函数型接口，而后者需要的是函数描述符为(T -> Optional<U>)的函数型接口。

## public T orElse(T other) {..}
当value为null的时候允许指定参数进行替换。
```Java
public T orElse(T other) {
    return value != null ? value : other;
}
```

##  public T orElseGet(Supplier<? extends T> other) {..}
当value为null的时候，指定其他get方法获取到value值。
```Java
public T orElseGet(Supplier<? extends T> other) {
    return value != null ? value : other.get();
}
```

## public <X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier) throws X {..}
当value值为null的时候允许抛出指定的异常。
```Java
public <X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier) throws X {
if (value != null) {
    return value;
} else {
    throw exceptionSupplier.get();
}
```

## public boolean equals(Object obj) {..}
判断两个`Optional`是否相等。
```Java
@Override
public boolean equals(Object obj) {
    if (this == obj) {
        return true;
    }

    if (!(obj instanceof Optional)) {
        return false;
    }

    Optional<?> other = (Optional<?>) obj;
    return Objects.equals(value, other.value);
}
```
## 总结
说了这么多的方法，有几个关键的地方总结一下：
1. 创建`Optional`： 通过使用`empty`、`of`和`ofNullable`分别创建指定空，指定非空value和value任意的`Optional`对象。
2. 使用`Optional`： 常使用方法`ifPresent`来避免NPE。例如：
```JAVA
User user = new User("aaa@mail.com");
Optional<User> opt = Optional.of(user);
opt.ifPresent( u -> assertEquals(user.getEmail(), u.getEmail()));
```
上面这段代码，只有当u存在，即 `u!= null`的时候，后面的代码才会执行，也就避免了由于u为null造成的NPE问题。
3. 返回默认值： 使用方法`orElse`或`orElseGet`方法来返回默认值：
```JAVA
User result = Optional.ofNullable(user).orElseGet( () -> user2);
```
上面这段代码，当user为null的时候，将返回user2作为替换。
`orElse()`和`orElseGet()`方法还存在差异，具体实验如下：
首先看下`Optional`为空时候两者的行为：
```JAVA
@Test
public void givenEmptyValue_whenCompare_thenOk() {
    User user = null
    logger.debug("Using orElse");
    User result = Optional.ofNullable(user).orElse(createNewUser());
    logger.debug("Using orElseGet");
    User result2 = Optional.ofNullable(user).orElseGet(() -> createNewUser());
}
private User createNewUser() {
    logger.debug("Creating New User");
    return new User("extra@gmail.com", "1234");
}
```
以上代码运行后都会调用`createNewUser`方法并创建一个对象，输出结果如下：
```java
Using orElse
Creating New User
Using orElseGet
Creating New User
```
可以看到，当对象为null的时候，`orElse`和`orElseGet`并无不同。
下面测试当`Optional`不为空时候的情况：
```JAVA
@Test
public void givenPresentValue_whenCompare_thenOk() {
    User user = new User("john@gmail.com", "1234");
    logger.info("Using orElse");
    User result = Optional.ofNullable(user).orElse(createNewUser());
    logger.info("Using orElseGet");
    User result2 = Optional.ofNullable(user).orElseGet(() -> createNewUser());
}
```
发现打印结果如下：
```JAVA
Using orElse
Creating New User
Using orElseGet
```
在这个示例中，`Optional`对象包含非null值，两个方法都会返回这个值，只是`orElse`仍旧还会创建对象，与之相关，`orElseGet`方法就不会再创建对象。
所以基于性能的考虑，当方法调用比较密集的时候，比如Web服务或查询数据，我们最好还是使用`orElseGet`方法比较好。
4. 抛出异常： 使用`orElseThrow`来抛出默认异常，例如：
```JAVA
@Test(expected = IllegalArgumentException.class)
public void whenThrowException_thenOk() {
    User result = Optional.ofNullable(user)
      .orElseThrow( () -> new IllegalArgumentException());
}
```
以上，当user为null的时候就会抛出`IllegalArgumentException`， 这样我们就避免单调地只抛出NPE了。
5. 转换值： 方法`map`、`flatMap`和`filter`是用来转化值的，前两者将map的结果包装成一个新的`Optional`，最后一个则断言`Optional`能够通过某种测试。
下面是一个`map`方法的例子：
```JAVA
@Test
public void whenMap_thenOk() {
    User user = new User("anna@gmail.com", "1234");
    String email = Optional.ofNullable(user)
      .map(u -> u.getEmail()).orElse("default@gmail.com");
    assertEquals(email, user.getEmail());
}
```
上面的例子中，`map`的执行结果将被包装成一个`Optional`，然后进行接下来的链式调用`orElse`。对于`flatMap`来说，要特别注意函数描述符为(T -> Optional<U>)，所以我们可以给`User`添加一个方法，用来返回`Optional`：
```JAVA
public class User {    
    private String position;
    public Optional<String> getPosition() {
        return Optional.ofNullable(position);
    }
    //...
}
```
既然 `getter `方法返回 `String` 值的 `Optional`，你可以在对 `User` 的 `Optional` 对象调用 `flatMap()` 时，用它作为参数。其返回的值是解除包装的 `String` 值：
```JAVA
@Test
public void whenFlatMap_thenOk() {
    User user = new User("anna@gmail.com", "1234");
    user.setPosition("Developer");
    String position = Optional.ofNullable(user)
      .flatMap(u -> u.getPosition()).orElse("default");

    assertEquals(position, user.getPosition().get());
}
```
`filter`方法比较简单，就是用来过滤对象属性，比较简单，下面是一个根据邮箱验证接受或拒绝用户的示例：
```JAVA
@Test
public void whenFilter_thenOk() {
    User user = new User("anna@gmail.com", "1234");
    Optional<User> result = Optional.ofNullable(user)
      .filter(u -> u.getEmail() != null && u.getEmail().contains("@"));

    assertTrue(result.isPresent());
}
```
## 综合示例
`Optional`的使用能够避免NPE异常且让代码变得优雅。对于下面这样一段原始的代码：
```JAVA
String isocode = user.getAddress().getCountry().getIsocode().toUpperCase();
```
如果其中任何一个执行结果是null，那么这段代码都将抛出NPE异常。为了安全，原始而落后的写法是：
```JAVA
if (user != null) {
    Address address = user.getAddress();
    if (address != null) {
        Country country = address.getCountry();
        if (country != null) {
            String isocode = country.getIsocode();
            if (isocode != null) {
                isocode = isocode.toUpperCase();
            }
        }
    }
}
```
这段代码写得太“脏”了，难以维护，所以我们可以使用`Optional`对其进行维护，首先要重构类，使得get方法返回`Optional`对象：
```JAVA
public class User {
    private Address address;

    public Optional<Address> getAddress() {
        return Optional.ofNullable(address);
    }

    // ...
}
public class Address {
    private Country country;

    public Optional<Country> getCountry() {
        return Optional.ofNullable(country);
    }

    // ...
}
```
这样就能删除null的检查，全部替换为`Optional`：
```JAVA
String result = Optional.ofNullable(user)
  .flatMap(User::getAddress)
  .flatMap(Address::getCountry)
  .map(Country::getIsocode)
  .orElse("default");
```


参考：
* [理解、学习与使用 JAVA 中的 OPTIONAL](https://www.cnblogs.com/zhangboyu/p/7580262.html)
