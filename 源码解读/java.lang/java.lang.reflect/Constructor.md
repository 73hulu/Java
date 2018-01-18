# Constructor

构造方法是特殊的方法，所以专门有一个类用来描述构造方法。用类对象的`getConstructors()`，`getConstructor(Class[])`， `getDeclareConstructors()`方法可以取得`Constructor`对象。

该类的结构如下：

![Constructor](http://ovn0i3kdg.bkt.clouddn.com/Constructor.png)


可以看到其中大部分的方法都很眼熟，在`Field`和`Method`中看到很多了。

其中有一个方法需要特别注意——`newInstance`方法，这表示使用这个构造方法来创建类的实例。下面是一个简单的测试例子：
```java
public class Person {

    private int age;

    public String name;

    public String sex;

    public Person() {
    }

    public Person(int age, String name, String sex) {
        this.age = age;
        this.name = name;
        this.sex = sex;
    }

    public int getAge() {
        return age;
    }

    @Override
    public String toString() {
        return "Person{" +
                "age=" + age +
                ", name='" + name + '\'' +
                ", sex='" + sex + '\'' +
                '}';
    }

    public static void main(String[] args) {

        Class cls = Person.class;
        Constructor [] cons = cls.getConstructors();
        try {
            Constructor constructor = cls.getConstructor(int.class, String.class, String.class);
            Person person1 = (Person) constructor.newInstance(10, "Bob", "male");
            System.out.println(person1.toString()); //Person{age=10, name='Bob', sex='male'}
        } catch (NoSuchMethodException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        }
        System.out.println(cons.length); // 2
    }
}
```

讲到这里，我们可以总结下创建一个类的实例的方法了，有以下几种方法：
1. 通过new方法自主创建
2. 获取类的类对象，通过类对象的`newInstance`方法可以创建实例。
3. 获取类的类对象，获取指定的构造方法`Constructor`实例，通过这个实例的`newInstance`方法创建类的实例。
