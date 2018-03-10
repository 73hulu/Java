# AtomicReference

提供了引用变量的读写原子性操作。

![AtomicReference](http://ovn0i3kdg.bkt.clouddn.com/AtomicReference.png?imageView/2/w/400)

什么叫提供原子性的操作呢？原子性是指同一个时间这个对象只能被一个线程操作，这叫原子性，那为什么说`AtomicReference`提供了原子性的操作呢？我们可以从源码中看到，这里面的所有方法，实际上调用的都是`sun.misc.Unsafe`类中的方法，而这个类中所有的方法都是native方法，应该和OS的硬件相关，从源码中我们并不知道怎么实现的。只要记住这是原子性的操作。

## 构造函数

```Java
private volatile V value;

/**
* Creates a new AtomicReference with the given initial value.
*
* @param initialValue the initial value
*/
public AtomicReference(V initialValue) {
   value = initialValue;
}

/**
* Creates a new AtomicReference with null initial value.
*/
public AtomicReference() {
}
```

## get方法
用来返回当前的引用。
```Java
/**
 * Gets the current value.
 *
 * @return the current value
 */
public final V get() {
    return value;
}
```

##  public final boolean compareAndSet(V expect, V update) {..}
如果当前值与给定的expect相等，（注意是引用相等而不是equals()相等），更新为指定的update值。
```Java
public final boolean compareAndSet(V expect, V update) {
    return unsafe.compareAndSwapObject(this, valueOffset, expect, update);
}
private static final Unsafe unsafe = Unsafe.getUnsafe();
```
## public final V getAndSet(V newValue){..}
原子地设为给定值并返回旧值。
```Java
public final V getAndSet(V newValue) {
    return (V)unsafe.getAndSetObject(this, valueOffset, newValue);
}
```
##  public final void set(V newValue) {..}
给当前值设定新值。注意这个方法不是原子地。
```Java
public final void set(V newValue) {
    value = newValue;
}
```
## 使用
根据上面的操作，似乎有这样一种感觉，`AtomicReference`似乎起到了和“锁”类似的作用。我们看下面一个例子：
假设有一个类 Person，定义如下：
```Java
class Person {
    private String name;
    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String toString() {
        return "[name: " + this.name + ", age: " + this.age + "]";
    }
}
```
如果使用**普通对象引用**，那么在**多线程环境**下，对此对象的更新可能会导致不一样的结果，即存在同步问题。这个问题我们可以利用锁机制来解决。而另一个解决办法是利用原子对象引用`AtomicReference`来保证多线程环境下对象更新操作的一致性。

```Java
//普通引用
private static Person person;

//原子性引用
private static AtomicReference<Person> aPerson;

public static void main(String[] args){
  person = new Person("Tom", 18);
  aPerson = new AtomicReference<Person>(person);

  System.out.println("Atomic Person is " + aPerson.get().toString());

  Thread t1 = new Thread(new Task1());
  Thread t1 = new Thread(new Task1());
    Thread t2 = new Thread(new Task2());

    t1.start();
    t2.start();

    t1.join();
    t2.join();
    System.out.println("Now Atomic Person is " + aRperson.get().toString());
}
static class Task1 implements Runnable {
    public void run() {
        aRperson.getAndSet(new Person("Tom1", aRperson.get().getAge() + 1));

        System.out.println("Thread1 Atomic References "
                + aRperson.get().toString());
    }
}

static class Task2 implements Runnable {
    public void run() {
        aRperson.getAndSet(new Person("Tom2", aRperson.get().getAge() + 2));

        System.out.println("Thread2 Atomic References "
                + aRperson.get().toString());
    }
}
```
以上程序的输出结果可能是：
```
Atomic Person is [name: Tom, age: 18]
Thread1 Atomic References [name: Tom1, age: 19]
Thread2 Atomic References [name: Tom2, age: 21]
Now Atomic Person is [name: Tom2, age: 21]
```



参考
* [AtomicReference 原子引用](http://blog.csdn.net/chuchus/article/details/50801993)
* [Java 原子性引用 AtomicReference](https://www.jianshu.com/p/882d0e2c3ea6)
