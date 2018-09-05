# Collection

`Collection`接口是Java框架系统中的根接口。“集合”表示一组对象，这些对象被称为“元素”。一些集合允许有重复元素，一些集合不允许有重复元素。一些集合是有序的，一些集合是无序的。JDK不提供任何这种接口的直接实现：它提供了更多的特定子接口的实现，如`Set`和`List`。这种接口用于传递集合，并在需要最大的通用性的情况下对它们进行操作。

包或多重复集（可能包含重复元素的无序集合）应直接实现此接口。

所有通用的`Collection`实现类（通常通过其子接口间接实现`Collection`）应该提供两个“标准”构造函数：一个void(无参数)构造函数，它创建一个空集合，以及一个具有单个参数类型的构造函数，它创建一个与其参数相同的元素的新集合。实际上，后一个构造函数允许用户复制任何集合，从而生成所需实现类型的等价集合。没有办法强制执行此约定（因为接口不能包含构造函数），而Java平台库中的所有通用`Collection`实现都符合要求。

对于某个集合，如果此集合不支持该操作，则此接口中包含“破坏性”方法（即修改其操作的集合的方法）将被指定为抛出`UnsupportedOperationException`异常。如果是这种情况，但不是必需的。例如，如果要添加的集合为空，则在不可修改的集合上调用`addAll(Collection)`方法可能抛出异常（也可能不抛出异常）。

某些集合实现对它们可能包含的元素有限制。例如，一些实现禁止有空元素，一些实现方式对元素的类型有限制。尝试添加不合格的元素引发免检异常，通常为`NullPointerException`或`ClassCastException`。尝试查询不合格元素的存在可能会引发异常，或者可能只是返回false；一些实现将表现出前者的行为，一些实现将展示后者。更一般地说，尝试对不符合条件的元素进行操作时，其完成不会导致将不合格元素插入到集合中没可能会导致异常，或者可能会成功执行该选项。此异常在此接口的规范中标记为“可选”。

每个集合决定自己的同步策略。在没有执行的更强保证的情况下，未定义的行为可能是由于另一个线程被入编的集合上的任何方法的额调用而导致的；这包括直接调用，将集合传递给可能执行调用的方法，并使用现有的迭代器来检查集合。

集合框架的接口中许多方法都是用`equals`方法来定义的。例如：`contains(Object o)`方法的规范说：“当且仅当此集合至少包含一个元素e才能返回true”，以便`(o == null ? e == null : o.equals(e))`。该规范不应该被解释为暗示使用非空参数调用`Collection.contains(o)`将为任何元素e调用`o.equals(e)`。实现可以自由地实现优化，从而避免等待调用。例如，首先比较两个元素的哈希值（`Object.hashCode（）`规范保证具有不等的哈希码的两个对象不能相等）。更一般地，各种集合框架接口的实现可以随礼使用底层`Object`方法的执行行为，只要执行者认为合适。

执行递归遍历集合的一些集合操作可能会失败，并且对于自引用实例的异常，其中集合直接或间接包含自身。 这包括`clone（）`，`equals（）`，`hashCode（）`和`toString（）`方法。 实现可以可选地处理自参考场景，然而大多数当前实现不这样做。

上面这些是官方注解，给出了整个框架体系的概览。下面就从这个根接口开始学习集合框架。`Collection`接口的结构如下：

![Collection](http://ovn0i3kdg.bkt.clouddn.com/Collection.png)


## public interface Collection<E> extends Iterable<E>
类声明，可以继承了Iterable接口，说明子类必须实现Iterable中的抽象方法，即每个实现`Collection`的子类都应该实现一个方法来返回这个集合上的迭代器。

## int size();
返回集合元素的个数。如果数据大于 `Integer.MAX_VALUE`，就只能返回`Integer.MAX_VALUE`。

## boolean isEmpty();
是否是空集合。是的话就返回true。

## boolean contains(Object o);
集合是否包含某个特定的元素，包含的话返回true。一般的，当且仅当集合中至少包含一个元素o才会返回true。

## Iterator<E> iterator();
实现`Iterable`接口，返回此集合中元素的迭代器，这个迭代器不能保障元素返回顺序，除非这个集合是提供保证的某各类的实例。
> 什么是迭代器。

##  Object[] toArray();
返回一个包含此集合中所有元素的数组。如果此集合对其迭代返回的元素的顺序做出了某个保证，则此方法必须以相同的顺序返回元素。

返回的数据将是“安全”的，因为该集合不会保留对它的引用。（换句话说，这个方法必须分配一个新数组，即使这个数组是由数组支持的）。因此，调用者可以自由地修改返回的数组。此方法将位于数组和集合的API之间的桥梁。


## <T> T[] toArray(T[] a);
返回一个包含此集合中所有元素的数组。返回的数组的运行时类型是指定数组的运行时类型。如果集合适合指定的数组，则返回其中。否则将指定数组的运行时类型和此集合的大小分配一个新数组。

如果这个集合是适合指定的集合，有空余的空间（即该数组具有比此集合更多的元素空间，紧跟在集合结束后的数组中的元素将设置为null。（仅当调用者知道此集合不好喊任何控制元素时，此方法此有助于确定此集合的长度。）

如果此集合对齐迭代器返回的元素的顺序做出任何保证，则此方法必须以相同的顺序返回元素。和上一个`toArray`方法一样，此方法充当了数组和集合API之间的桥梁。此外，该方法允许精确地控制输出数组运行时类型，并且在某些情况下可以用于节省分配成本。

假设x是一个已知的只包含字符串的集合，以下代码可以用于将集合转储到新分配的String数组中：
```java
String [] y = x.toArray(new String[0]);
```
请注意， `toArray(new Object[0])` 和`toArray()`相同。


## boolean add(E e);
向集合中添加元素，这是一个可选操作（为什么可选？在这篇博文最初已经解释了，因为可能是破坏性方法）。如果添加成功了，则返回true，否则返回false。（比如有些集合不允许包含重复元素，则返回false）。

支持此操作的集合可能会限制这个被添加的元素e。特别的，一些集合将拒绝添加null元素，而其他集合将对可能添加的元素的类型施加限制。集合类应该在文档中明确规定可能天机哪些元素的限制。

如果集合拒绝添加特定元素，除了它已经包含该元素之外，它必须抛出一个异常（而不是返回false）。这保留了在该调用返回后，集合始终包含指定元素的不变量。


##  boolean remove(Object o);
从该集合中删除指定元素的单个实例（如果存在），这同样是一个可选操作。如果这个集合包含一个或多个此类元素则删除元素e，使得`(o == null ? e == null : o.equals(e))`。如果此集合包含指定的元素，也就是调用这个方法后集合改变了，那么该方法将返回true。


## boolean containsAll(Collection<?> c);
测试该集合是不是包含指定集合中的所有元素，是的话将返回true。

## boolean addAll(Collection<? extends E> c);
将指定集合中的所有元素添加到此集合（可选操作）。 如果指定的集合在操作进行中被修改，则此操作的行为是未定义的。 （这意味着如果指定的集合是此集合，则此调用的行为是未定义的，并且此集合是非空的。）

##  boolean removeAll(Collection<?> c);
删除指定集合中包含的所有此集合的元素（可选操作）。 此调用返回后，此集合将不包含与指定集合相同的元素。


## default boolean removeIf(Predicate<? super E> filter){...}
这是在接口中声明了一个非抽象方法，用`default`关键词修饰，是JDK1.8中的新特性。定义如下：
```java
default boolean removeIf(Predicate<? super E> filter) {
     Objects.requireNonNull(filter);
     boolean removed = false;
     final Iterator<E> each = iterator();
     while (each.hasNext()) {
         if (filter.test(each.next())) {
             each.remove();
             removed = true;
         }
     }
     return removed;
 }
```
其中的`Predicate`也是1.8的新特性。总之，该方法删除满足给定谓词的此集合的所有元素。 在迭代或谓词中抛出的错误或运行时异常被转发给调用者。要对接口的实现着有以下要求：默认实现使用`iterator（）`遍历集合的所有元素。 使用`Iterator.remove（）`删除每个匹配元素。 如果集合的迭代器不支持删除，则会在第一个匹配元素上抛出`UnsupportedOperationException`异常。

## boolean retainAll(Collection<?> c);
仅保留此集合中包含在指定集合中的元素（可选操作）。 换句话说，从该集合中删除所有不包含在指定集合中的元素。

##  void clear();
清空集合。该方法调用完成后，这个集合将是空集合。

##  boolean equals(Object o);
使用此集合来指定对象的等同性。

尽管`Collection`界面没有为`Object.equals`的一般合同添加任何规定，但是直接“实现Collection”接口的程序员（换句话说，创建一个`Collection`，而不是`Set`或`List`）必须保护如果他们选择覆盖`Object.equals`。没有必要这样做，最简单的行动是依靠`Object`的实现，但是实现者可能希望实现“值比较”来代替默认的“参考比较”。 （列表和集合接口要求这样的价值比较。）

`Object.equals`方法的一般合同规定，等于必须是对称的（换句话说，`a.equals（b）`当且仅当`b.equals（a）`）。 `List.equals`和`Set.equals`的合同规定列表仅等于其他列表，并设置为其他集合。因此，当将集合与任何列表或集合进行比较时，不会同时实现List或Set接口的集合类的自定义`equals`方法必须返回false。 （通过相同的逻辑，不可能编写正确实现`Set`和`List`接口的类。）


## int hashCode();
返回此集合的哈希码值。 虽然`Collection`接口对`Object.hashCode`方法的通用合同没有添加任何规定，但程序员应该注意，覆盖`Object.equals`方法的任何类也必须覆盖`Object.hashCode`方法，以满足`Object`的一般约定。 特别地，`c1.equals（c2）`意味着`c1.hashCode（）== c2.hashCode（）`。

## default Spliterator<E> spliterator(){...}
非抽象方法。定义如下：
```java
default Spliterator<E> spliterator() {
    return Spliterators.spliterator(this, 0);
}
```
在此集合中的元素上创建一个`Spliterator`。 实施应该记录分配器报告的特征值。 如果拼接器报告`Spliterator.SIZED`并且此集合不包含元素，则不需要报告此类特征值。
应该通过可以返回更高效的拼接器的子类覆盖默认实现。 为了保留`stream（）`和`parallelStream（）`）方法的预期懒惰行为，分配器应该具有`IMMUTABLE`或`CONCURRENT`的特征，或者具有后期绑定。 如果这些都不实用，那么重写类应该描述绑定器的绑定和结构干扰策略，并且应该使用`stream（）`和`parallelStream（）`方法来创建流，使用`Spliterator`的供应商，如：
```java
Stream<E> s = StreamSupport.stream(() -> spliterator(), spliteratorCharacteristics)
```
这些要求确保由`stream()`和`parallelStream()`方法生成的流将在终端流操作启动时反映集合的内容。

##  default Stream<E> stream(){...}
```java
default Stream<E> stream() {
   return StreamSupport.stream(spliterator(), false);
}
```

返回一个以此集合为源的顺序流。当`spliterator（）`方法无法返回`IMMUTABLE`，`CONCURRENT`或后期绑定的拼接器时，应该覆盖此方法。 （有关详细信息，请参阅spliterator（））

## default Stream<E> parallelStream(){...}
```java
default Stream<E> parallelStream() {
    return StreamSupport.stream(spliterator(), true);
}
```
返回一个可能并行的Stream，该集合作为其源。 该方法允许返回顺序流。当`spliterator（）`方法无法返回`IMMUTABLE`，`CONCURRENT`或后期绑定的拼接器时，应该覆盖此方法。 （有关详细信息，请参阅spliterator（））
