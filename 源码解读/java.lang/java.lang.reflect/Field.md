# Field

提供有关类或接口的单个域的信息或动态访问权限，这个字段可以是静态域或者实例域。

可以通过类对象中的`getFields()`、`getField(fieldName)`、`getDeclaredFields()`或`getDeclaredField(fieldName)`来取得域数组或域对象。注意这几种方法的区别。

该类结构如下：

![Field](http://ovn0i3kdg.bkt.clouddn.com/Field.png)

### public final class Field extends AccessibleObject implements Member
类声明，说明Field类无法被继承。

### Field(Class<?> declaringClass, String name, Class<?> type, int modifiers, int slot, String signature,byte[] annotations)
构造方法。包内访问，反正开发人员是不能调用了。
```java
Field(Class<?> declaringClass,
      String name,
      Class<?> type,
      int modifiers,
      int slot,
      String signature,
      byte[] annotations)
{
    this.clazz = declaringClass;
    this.name = name;
    this.type = type;
    this.modifiers = modifiers;
    this.slot = slot;
    this.signature = signature;
    this.annotations = annotations;
}

private Class<?>            clazz;
private int                 slot;
// This is guaranteed to be interned by the VM in the 1.4
// reflection implementation
private String              name;
private Class<?>            type;
private int                 modifiers;
// Generics and annotations support
private transient String    signature;
// generic info repository; lazily initialized
private transient FieldRepository genericInfo;
private byte[]              annotations;
```
重点是这些参数：
1. clazz：代表这个域所属的类
2. name：代表这个域的名称
3. type：这个和clazz有什么区别呢？
4. modifiers：int类型，表示这个域的修饰符？为什么不是一个数组呢？
5. slot： int类型，暂时还不知道这是什么意思
6. signature：方法签名的意思么？
7. annotations：注解，这个很好理解，一个域可能被很多注解注释，这里是byte类型的数组，为什么是byte类型？


## 常用方法
类中的方法很多，我们不需要每个都了解，只要知道一些常用的方法就可以了，毕竟我也不会去写框架什么的。常用方法分为两类吧，一是获取域属性，二是修改属性。

## 获取属性
获取域的属性，哪些属性？比如属性的名称、修饰符、所属类等等。

| 方法 | 说明 |
| :------------- | :------------- |
| Class<?> getDeclaringClass | 获取所属类的全称    |
| Class<?> getType()  |获取属性声明类型，返回Class独享   |
|Type getGenericType()|获取属性类型，返回的是Type类型 |
|String getName   |  获取属性声明时的名字  |
|boolean isEnumConstant   | 判断该属性是否为枚举型  |
|int getModifiers   |  获取属性对象的修饰符 |
|boolean isSynthetic   |  判断这个域是否是人工合成的域？？？ 什么是人工合成域 |
| Class<?> getType()  |获取属性类型   |
|getAnnotations()    |   获得这个属性上所有的注释 |

。。。。
以上是一些主要的方法。

> 注意`getType`方法和`getGenericType`方法的区别。首先返回值类型不同，前者返回`Class`对象，后者返回`Type`接口。再者，如果一个属性是泛型，那么前者只能得到这个属性的接口类型，而后者还等得到这个属性的泛型类型。

## 获取属性并修改属性

`Field`类中有两个特殊的方法
1. public Object get(Object obj)：用来获取obj这个对象的这个域的值
2. public void set(Object obj, Object value)：用来修改obj这个对象的域的值。

下面是一个测试代码。
首先定义一个类：
```java
public class Person {

    private int age;

    public String name;

    public String sex;

    //setter and getter
    public static void main(String[] args) {
        Person person = new Person();
        person.setAge(10);
        person.setName("Alice");
        person.setSex("female");

        Class cls = person.getClass();

//        Field[] fields = cls.getFields();// 只能取得public的域
        Field[] fields = cls.getDeclaredFields(); // 可以取得本类的所有域，不限访问修饰符

        for (Field field : fields){
            System.out.println(field.getName());
        }

        try {
            Field field = cls.getField("age");//age是私有域，将抛出异常
            System.out.println(field.get(person));
        } catch (NoSuchFieldException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }

        try {
            Field field = cls.getDeclaredField("age");//age是私有域，将抛出异常
            field.setAccessible(true);//当获取私有属性值的时候，需要设置Accessible为true
            System.out.println(field.get(person)); // 10
            field.set(person, 20);
            System.out.println(field.get(person)); // 20
        } catch (NoSuchFieldException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
    }
}


```
