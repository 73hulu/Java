# LinkedHashMap
学习`HashMap`源码可以知道，`HashMap`的存储顺序和遍历顺序没有必然的联系。如果我们需要保证存储顺序和遍历顺序的统一，则需要使用`LinkedHashMap`这个类。

![LinkedHashMap](http://ovn0i3kdg.bkt.clouddn.com/LinkedHashMap.png)

## public class LinkedHashMap<K,V> extends HashMap<K,V> implements Map<K,V>
类声明，继承自`HashMap`。

> HashMap的数据结构为： 数组 + 单向链表 + 红黑树
> LinkedHashMap的数据结构为: 数组 + 单向链表 + 红黑树 + 双向链表

## 构造方法
重载了5种构造方法。所有方法的第一步都是使用`super`来初始化容量和装载因子。第二部分是给`accessOrder`赋值。这是一个`boolean`量，其作用是定义`LinkedHashMap`的顺序：
* true : 基于访问的顺序，即遍历的时候按照LUR的顺序
* false : 基于插入的顺序，即遍历的时候按照FIFO的顺序

这是什么意思呢？下面是一个测试程序：
```Java
public class LinkedHashMapTest {
    public static void main(String[] args) {

        Map<Integer, Character> map = new LinkedHashMap<Integer, Character>(16, 0.75f, true);
        map.put(1, 'a');
        map.put(2, 'b');
        map.put(3, 'b');

        display(map);//map初始化之后，不管accessOrder的值是false还是true，这里都将按照初始化的顺序进行打印

        //此处对map进行了访问
        map.get(1);
        map.get(2);

        display(map);//由于初始化map的时，accessOrder设置成true，即基于访问顺序，所以此处将按照312的顺序打印

    }

    public static void display(Map<Integer, Character> map){
        Iterator<Map.Entry<Integer, Character>> iterator = map.entrySet().iterator();
        while (iterator.hasNext()){
            Map.Entry<Integer, Character> entry = iterator.next();
            System.out.println(entry.getKey() + " : " + entry.getValue());
        }
    }
}
```
从上例可以看出，当`accessOrder`设置为true的顺序时，`LinkedList`将基于访问的顺序，什么意思呢？就是使用LUR(最近最少被使用的调度算法)，将最近被访问的元素加到了最后。

从`LinkedHashMap`的构造方法中并不能看出它是如何维护元素的顺序的。实际上，`LinkedHashMap`仍旧采用`HashMap`来操作数据结构，另外使用了`LinkedList`来维护插入元素的先后顺序。下面就比较一下`HashMap`与`LinkedHashMap`中元素节点的数据结构的不同：
```Java
//HashMap
static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;
}
// LinkedHashMap
static class Entry<K,V> extends HashMap.Node<K,V> {
    Entry<K,V> before, after;
    Entry(int hash, K key, V value, Node<K,V> next) {
        super(hash, key, value, next);
    }
}
//双向链表的头结点
transient LinkedHashMap.Entry<K,V> head;

//双向链表的尾节点
transient LinkedHashMap.Entry<K,V> tail;

```
在`HashMap`中，内部类`Node`作为了节点的数据结构，其中具有属性`hash`、`key`、`value`和`next`，`HashMap`整体采取了哈希桶+单向链表的数据结构来存储互数据。而`LinkedHashMap`中，内部类`Entry`作为了节点的数据结构，它继承自`HashMap`的内部类`Node`，比它多了一两个属性before和after，所以`LinkedHashMap`是利用双向列表来维护元素的访问顺序的。看到这里可能有疑问呢，next和after不是一个意思么？不是的！！！千万要记住，**next用来维护HashMap上指定table上的连接的Entry的顺序，而before和after是用来维护Entry的插入顺序的**。

在`LinkedHashMap`的构造方法中，并未看到对于双向链表的创建和使用。这些操作发生在添加元素的时候。当添加元素的时候，将会调用继承自`HashMap`的`putVal`方法，这个方法中将判断是否已经存在该节点，当不存在的情况下，根据当前是`Node`类型还是`TreeNode`类型来new一个新的节点，即调用`newNode`或`newTreeNode`方法。在`LinkedHashMap`中这两个方法被重写：
```Java

Node<K,V> newNode(int hash, K key, V value, Node<K,V> e) {
    LinkedHashMap.Entry<K,V> p =
        new LinkedHashMap.Entry<K,V>(hash, key, value, e);
    linkNodeLast(p);
    return p;
}

TreeNode<K,V> newTreeNode(int hash, K key, V value, Node<K,V> next) {
    TreeNode<K,V> p = new TreeNode<K,V>(hash, key, value, next);
    linkNodeLast(p);
    return p;
}

private void linkNodeLast(LinkedHashMap.Entry<K,V> p) {
    LinkedHashMap.Entry<K,V> last = tail;
    tail = p;
    if (last == null)
        head = p;
    else {
        p.before = last;
        last.after = p;
    }
}
```
可以看到，`newNode`中除了创建一个`Node`的实例外，还将这个节点链接到双向列表的最后。`newTreeNode`中，`TreeNode`是`LinkedHashMap.Entry`的子类，所以可以将其链接到双向链表的最后。

所以这样一来，`LinkedHashMap`的结构大致是这样：

![LinkedHashMap](http://img.blog.csdn.net/20170412153450906?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMjQwMzI5MA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

`HashMap`中不存在before和after域，所以遍历的时候只是按照哈希桶和链表进行访问，并不能保证按照元素的插入顺序进行访问。而`LinkedHashMap`维护了一个双向链表，这个链表的顺序就是访问访问顺序，所以能够按照插入顺序进行访问。

在对元素进行插入和删除的时候，除了对哈希桶和链表本身做增加和删除的操作，还需要修改双向链表，而这个操作在`afterNodeInsertion`和`afterNodeRemoval`中进行。这两个方法在`HashMap`中定义，但是方法体为空，在`LinkedHashMap`中重写了这两个方法。
```Java
void afterNodeInsertion(boolean evict) { // possibly remove eldest
    LinkedHashMap.Entry<K,V> first;
    if (evict && (first = head) != null && removeEldestEntry(first)) {
        K key = first.key;
        removeNode(hash(key), key, null, false, true);
    }
}

void afterNodeRemoval(Node<K,V> e) { // unlink
    LinkedHashMap.Entry<K,V> p =
        (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
    p.before = p.after = null;
    if (b == null)
        head = a;
    else
        b.after = a;
    if (a == null)
        tail = b;
    else
        a.before = b;
}
```

之前我们说到，在创建`LinkedHashMap`的时候，可以定义`accessOrder`为true，这样可以基于访问顺序进行访问，即最近被访问的元素总会被至于双向链表的最后。这种实现在方法`afterNodeAccess`中体现，同样，该方法定义在`HashMap`中，方法体为空，在`LinkedHashMap`被重写：
```Java
void afterNodeAccess(Node<K,V> e) { // move node to last
    LinkedHashMap.Entry<K,V> last;
    if (accessOrder && (last = tail) != e) {
        LinkedHashMap.Entry<K,V> p =
            (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
        p.after = null;
        if (b == null)
            head = a;
        else
            b.after = a;
        if (a != null)
            a.before = b;
        else
            last = b;
        if (last == null)
            head = p;
        else {
            p.before = last;
            last.after = p;
        }
        tail = p;
        ++modCount;
    }
}
```
它实际上就是修改了中双向链表中最后被访问的元素节点的两个指针，使得该节点处于双向链表的最后。而这个方法在任何元素访问的方法（比如get，replace等）中被调用。

> LinkedHashMap 的内容还需要细纠！

参考
* [初识LinkedHashMap](https://www.cnblogs.com/xrq730/p/5052323.html)
* [【集合框架】JDK1.8源码分析之LinkedHashMap（二）](https://www.cnblogs.com/leesf456/p/5248868.html)
