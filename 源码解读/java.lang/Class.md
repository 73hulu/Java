# Class

Class真的很重要啊，年少无知的我听到"反射"一词就很烦躁，但是反射基本是所有厉害的Java框架的基础，结构如：

![Class](http://ovn0i3kdg.bkt.clouddn.com/Class_structure_1.png)
![Class](http://ovn0i3kdg.bkt.clouddn.com/Class_structure_2.png)
![Class](http://ovn0i3kdg.bkt.clouddn.com/Class_structure_3.png)
![Class](http://ovn0i3kdg.bkt.clouddn.com/Class_structure_4.png)


在阅读源码之前，我们了解下基本概念，`Class`到底是什么？`Class`是一个，跟Java API中定义的诸如`Thread`、`Integer`没有什么两样。类是对一类事物的抽象，那么`Class`抽象了什么呢？它的实例又代表了什么呢？

在一个运行的程序中，会有许多的类和接口存在。我们用`Class`来表示对这些类和接口的抽象，而`Class`类的每个实例则代表了运行中的一个类。例如，运行的程序有ABC三个类，那么Class类就是对ABC三个类的抽象。所谓抽象，就是提取这些类的一个公共特征，比如这些类都有类名，都有对应的hashcode方法，可以判断类型是属于class、interface、enum还是annotation，这些可以封装成`Class`类的域，另外可以定义一些方法，比如获取某个方法、获取类型名字等等，这样就封装了一个表示类型(type)的类。

Java中，每个类class都有一个相应的Class，也就是说，当我们编写一个类，编译完成之后，在生成的`.class`文件中，就会产生一个Class对象，用于表示这个类的信息。运行程序时，JVM首先检查是否要加载的类对应的Class对象是否已经加载，如果没有加载，JVM就会根据类名查找.class文件，并将其Class对象载入。虚拟机只会产生一份字节码，用这份字节码可以产生多个实例对象。

特别的是，8种基本数据类型和void关键字也都对应了一个Class对象；每个数组属于被映射为Class对象的一个类，所有具有相同元素类型和维数的数组都共享该Class对象。

一般某个类的Class对象被载入内存，它就用来创建这个类的所有对象。


### public final class Class<T> implements java.io.Serializable, GenericDeclaration,Type, AnnotatedElement

看到类声明，Class是一个终类，且是一个泛型类。实现了`Serializable`、`GenericDeclaration`、`Type`和`AnnotatedElement`接口。
后三个接口有点陌生啊，他们都在`java.lang.reflect`子包中，这个包与反射机制密切相关。`GenericDeclaration`可以声明类型变量的实体的公共接口，也就是说，只有实现了该接口才能在对应的实体上声明（定义）类型变量，这些实体目前只有三个：`Class`（类）、`Constructor`(构造器)、`Method`(方法)
这个接口只有一个方法：
```java
public TypeVariable<?>[] getTypeParameters();
```
用来返回实体声明（定义）的所有类型变量，测试程序参考 http://blog.csdn.net/a327369238/article/details/52710827

`Type`接口是Java变成语言中所有类型的公共高级接口。也只有一个方法：
```java
default String getTypeName() {
    return toString();
}
```
> 这里的default是可访问性修饰符么，相当于为空？？？

`AnnotatedElement`接口表示目前正在此 VM 中运行的程序的一个已注释元素。定义的方法如下：
![AnnotatedElement](http://ovn0i3kdg.bkt.clouddn.com/AnnotatedElement.png)

 `java.lang.reflect`这个子包很重要，详情见专门的博客，这里就不喧宾夺主了。
 ### private static native void registerNatives();
 这句话就很熟悉了，在`Object`中看到过，当时是通过静态初始块来执行这个私有方法的，同样的，在`Class`类中也是这样，后面跟着一个静态初始块。
 ```java
 private static native void registerNatives();
 static {
     registerNatives();
 }
 ```

###  private Class(ClassLoader loader){...}
诶这就怪了，网上看到的大部分说`Class`没有构造方法，不能手动创建，为什么源码里明明就有构造方法啊？

网上的说法不准确，`Class`不能手动创建对象的原因并不是没有构造方法，而是因为构造方法是private，不能调用。准确的说是：“Class类没有公共构造方法”。Class对象是在加载类时由 Java 虚拟机以及通过调用类加载器中的defineClass方法自动构造的。这个私有的构造方法怎么定义的呢？
```java
/*
 * Private constructor. Only the Java Virtual Machine creates Class objects.
 * This constructor is not used and prevents the default constructor being
 * generated.
 */
private Class(ClassLoader loader) {
    // Initialize final field for classLoader.  The initialization value of non-null
    // prevents future JIT optimizations from assuming this final field is null.
    classLoader = loader;
}
```
接收一个`ClassLoader`类型的参数，赋值给自己的属性`classLoader`，它是这么声明的：
```java
// Initialized in JVM not by private constructor
// This field is filtered from reflection access, i.e. getDeclaredField
// will throw NoSuchFieldException
private final ClassLoader classLoader;
```
那么这个参数又是怎么回事呢?等下回我们分析`ClassLoader`这个类的时候再说吧。

说道这里，我们知道`ClassLoader`对象由自己创建，但是反射机制就是通过类对象来操作类啊，我们还必须得得到这个类对象才行!既然不能自己创建，那么能不能获取呢。可以的，有三种来获取Class对象。
1.  通过某个对象的`getClass()`方法
还记得`Object`类的源码么？里面的`getClass`方法就是用来获取类对象的。比如`Class userClass = new User().getClass();`
2. 通过Class类的静态方法`forName`方法来获取
这个方法的定义如下：
```java
@CallerSensitive
public static Class<?> forName(String className)
            throws ClassNotFoundException {
    Class<?> caller = Reflection.getCallerClass();
    return forName0(className, true, ClassLoader.getClassLoader(caller), caller);
}
```
这里有个JVM注解`@CallerSensitive`，作用可以参考http://blog.csdn.net/hel_wor/article/details/50199797 还有 http://blog.csdn.net/aguda_king/article/details/72355807， 这个方法中调用了`Reflection`的静态方法，具体怎么实现我也不追究了（主要是好复杂啊 (⊙﹏⊙)b）。反正可以获取到就行了，注意这里可能会抛出异常。
`forName`还有两个重载的方法，不常用就不说了。
2. 通过类名。如果T是一个Java类型，那么T.class就代表了匹配的类对象。
例如：

```java
public class ClassTest {
  public static void main(String[] args) {
       Class userClass = User.class;
       System.out.println(userClass.getName());//JavaLangJarTest.User

       Class cls1 = int.class;
       System.out.println(cls1.getName()); //int

       Class cls2 = double.class;
       System.out.println(cls2.getName()); //double

       Class cls3 = Double[].class;
       System.out.println(cls3.getName()); //Ljava.lang.Double;

       Double[] doubleArr = new Double[10];
       Class cls4 = doubleArr.getClass();
       System.out.println(cls4.getName());//Ljava.lang.Double;

       System.out.println(cls4 == cls3); // true

       Double[] doubleArr2 = new Double[11];
       Class cls5 = doubleArr2.getClass();
       System.out.println(cls4 == cls5); //true
   }
}
```
上面的测试程序可以验证这几点：
①基本数据类型也对应了各自的类对象，也就是说Class对象实际上描述的只是类型，而这类型未必是类或接口。

②相同类型的数组对象的类对象是同一个；

③由于历史原因，数组类型的getName方法会返回奇怪的名字，比如上面的`Ljava.lang.Double`;

### public String getName(){...}
这个方法在前面用到了，方法将以字符串的行驶返回此Class对象所表示的实体(类、接口、数组类、基本类型或void)**完整**名称。方法定义如下:
```java
public String getName() {
   String name = this.name;
   if (name == null)
       this.name = name = getName0();
   return name;
}
```
这里`this.name`可以看到`Class`本身有一个属性，定义如下：
```java
private transient String name;
```
这个属性被`transient`关键词修饰，说明不参与序列化。而当this.name是null的时候，调用了`getName0`方法，这个方法的定义如下：
```java
private native String getName0();
```
又是一个native方法，好吧不管这么多了，反正最后一定可以得到name。

> 记住Enum是类，而Annotation是接口。

### public T newInstance() throws InstantiationException, IllegalAccessException{...}
这个方法就常用了，使用类对象来创建对象，←_←嗯 好像是没有说错。定义如下：
```java
@CallerSensitive
   public T newInstance()
       throws InstantiationException, IllegalAccessException
   {
       if (System.getSecurityManager() != null) {
           checkMemberAccess(Member.PUBLIC, Reflection.getCallerClass(), false);
       }

       // NOTE: the following code may not be strictly correct under
       // the current Java memory model.

       // Constructor lookup
       if (cachedConstructor == null) {
           if (this == Class.class) {
               throw new IllegalAccessException(
                   "Can not call newInstance() on the Class for java.lang.Class"
               );
           }
           try {
               Class<?>[] empty = {};
               final Constructor<T> c = getConstructor0(empty, Member.DECLARED);
               // Disable accessibility checks on the constructor
               // since we have to do the security check here anyway
               // (the stack depth is wrong for the Constructor's
               // security check to work)
               java.security.AccessController.doPrivileged(
                   new java.security.PrivilegedAction<Void>() {
                       public Void run() {
                               c.setAccessible(true);
                               return null;
                           }
                       });
               cachedConstructor = c;
           } catch (NoSuchMethodException e) {
               throw (InstantiationException)
                   new InstantiationException(getName()).initCause(e);
           }
       }
       Constructor<T> tmpConstructor = cachedConstructor;
       // Security check (same as in java.lang.reflect.Constructor)
       int modifiers = tmpConstructor.getModifiers();
       if (!Reflection.quickCheckMemberAccess(this, modifiers)) {
           Class<?> caller = Reflection.getCallerClass();
           if (newInstanceCallerCache != caller) {
               Reflection.ensureMemberAccess(caller, this, null, modifiers);
               newInstanceCallerCache = caller;
           }
       }
       // Run constructor
       try {
           return tmpConstructor.newInstance((Object[])null);
       } catch (InvocationTargetException e) {
           Unsafe.getUnsafe().throwException(e.getTargetException());
           // Not reached
           return null;
       }
   }
   private volatile transient Constructor<T> cachedConstructor;
   private volatile transient Class<?>       newInstanceCallerCache;

```
哎好长好烦躁，总之这个方法是调用了**默认构造器（无参数）构造器**初始化新建对象。这里需要注意两点：① 如果类本身没有定义无参构造方法但是定义的有参构造方法，这时候是无法默认产生无参构造方法的，所以这时候用`newInstance`方式来创建对象的话，会抛出`InstantiationException`异常。②`newInstance`方法返回的是Object对象，需要进行类型转换。


###  public String toString() {...}
```Java
public String toString() {
     return (isInterface() ? "interface " : (isPrimitive() ? "" : "class "))
         + getName();
 }
```
>   `isPrimitive`方法是判断是不是基本类型。
在类名之前加上类型，是类的就加上"class"，是接口的就加上"interface", 是基本类型的就什么都不加。前面说到Enum是类，Annotation是接口，我们来验证一下：

```java
Class cls1 = Enum.class;
System.out.println(cls1.toString()); //class java.lang.Enum

Class cls2 = Annotation.class;
System.out.println(cls2.toString()); //interface java.lang.annotation.Annotation
```
结果很明显啦~

### public ClassLoader getClassLoader(){...}
用来获取该类的类加载器，至于获得了之后干什么，母鸡啊~
```Java
@CallerSensitive
public ClassLoader getClassLoader() {
    ClassLoader cl = getClassLoader0();
    if (cl == null)
        return null;
    SecurityManager sm = System.getSecurityManager();
    if (sm != null) {
        ClassLoader.checkClassLoaderPermission(cl, Reflection.getCallerClass());
    }
    return cl;
}

// Package-private to allow ClassLoader access
ClassLoader getClassLoader0() { return classLoader; }

// Initialized in JVM not by private constructor
// This field is filtered from reflection access, i.e. getDeclaredField
// will throw NoSuchFieldException
private final ClassLoader classLoader;
```

### 其他一些常用的方法
还有一些常用的方法，源码实现就略过了，只是讲下作用和使用示例。

| 方法 | 作用   |  使用示例|
| :------------- | :------------- | :--- |
| public Field[] getFields() throws SecurityException{...}   | 反射中获得public域成员，包括父类  |  Field[] fields = String.class.getFields();for (Field field : fields){  System.out.print(field.toString()); //public static final java.util.Comparator java.lang.String.CASE_INSENSITIVE_ORDER }|
|public Field getField(String name) throws NoSuchFieldException, SecurityException{..}    |  反射中获得特定名称public域成员，可能会抛出异常|   |
|public Field[] getDeclaredFields() throws SecurityException{...}   | 反射中获得所有域成员，不限访问控制符，但只能是自己的，不包括父类，  |   |
|public Field getDeclaredField(String name) throws NoSuchFieldException, SecurityException{...}   | 反射中获得特定名称的域成员，可能会抛出异常  |   |
|public Method[] getMethods(){...}    | 获得方法  |   |
|getConstructors() {...}   | 获得所有的构造方法  |   |
|public Method getDeclaredMethod(String name, Class<?>... parameterTypes){...}   | 加个Declared代表本类，继承，父类均不包括。  |   |
| public native Class<? super T> getSuperclass();    |获取类的父类，继承了父类则返回父类，否则返回java.lang.Object。返回Object的父类为空-null。 |  Class superClass = String.class.getSuperclass(); System.out.println(superClass.getName()); // java.lang.Object|
|  public Package getPackage() {...}| 反射中获得package |   Package classPackage = String.class.getPackage(); System.out.println(classPackage.toString()); //package java.lang, Java Platform API Specification, version 1.8|
|public Class<?>[] getInterfaces(){...}   |  获得类实现的接口 |  Class[] classImplements = String.class.getInterfaces(); for (Class cls : classImplements){System.out.print(cls.getName() + " "); //java.io.Serializable java.lang.Comparable java.lang.CharSequence }|
|public native int getModifiers();   |反射中获得修饰符，如public， static ,void等， 注意这里的返回类型是int |   int modifier = String.class.getModifiers(); System.out.println(modifier); // 17|
| public boolean isEnum()   |  判断是否是枚举类型 |   |
|public native boolean isArray()    |  判断是否是数组类型 |   |
|public native boolean isPrimitive()   |  判断是否是基本类型 |   |
|public boolean isAnnotation()    |  判断是否是注解类型 |  xxx |

网上找到一些具体是操作例子可以参考下：
* http://blog.csdn.net/sd4000784/article/details/7448221
* http://blog.csdn.net/liushuijinger/article/details/15378475

参考：
1. [深入研究java.lang.Class类](http://lavasoft.blog.51cto.com/62575/15433/)
2. [java Class类](http://www.cnblogs.com/whgw/archive/2011/10/04/2198837.html)
3. [java Class 类简介](http://blog.csdn.net/a379039233/article/details/6158816/)
4. [理解java.lang.Class类](http://blog.csdn.net/bingduanlbd/article/details/8424243/)
5. [我眼中的Java-Type体系（1）](http://www.jianshu.com/p/7649f86614d3)
6. [菜鸟学Java（十五）——Java反射机制（二)](http://blog.csdn.net/liushuijinger/article/details/15378475)
7. [菜鸟学Java（十四）——Java反射机制（一)](http://blog.csdn.net/liushuijinger/article/details/14223459)
8. [Java基础加强--17.利用反射操作泛型VI](http://blog.csdn.net/benjaminzhang666/article/details/9880561)
