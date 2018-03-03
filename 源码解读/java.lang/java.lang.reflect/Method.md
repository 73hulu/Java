# Method

`Method`类提供了关于类或接口上单独方法（以及如何访问该方法）的信息。

使用类对象的`getMethods()`、`getMethods(String, Class[])`、`getDeclaredMethods()`和`getDeclaredMethod(String, Class[])`可以获取`Method`对象。同样注意有`declared`和没有`declared`方法的区别。

![Method](http://ovn0i3kdg.bkt.clouddn.com/Method.png)

## public final class Method extends Executable
类声明。不可继承，实现`Executable`接口。

## 常用方法
同样有些方法只需要会用就行了，这个方法大都用力获取方法的各类属性，比如返回值类型、参数个数、参数类型等，大致如下：

| 方法 | 说明 |
| :------------- | :------------- |
| Class<?> getDeclaringClass()       | 获取所属类的类对象      |
|String getName()   |  获取方法名称 |
|TypeVariable<Method>[] getTypeParameters()   |  获取参数列表 |
|Type[] getGenericParameterTypes()| 获取参数列表|
|Class<?>[] getParameterTypes()|获取参数类型|
|int getParameterCount()|获取参数个数 |
|int getModifiers()   |获取修饰符   |
|Class<?> getReturnType()   |  获取返回值类型 |
|Type getGenericReturnType()  | 获取返回值类型  |
|Class<?>[] getExceptionTypes() |  获取异常类型 |
|  Type[] getGenericExceptionTypes() |  获取异常类型  |

。。。
常用的大致是以上这些了。可以发现`getXXXType`和`getGenericXXXTypes`这种形式经常成对出现，他们的区别和`Field`中讲解的一样。

## invoke方法
这是最重要的方法，反射取得Method的目的不都是执行么，这个方法就是触发方法执行。定义如下：
```java
public Object invoke(Object obj, Object... args)
    throws IllegalAccessException, IllegalArgumentException,
       InvocationTargetException
{
    if (!override) {
        if (!Reflection.quickCheckMemberAccess(clazz, modifiers)) {
            Class<?> caller = Reflection.getCallerClass();
            checkAccess(caller, clazz, obj, modifiers);
        }
    }
    MethodAccessor ma = methodAccessor;             // read volatile
    if (ma == null) {
        ma = acquireMethodAccessor();
    }
    return ma.invoke(obj, args);
}
```
方法有两个参数，第一个参数指定方法触发的对象，第二个参数是一个变长参数列表，指定了方法的参数。另外需要注意该方法的返回值是`Object`类型，我们需要将执行结果进行类型转换。

> 当然，反射中触发方法并不是都调用invoke方法， 可以反射取得一个实例，然后一般的实例方法调用就行。

下面是`invoke`方法使用的例子:
```java
public class MethodTest {

    public String saySomething(String name, String msg){
        String res = "Hello " + name  + ", " + msg;
        return res;

    }

    public static void main(String[] args) {
        //通过反射触发saySomething方法

        Class cls = MethodTest.class;

        //触发方式1： 得到method实例，调用invoke方法
        try {
            Method method = cls.getDeclaredMethod("saySomething", String.class, String.class);

           System.out.println((String)method.invoke(new MethodTest(), "baobao", "how are you"));//Hello baobao, how are you

        } catch (NoSuchMethodException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        }

        //触发方式2：反射得到一个实例，用实例触发

        try {
            MethodTest methodTest = (MethodTest) cls.newInstance(); //注意要进行类型转换
            System.out.println(methodTest.saySomething("Alice", "how are you"));//Hello Alice, how are you


        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }

    }
}
```

> method.invoke方法也是动态代理类中用到的一个基础知识，明白了这个再看`InvokationHandler`接口实现类的`invoke`方法会相对容易些。
