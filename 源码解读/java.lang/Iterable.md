# Iterable

这是一个接口（不同于java.util中的`Iterator`），定义了迭代器所具有的功能。实现此接口允许对象成为“for-each loop”语句的目标。

什么是"for-each loop"？官方doc对[for each loop](https://docs.oracle.com/javase/8/docs/technotes/guides/language/foreach.html)专门做了解释：

集合迭代的代码实现往往非常臃肿。考虑下面这个例子，它遍历定时器任务并取消它们：
```java
void cancelAll(Collection<TimerTask> c) {
    for (Iterator<TimerTask> i = c.iterator(); i.hasNext(); )
        i.next().cancel();
}
```
这个迭代器的使用非常繁琐也容易导致错误。因为迭代器在循环中出现了3次，后面两次都可能出错。而for-each循环解决了这些问题：
```java
void cancelAll(Collection<TimerTask> c) {
    for (TimerTask t : c)
        t.cancel();
}
```
其中冒号读作“in”。这个循环读作“for each TimerTask t in c.”for-each循环和泛型完美结合，此时我们不需要声明迭代器，因此也不需要为它提供泛型定义（实际上编译器为我们实现了这些）。所以它不但提供了类型安全，也去掉了复杂的语法 。

下面是嵌套循环中容易出现的另一个错误：
```java
List suits = ...;
List ranks = ...;
List sortedDeck = new ArrayList();

// BROKEN - throws NoSuchElementException!
for (Iterator i = suits.iterator(); i.hasNext(); )
    for (Iterator j = ranks.iterator(); j.hasNext(); )
        sortedDeck.add(new Card(i.next(), j.next()));

```

如果你没有发现这个bug请不要灰心，许多专家级的程序员也会时不时犯这个错误。仔细检查会发现suits的next方法被调用了太多次。要解决这个问题可以采用下列方法：
```java
// Fixed, though a bit ugly
for (Iterator i = suits.iterator(); i.hasNext(); ) {
    Suit suit = (Suit) i.next();
    for (Iterator j = ranks.iterator(); j.hasNext(); )
        sortedDeck.add(new Card(suit, j.next()));
}
```
那这和for-each循环有什么关系呢？实时上它可以完美解决这个问题：
```java
for (Suit suit : suits)
    for (Rank rank : ranks)
        sortedDeck.add(new Card(suit, rank));
```
for-each对数组同样有效：
```java
// Returns the sum of the elements of a
int sum(int[] a) {
    int result = 0;
    for (int i : a)
        result += i;
    return result;
}
```
那么我们什么时候采用for-each循环呢？实际上我们应该尽可能使用它。它可以让代码更加完美。但是它也有自己的缺点。因为`for-each`隐藏了迭代器，因此我们无法调用`remove`方法以删除元素，也无法在循环中替换元素。同时它也不适合并行遍历多个集合。
事实上`for-each`的设计者们精心考虑过这些问题，目前的`for-each`设计是一个简洁易用而且足以覆盖大多数情况的解决方案。

`Iterable`接口结构如下：

![Iterable](http://ovn0i3kdg.bkt.clouddn.com/Iterable.png)

其中后两个方法是1.8新加的非抽象方法，目前我们只需要知道第一个抽象方法就行了。

### Iterator<T> iterator();
实现`Iterable`接口的类需要实现该抽象方法，该方法获取到了在这个对象上的迭代器。
