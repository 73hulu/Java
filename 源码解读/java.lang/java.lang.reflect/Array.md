# Array


`Array`提供了数组的动态创建爱你和访问Java数组的办法，比较特殊。为什么特殊呢？因为我们平常创建数组都不经过这个类，而是直接创建，比如我们创建一个`String`类型的数组，通常会这样做：
```Java
String [] array = new String[10];
```
全程都没有没有`Array`这个类什么事情，但是我们可以通过这个类动态创建数组，什么是动态当然是反射了。先看这个类的结构：

![Array](http://ovn0i3kdg.bkt.clouddn.com/Array.png)

很显然，看到了几个熟悉的方法，比如`newInstance`、`get`和`set`方法，这三个是主要的操纵数组的方法。

我们不需要知道底层如何实现，但是需要记住方法签名。
* public static Object newInstance(Class<?> componentType, int length)
第一个参数是数组元素的类对象，第二个元素是长度，所以由这个方法创建的数组是一维数组。需要注意，**方法的而返回类型是Object。**

* public static Object newInstance(Class<?> componentType, int... dimensions)
第一个参数是数组元素的类对象，第二个数组是一个变长参数，指的是各个维度，所以由这个方法创建的数组是一个多维数组。

* public static native Object get(Object array, int index) throws IllegalArgumentException, ArrayIndexOutOfBoundsException;
注意到这个一个本地方法，两个参数，第一个是操纵的数组的对象，第二个是位置的索引

* public static native void set(Object array, int index, Object value)throws IllegalArgumentException, ArrayIndexOutOfBoundsException;
同样是本地方法，三个参数，第一个是操纵的数组的对象，第二个是位置的索引，第三个是用来替换的对象

`Array`类有一个特点，那就是所有方法都是静态方法。

> ！！！！ 注意,`Array`类没有public的构造方法，所以根本不能实例化，所以`Array array = new Array()`这种写法是错误的。

下面是一个利用`Array`类来创建`String`类型数组，并向其中添加内容的例子:
```Java
public class ArrayTest {

    public static void main(String[] args) {
        Object array = Array.newInstance(String.class, 10); //创建一个大小为10的String类型的数组，注意返回值是Object

        //往数组中添加一项内容
        Array.set(array, 1, "Hello");
        Array.set(array, 2, "baobao");
        Array.set(array, 3, "how are you");

        //获取数组中的一项
        System.out.println(Array.get(array, 3)); // how are you
    }
}
```


学到这里我们可能会想另外一个和数组有关的类`Arrays`，这个类也提供了对数组的操作，比如排序啊，查找啊这种，有什么区别。

`Array`和`Arrays`的关系就像是`Object`和`Objects`、`Collection`和`Collections`的关系，那就是后者都是一个工具类，前者才是正主！。`Array`所在包是`java.lang.reflect`，而`Arrays`所在包是`java.util`，前者可以通过反射来创建数组并操作数组，后者只能用来操纵数组。
