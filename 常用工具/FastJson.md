# FastJson

## 概览
Java最快的JSON处理工具，工程首选。

Maven依赖：
```xml
<dependency>
	<groupId>com.alibaba</groupId>
	<artifactId>fastjson</artifactId>
	<version>1.2.12</version>
</dependency>
```

工具入口为`com.alibaba.fastjson.JSON`。

常用的方法有：

| 常用方法 | 说明 |
| :------------- | :------------- |
| public static final Object parse(String text); | 把JSON文本parse为JSONObject或者JSONArray  |
|public static final JSONObject parseObject(String text);   | 把JSON文本parse成JSONObject      |
|**public static final  T parseObject(String text, Class clazz);  ** | **把JSON文本parse为JavaBean **  |
|public static final JSONArray parseArray(String text);   |  把JSON文本parse成JSONArray  |
|**public static final  List parseArray(String text, Class clazz);   **|  **把JSON文本parse成JavaBean集合 ** |
|**public static final String toJSONString(Object object); **  |   **将JavaBean序列化为JSON文本 **|
|  public static final String toJSONString(Object object, boolean prettyFormat);  | 将JavaBean序列化为带格式的JSON文本   |
|public static final Object toJSON(Object javaObject);    |   将JavaBean转换为JSONObject或者JSONArray|

在实际序列化和反序列化的时候，我们需要对字段进行各种个性化操作，下面是fastjson的个性化操作。

* 序列化：
```java
String jsonString = JSON.toJSONString(obj);
```
* 反序列化
```java
VO vo = JSON.parseObject("...", VO.class);
```
* **泛型反序列化**
```java
import com.alibaba.fastjson.TypeReference;
List<VO> list = JSON.parseObject("...", new TypeReference<List<VO>>() {});
```

## 定制序列化
###  `@JSONField`定制序列化
注解源码如下：
```java
package com.alibaba.fastjson.annotation;
public @interface JSONField {
    // 配置序列化和反序列化的顺序，1.1.12版本之后才支持。默认情况下，序列化的时候是根据fieldName字母顺序进行序列化的。通过ordinal可以指定序列化顺序
    int ordinal() default 0;
     // 指定字段的名称, 配置在字段或setter/getter方法上，注意，如果字段是private，必须得有setter，否则无法反序列化
    String name() default "";
    // 指定字段的格式，对日期格式有用
    String format() default "";
    // 是否序列化
    boolean serialize() default true;
    // 是否反序列化
    boolean deserialize() default true;

    SerializerFeature[] serialzeFeatures() default {};

    Feature[] parseFeatures() default {};

    String label() default "";

    //1.2.16版本之后，制定属性的序列化类，对单独的某个属性定制序列化数据
    Class<?> serializeUsing() default Void.class;
    //1.2.16版本之后，执行属性的反序列化，
    Class<?> deserializeUsing() default Void.class;
}
```
用法如下：
1. 其中`name`、`format`、`serialize`、`deserialize`的用法比较简单，具体可以参考 [这里](https://www.w3cschool.cn/fastjson/fastjson-jsonfield.html) 的例子。
2. `serializeUsing`可以用来针对某个属性来定制序列化的格式，例如：
```java
public class User implements   {

    @JSONField(name = "ID", ordinal = 3)
    private long id;

    @JSONField(ordinal = 2)
    private String name;

    @JSONField(serializeUsing = ModelValueSerializer.class)
    private short age;

    @JSONField(format = "yyyy-MM-dd", ordinal = 1)
    private Date birthday;

    public User() {
    }

    public User(long id, String name, short age, Date birthday) {
        this.id = id;
        this.name = name;
        this.age = age;
        this.birthday = birthday;
    }

    // omit setter and getter
}
```
其中`ModelValueSerializer`类定义为：
```java
public class ModelValueSerializer implements ObjectSerializer {
    @Override
    public void write(JSONSerializer jsonSerializer, Object o, Object o1, Type type, int i) throws IOException {
        short age = (short)o;
        String text = age + "岁";
        jsonSerializer.write(text);
    }
}
```
测试程序为：
```java
public class UserTest {
    @Test
    public void jsonFeathure(){
        User user = new User(1, "Austin", (short) 18, new Date());

        String userStr = JSON.toJSONString(user);

        Assert.assertEquals("{\"age\":\"18岁\",\"birthday\":\"2018-01-25\",\"name\":\"Austin\",\"ID\":1}", userStr);
    }
}
```
3. 1.2.21版本之后，使用`alternateNames`支持反序列化时使用不同的字段名称。详见 https://www.w3cschool.cn/fastjson/fastjson-jsonfield.html
4. 1.2.21版本之后，新增`jsonDirect`属性，但是这个属性的作用我没有看懂？？？

### `@JSONType`配置
定义的位置在类上，源码如下：
```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE})
public @interface JSONType {
    boolean asm() default true;

    String[] orders() default {};

    String[] includes() default {};

    String[] ignores() default {};

    SerializerFeature[] serialzeFeatures() default {};

    Feature[] parseFeatures() default {};

    boolean alphabetic() default true;

    Class<?> mappingTo() default Void.class;

    Class<?> builder() default Void.class;

    String typeName() default "";

    Class<?>[] seeAlso() default {};

    Class<?> serializer() default Void.class;

    Class<?> deserializer() default Void.class;
}
```
`@JSONType`的配置会覆盖`@JSONField`

### `SerializeFilter`定制序列化
有时候我们会在不同的情况下制定不同的序列化策略，这时候用`@JSONType`或`@JSONField`可能都不能满足我们的需求，这时候`SerializeFilter`是最好的选择。它使用扩展编程的方式实现定制序列化，fastjson提供了多种serializer：
* PropertyPreFilter 根据PropertyName判断是否序列化
  我们一般使用`SimplePropertyPreFilter`类进行设置，例如：
  ```java
  @Test
    public void serializeFilter(){
      User2 user = new User2("Austin", (short)18);

      SimplePropertyPreFilter filter1 = new SimplePropertyPreFilter(User.class, "name");

      String res = JSON.toJSONString(user, filter1);
      Assert.assertEquals("{\"name\":\"Austin\"}", res);

      SimplePropertyPreFilter filter2 = new SimplePropertyPreFilter();
      filter1.getExcludes().add("age");

      res = JSON.toJSONString(user, filter1);
      Assert.assertEquals("{\"name\":\"Austin\"}", res);
  }
  ```
* PropertyFilter 根据PropertyName和PropertyValue来判断是否序列化
  需要自己实现序列化策略，例如：
  ```java
  @Test
  public void propertyFilterTest(){
     User2 user = new User2("Austin", (short)18);

     List<User2> list = new ArrayList<>();
     for (int i = 1; i <= 10; i ++){
         list.add(new User2("Austin", (short)i));
     }

     PropertyPreFilter filter = new PropertyPreFilter() {
         @Override
         public boolean apply(JSONSerializer jsonSerializer, Object o, String s) {
             if ("age".equalsIgnoreCase(s)){ // 会过滤掉年龄在等于5或者小于5的对象，
                 short age = ((User2)o).getAge();
                 return age <= 5;
             }
             return true;
         }
     };

     String res = JSON.toJSONString(list, filter);
     System.out.println(res); //[{"age":1,"name":"Austin"},{"age":2,"name":"Austin"},{"age":3,"name":"Austin"},{"age":4,"name":"Austin"},{"age":5,"name":"Austin"},{"name":"Austin"},{"name":"Austin"},{"name":"Austin"},{"name":"Austin"},{"name":"Austin"}]
  }
  ```
  程序执行结果跟我预料中的有点不同，我还以为它会过滤掉年龄不符合规范的整个对象，但是实际上，当年龄不符合的时，还是留下了该对象的其他字段。有什么办法可以实现我想要的效果？

  另外对于嵌套属性的过滤，可以参考http://blog.csdn.net/u010523770/article/details/51525001 这篇博文的内容。
* NameFilter 修改Key，如果需要修改Key, process返回值则可， 内置了实现类`PascalNameFilter`，这个类的作用是将属性名首字母从小写转为大小，自己可以自定义。比如下面自定义的`NameFilter`实现属性名全部大写的效果：
```java
@Test
public void nameFilterTest(){
    User2 user = new User2("Austin", (short)18);
    NameFilter filter = new NameFilter() {
        @Override
        public String process(Object source, String name, Object value) {
            if (name != null && name.length() != 0){
                return name.toUpperCase();
            }
            return name;
        }
    };
    String res = JSON.toJSONString(user, filter);
    System.out.println(res);
}
```
* ValueFilter 修改Value, 没有内置实现类，需要自己实现。如下面的程序会将`name`字段的值全部变成大写：
```java
@Test
public void valueFilterTest(){
    User2 user = new User2("Austin", (short)18);
    ValueFilter filter = new ValueFilter() {
        @Override
        public Object process(Object o, String s, Object value) {
            if (s != null && s.length() != 0){
                if ("name".equalsIgnoreCase(s) && value != null){
                    String v = (String)value;
                    return v.toUpperCase();
                }else {
                    return value;
                }
            }
            return value;
        }
    };
    String res = JSON.toJSONString(user, filter);
    System.out.println(res);
}
```
* BeforeFilter 序列化时在最前添加内容
```java
public abstract class BeforeFilter implements SerializeFilter {
     protected final void writeKeyValue(String key, Object value) { ... }
     // 需要实现的抽象方法，在实现中调用writeKeyValue添加内容
     public abstract void writeBefore(Object object);
}
```
* AfterFilter 序列化时在最后添加内容
```java
public abstract class AfterFilter implements SerializeFilter {
   protected final void writeKeyValue(String key, Object value) { ... }
   // 需要实现的抽象方法，在实现中调用writeKeyValue添加内容
   public abstract void writeAfter(Object object);
}
```

## 定制反序列化
`ParseProcess`是编程扩展定制反序列化的接口，内置了两个接口——`ExtraProcessor`和`ExtraTypeProvider`，前者用来处理多余字段，后者用于处理多余字段时提供类型信息

### `ExtraProcessor`
该接口用来处理**多余字段**，如下：
```java
@Getter
@Setter
public class VO implements Serializable{
    private int id;

    private Map<String, Object> attributes = new HashMap<String, Object>();

    public int getId() { return id; }

    public void setId(int id) { this.id = id;}

    public Map<String, Object> getAttributes() { return attributes;}
}
//测试程序
@Test
public void extraProcessorTest(){
   ExtraProcessor processor = new ExtraProcessor() {
       @Override
       public void processExtra(Object object, String key, Object value) {
           VO vo = (VO)object;
           vo.getAttributes().put(key, value);
       }
   };

   VO vo = JSON.parseObject("{\"id\":123,\"name\":\"abc\"}", VO.class, processor);
   Assert.assertEquals(123, vo.getId());
   Assert.assertEquals("abc", vo.getAttributes().get("name"));
}
```
上例中，`id`属性能在`VO`中找到对应的属性，但是`name`属性找不到，所以这里`processor`起了作用，它会将这个属性放到对象的`attributes`中。

### `ExtraTypeProvider`
该接口用来处理多余字段的类型信息。这个接口实现需要配合`ExtraProcessor`。如下：
```Java
//`ExtraProcessor`和`ExtraTypeProvider`配合使用
public class MyExtraProcessor implements ExtraProcessor, ExtraTypeProvider {
    @Override
    public void processExtra(Object object, String key, Object value) {
        VO vo = (VO) object;
        vo.getAttributes().put(key, value);
    }

    @Override
    public Type getExtraType(Object o, String key) {
        if ("value".equals(key)){
            return int.class;
        }
        return null;
    }
}

//测试程序
@Test
public void extraTypeProvider(){
   ExtraProcessor processor = new MyExtraProcessor();

   VO vo = JSON.parseObject("{\"id\":123,\"value\":\"123456\"}", VO.class, processor);
   Assert.assertEquals(123, vo.getId());
   Assert.assertEquals(123456, vo.getAttributes().get("value")); // value本应该是字符串类型的，通过getExtraType的处理变成Integer类型了。
}
```

> 在实践过程中遇到这样一个问题：有一个布尔量，并未对其设置serialize=false属性，但是序列化的时候无法序列化该属性。后来经过[JavaBean的boolean isXXX反序列化问题](https://www.cnblogs.com/xulingfeng/p/6143317.html)这篇文章的提醒，发现我的布尔量的命名是isXXX，无论是利用`@Getter`和`@Setter`注解省略getter和setter方法，还是idea生成getter和setter方法，其getter/setter为`isXXX/setXXX`，而`Boolean`的getter/setter为`getXXX/setXXX`。
> 对于boolean量，将其写成`isXXX`实际上是不规范的，此时IDE默认的getter/setter并不会被fastjson识别，反序列化过程错误。解决办法就是手动将`isXXX/setXXX`改成`isIsXXX/setIsXXX`.


参考
* [源码地址](https://github.com/alibaba/fastjson)
* [学习教程](https://www.w3cschool.cn/fastjson)
