# Iterator

迭代器。注意这是一个**接口**而不是一个类。作为一种**设计模式**，迭代器可以用于遍历一个对象，对于这个对象的底层结构开发人员不必去了解。java中的`Iterator`一般称为“轻量级”对象，创建它的代价是比较小的。


`Iterator`是一个集合的迭代器。 迭代器代替Java集合框架中的`Enumeration`。 以下两点是`Iterator`不同于`Enumeration`的地方：
1. 迭代器允许调用者在迭代期间从底层集合中删除元素，并具有明确定义的语义。
2. 方法名称得到改进。

![Iterator](http://ovn0i3kdg.bkt.clouddn.com/Iterator.png)

##  boolean hasNext();
如果迭代器有更多的元素，则返回true。

## E next();
用来返回迭代中的下一个元素，迭代出的元素是集合中元素的拷贝。特别需要注意的是，在返回下一个元素的时候，同时，**迭代游标cursor后移**。
所以看下面这段代码：

```java
List suits = ...;
List ranks = ...;
List sortedDeck = new ArrayList();
// BROKEN - throws NoSuchElementException!
for (Iterator i = suits.iterator(); i.hasNext(); )
    for (Iterator j = ranks.iterator(); j.hasNext(); )
        sortedDeck.add(new Card(i.next(), j.next()));
```
这是一个常常会犯错的错误，执行后将会抛出`NoSuchElementException`异常，原因在于在内层循环中`suits`的`next`方法执行了太多次，而这个方法每执行一次都会将迭代游标后移，所以最后会过早将元素读完，抛出异常。正确的写法如下：
```java
for (Iterator i = suits.iterator(); i.hasNext(); ) {
  Suit suit = i.next();
  for (Iterator j = ranks.iterator(); j.hasNext() ; ) {
    sortedDeck.add(new Card(suit, j.next()));
  }
}
```
上面这种写法是对的，但是代码很臃肿，更优雅的方式是使用for-each loop:
```java
for (Suit suit : suits  ) {
  for (Rank rank : ranks) {
    sortedDeck.add(new Card(suit, rank));
  }
}
```

##  default void remove(){...}
用来删除最近一次已经迭代出去的那个元素。定义如下：
```java
default void remove() {
    throw new UnsupportedOperationException("remove");
}
```
这个方法使用是有限制的，否则会抛出`UnsupportedOperationException`异常。什么限制呢？

**只有当`next`执行完之后，才能调用`remove`函数。** 比如要删除第一个元素，不能直接调用`remove`方法，而是要先`next`一下。


上面三个方法一般配合使用（尤其是`hasNext`和`next`方法，而`remove`方法一般很少使用，以至于Eclipse自动补齐时常把它忽略），一个例子如下：
```java
public static void main(String[] args) {
    List<Integer> list = new ArrayList<>();

    list.add(1);
    list.add(2);
    list.add(3);

    //不适用for-each循环而手动迭代

    Iterator<Integer> iter = list.iterator(); // 获取ArrayList的迭代器

    while (iter.hasNext()){ //探测是否能够继续探测
        System.out.println(iter.next());//可以继续探测的时，取出本次迭代的元素

        iter.remove();
    }


    for (Integer i : list){
        System.out.println(i);
    }
}
```
最后一个foreach循环认仍能够打印出list中的内容，这说明了，迭代器中的内容是集合内容的拷贝。注意，是内容的拷贝。那么对迭代器的修改会影响原来的集合的内容么？

有可能会！如果集合内容是基本类型，那么拷贝就是复制一份值。如果集合保存的是对象，其实质是保存的一份对象的引用，那么迭代器拷贝的也是一份引用。这个时候也要分为两种情况，如果对象是可变内容，此时通过迭代器的操作会影响集合的内容。如果对象是不可变的，如String，基本数据包装类型Integer等，那对迭代器的操作是不会反映到原集合的。下面是一个验证程序：
```java
public class IteratorTest {

    public static void main(String[] args) {
       List<Person> list = new ArrayList<>();

       list.add(new Person("Tom"));

       for (Person pr : list){
           pr.setName("Jerry");
       }
       System.out.println(list.get(0).getName()); //Jerry
    }
}
class Person {
    private String name;

    public Person(String name) {
        this.name = (name == null? "" : name);
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        if (name == null){
            name = "";
        }
        this.name = name;
    }
}
```
程序最后输出是"Jerry"而不是"Tom"，可见当集合内容为可变对象的时候，对迭代器的修改确实可以改变集合原内容。



## 实现自己的迭代器类
```java
public interface IteratorTest {

    public static void main(String[] args) {
        MyString s = new MyString("1234567");
        for (char c : s){
            System.out.print(c + " ");
        }
    }
}
class MyString implements Iterable<Character>{
    private int length = 0;
    private String inners = null;

    public MyString(String str) {
        this.inners = str;
        this.length = str.length();
    }

    @Override
    public Iterator<Character> iterator() {
        class iter implements Iterator<Character> {
            private int cur = 0;

            @Override
            public boolean hasNext() {

                return cur != length;
            }

            @Override
            public Character next() {
                Character ch = inners.charAt(cur);
                cur ++;
                return ch;
            }
        }
        return new iter();
    }
}
```
程序输出：
```java
1 2 3 4 5 6 7
```


参考：
* [Java迭代 : Iterator和Iterable接口](http://www.cnblogs.com/keyi/p/5821285.html)
