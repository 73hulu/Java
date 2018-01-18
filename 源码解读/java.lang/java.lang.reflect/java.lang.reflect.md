
这个子包包含了与反射相关的所有接口和类。
具体如下：

接口：

| 接口 | 说明 |
| :------------- | :------------- |
| AnnotatedElement       | 表示目前正在此 VM 中运行的程序的一个已注释元素。    |
|GenericArrayType   |   GenericArrayType 表示一种数组类型，其组件类型为参数化类型或类型变量。|
|GenericDeclaration   | 声明类型变量的所有实体的公共接口。  |
|InvocationHandler   |   InvocationHandler 是代理实例的调用处理程序 实现的接口。|
|Member   |   成员是一种接口，反映有关单个成员（字段或方法）或构造方法的标识信息。|
|ParameterizedType   |  ParameterizedType 表示参数化类型，如 Collection<String>。 |
|Type   |  Type 是 Java 编程语言中所有类型的公共高级接口。 |
|TypeVariable<D extends GenericDeclaration>   | TypeVariable 是各种类型变量的公共高级接口。  |
|WildcardType   |   WildcardType 表示一个通配符类型表达式，如 ?、? extends Number 或 ? super Integer。|


类：

| 类 | 说明  |
| :------------- | :------------- |
| AccessibleObject      | AccessibleObject 类是 Field、Method 和 Constructor 对象的基类。    |
|Array   |  Array 类提供了动态创建和访问 Java 数组的方法。 |
|Constructor<T>   | Constructor 提供关于类的单个构造方法的信息以及对它的访问权限。  |
|Field   | Field 提供有关类或接口的单个字段的信息，以及对它的动态访问权限。  |
|Method   | Method 提供关于类或接口上单独某个方法（以及如何访问该方法）的信息。  |
|Modifier   | Modifier 类提供了 static 方法和常量，对类和成员访问修饰符进行解码。  |
|Proxy   |  Proxy 提供用于创建动态代理类和实例的静态方法，它还是由这些方法创建的所有动态代理类的超类。 |
|ReflectPermission   | 反射操作的 Permission 类。  |
参考
* [ 菜鸟学Java（十四）——Java反射机制（一）](http://blog.csdn.net/liushuijinger/article/details/14223459)

阅读材料

* 《Java深度历险》（看看第1、2、5三章）

* 《 java reflection in action 》（有一定基础再看）
