# TreeMap

红黑树的实现。真正变态级的数据结构，更多红黑树的概念和操作参考数据结构相关博文。

该类的结构如下：

![TreeMap_1](http://ovn0i3kdg.bkt.clouddn.com/%08TreeMap_1.png)
![TreeMap_2](http://ovn0i3kdg.bkt.clouddn.com/TreeMap_2.png)

### public class TreeMap<K,V> extends AbstractMap<K,V> implements NavigableMap<K,V>, Cloneable, java.io.Serializable
类声明，继承自`AbstractMap`，实现`NavigableMap`、`Cloneable`、`Serializable`三个接口。其中`AbstractMap`表明TreeMap为一个Map即支持key-value的集合， `NavigableMap`（更多）则意味着它支持一系列的导航方法，具备针对给定搜索目标返回最接近匹配项的导航方法 。


### 映射关系Entry
`TreeMap`继承了抽象了`AbstractMap`，重写了映射关系的数据结构，定义如下：
```java
static final class Entry<K,V> implements Map.Entry<K,V> {
    K key;
    V value;
    Entry<K,V> left;
    Entry<K,V> right;
    Entry<K,V> parent;
    boolean color = BLACK;

    /**
     * Make a new cell with given key, value, and parent, and with
     * {@code null} child links, and BLACK color.
     */
    Entry(K key, V value, Entry<K,V> parent) {
        this.key = key;
        this.value = value;
        this.parent = parent;
    }

    /**
     * Returns the key.
     *
     * @return the key
     */
    public K getKey() {
        return key;
    }

    /**
     * Returns the value associated with the key.
     *
     * @return the value associated with the key
     */
    public V getValue() {
        return value;
    }

    /**
     * Replaces the value currently associated with the key with the given
     * value.
     *
     * @return the value associated with the key before this method was
     *         called
     */
    public V setValue(V value) {
        V oldValue = this.value;
        this.value = value;
        return oldValue;
    }

    public boolean equals(Object o) {
        if (!(o instanceof Map.Entry))
            return false;
        Map.Entry<?,?> e = (Map.Entry<?,?>)o;

        return valEquals(key,e.getKey()) && valEquals(value,e.getValue());
    }

    public int hashCode() {
        int keyHash = (key==null ? 0 : key.hashCode());
        int valueHash = (value==null ? 0 : value.hashCode());
        return keyHash ^ valueHash;
    }

    public String toString() {
        return key + "=" + value;
    }
}
```
每个树的节点都是一个映射关系，即`Entry`的实例，其中包含属性key(键)
、value(值)、left(左孩子)、right(右孩子)、parent(父节点)和颜色（默认为黑色），注意`BLACK`和`RED`都是布尔型的变量，分别对应true和false。而对于一棵树来说，只需要保存根节点即可。所以`TreeMap`中保存了根节点：
```java
 private transient Entry<K,V> root;
```

### 构造方法
定义了四种构造方法。
```java
public TreeMap() {
   comparator = null;
}

public TreeMap(Comparator<? super K> comparator) {
    this.comparator = comparator;
}

public TreeMap(Map<? extends K, ? extends V> m) {
   comparator = null;
   putAll(m);
}

public TreeMap(SortedMap<K, ? extends V> m) {
    comparator = m.comparator();
    try {
        buildFromSorted(m.size(), m.entrySet().iterator(), null, null);
    } catch (java.io.IOException cannotHappen) {
    } catch (ClassNotFoundException cannotHappen) {
    }
}
```
可以看出，`TreeMap`的构造中最重要的是对`comparator`的赋值，该变量定义如下：
```java
private final Comparator<? super K> comparator;
```
可以看到，这是一个比较器，我们可以认为指定比较器，或者为null的时候表明是自然排序，所以`TreeMap`是一个可以自定义排序键值对容器。

可以根据一个`SortedMap`来创建红黑树。


红黑树的核心在于节点插入和删除操作，分别对应`TreeMap`的`put`和`delete`方法。下面就着重将这两个方法的实现。

### public V put(K key, V value)
这个方法用来插入一个节点到红黑树中，如果红黑树中已经包含该key节点，则将原value值将被覆盖。
在看这个方法之前，先复习一下红黑树的插入算法。

红黑树的插入算法分为以下步骤：
1. 定位：红黑树首先是一棵BST树，所以按照BST的插入规则定位到应该插入的位置。
2. 着色：将新节点着色为红色。
3. 如果该节点是根节点，则置为黑色，插入算法结束。
4. 如果该节点的父节点是黑色，则什么都不用做，插入算法结束。
5. 如果该节点的父亲节点是红色，那么需要考虑叔叔节点的颜色：
  1. 如果叔叔节点(u)是红色，即此时叔叔节点和父亲节点是红色，祖父节点为黑色，则此时将父亲节点和叔叔节点置为黑色，将祖父节点置为红色，以祖父节点为当前节点，递归匹配情况。
  2. 如果叔叔节点(u)是黑色，此时根据新节点(n)、父亲节点(p)和祖父节点(g)的关系可以分为四种情况。
    1. 新节点(n)是父亲节点(p)的左节点，父亲节点(p)是祖父节点(g)的左节点，此时需要进行LL旋转，即以父亲节点(p)为轴心进行单项右旋。之后将父亲节点(p)设置为黑色，祖父节点(g)设置为红色。
    2. 新节点(n)是父亲节点(p)的右节点，父亲节点(p)是祖父节点(g)的右节点，此时需要进行RR旋转，即以父亲节点(p)为轴心进行单项左旋。之后将父亲节点(p)设置为黑色，祖父节点(g)设置为红色。
    3. 新节点(n)是父亲节点(p)的左节点，父亲节点(p)是祖父节点(g)的右节点，此时需要进行RL旋转，即以当前节点(n)为轴心进行先进行左旋后进行右旋。之后将当前节点(n)设置为黑色，祖父节点(g)设置为红色。
    4. 新节点(n)是父亲节点(p)的右节点，父亲节点(p)是祖父节点(g)的左节点，此时需要进行LR旋转，即以当前节点(n)为轴心进行先进行右旋后进行左旋。之后将当前节点(n)设置为黑色，祖父节点(g)设置为红色。
> LL旋转是单项右旋，以父节点为轴心；RR旋转是单项左旋，以父节点为轴心。LR旋转是先左旋后右旋，以当前节点为轴心；RL旋转是先右旋后左旋，以当前节点为轴心。

有了以上的认知，我们来看看`TreeMap`中的put方法如何实现节点插入：
```java

public V put(K key, V value) {
   //用t表示二叉树的当前节点
    Entry<K,V> t = root;
    //t为null表示一个空树，即TreeMap中没有任何元素，直接插入
    if (t == null) {
        //比较key值，个人觉得这句代码没有任何意义，空树还需要比较、排序？
        compare(key, key); // type (and possibly null) check
        //将新的key-value键值对创建为一个Entry节点，并将该节点赋予给root
        root = new Entry<>(key, value, null);
        //容器的size = 1，表示TreeMap集合中存在一个元素
        size = 1;
        //修改次数 + 1
        modCount++;
        return null;
    }
    int cmp;     //cmp表示key排序的返回结果
    Entry<K,V> parent;   //父节点
    // split comparator and comparable paths
    Comparator<? super K> cpr = comparator;    //指定的排序算法
    //如果cpr不为空，则采用既定的排序算法进行创建TreeMap集合
    if (cpr != null) {
        do {
            parent = t;      //parent指向上次循环后的t
            //比较新增节点的key和当前节点key的大小
            cmp = cpr.compare(key, t.key);
            //cmp返回值小于0，表示新增节点的key小于当前节点的key，则以当前节点的左子节点作为新的当前节点
            if (cmp < 0)
                t = t.left;
            //cmp返回值大于0，表示新增节点的key大于当前节点的key，则以当前节点的右子节点作为新的当前节点
            else if (cmp > 0)
                t = t.right;
            //cmp返回值等于0，表示两个key值相等，则新值覆盖旧值，并返回新值
            else
                return t.setValue(value);
        } while (t != null);
    }
    //如果cpr为空，则采用默认的排序算法进行创建TreeMap集合
    else {
        if (key == null)     //key值为空抛出异常
            throw new NullPointerException();
        /* 下面处理过程和上面一样 */
        Comparable<? super K> k = (Comparable<? super K>) key;
        do {
            parent = t;
            cmp = k.compareTo(t.key);
            if (cmp < 0)
                t = t.left;
            else if (cmp > 0)
                t = t.right;
            else
                return t.setValue(value);
        } while (t != null);
    }
    //将新增节点当做parent的子节点
    Entry<K,V> e = new Entry<>(key, value, parent);
    //如果新增节点的key小于parent的key，则当做左子节点
    if (cmp < 0)
        parent.left = e;
  //如果新增节点的key大于parent的key，则当做右子节点
    else
        parent.right = e;
    /*
     *  上面已经完成了排序二叉树的的构建，将新增节点插入该树中的合适位置
     *  下面fixAfterInsertion()方法就是对这棵树进行调整、平衡，具体过程参考上面的五种情况
     */
    fixAfterInsertion(e);
    //TreeMap元素数量 + 1
    size++;
    //TreeMap容器修改次数 + 1
    modCount++;
    return null;
}
```
上面的代码很好理解，实际上执行的是BST节点插入的过程，不同的是，这里可以根据开发者是否制定了比较器来决定插入的BST的插入顺序。对于红黑树来说，真正重要的是`fixAfterInsertion`方法，这个方法是进行树调整、平衡的过程，使其成为一个具备红黑树特征的平衡排序二叉树。其定义如下：
```java
/**
 * 新增节点后的修复操作
 * x 表示新增节点
 */
 private void fixAfterInsertion(Entry<K,V> x) {
        x.color = RED;    //新增节点的颜色为红色

        //循环 直到 x不是根节点，且x的父节点为红色
        while (x != null && x != root && x.parent.color == RED) {
            //如果X的父节点（P）是其祖父节点（G）的左节点
            if (parentOf(x) == leftOf(parentOf(parentOf(x)))) {
                //获取X的叔节点(U)
                Entry<K,V> y = rightOf(parentOf(parentOf(x)));
                //如果X的叔节点（U） 为红色，此时将父节点和叔节点置为黑色，祖父节点置为红色，以祖父节点为当前节点，继续进行。
                if (colorOf(y) == RED) {     
                    //将X的父节点（P）设置为黑色
                    setColor(parentOf(x), BLACK);
                    //将X的叔节点（U）设置为黑色
                    setColor(y, BLACK);
                    //将X的父节点的父节点（G）设置红色
                    setColor(parentOf(parentOf(x)), RED);
                    x = parentOf(parentOf(x));
                }
                //如果X的叔节点（U为黑色）；这里会存在两种情况
                else {   
                    //如果X节点为其父节点（P）的右子树，则先进行左旋在进行右旋，旋转中心是其父节点
                    if (x == rightOf(parentOf(x))) {
                        //将X的父节点作为X
                        x = parentOf(x);
                        //左旋转
                        rotateLeft(x);
                    }
                    //将X的父节点（P）设置为黑色
                    setColor(parentOf(x), BLACK);
                    //将X的父节点的父节点（G）设置红色
                    setColor(parentOf(parentOf(x)), RED);
                    //以X的父节点的父节点（G）为中心右旋转
                    rotateRight(parentOf(parentOf(x)));
                }
            }
            //如果X的父节点（P）是其父节点的父节点（G）的右节点
            else {
                //获取X的叔节点（U）
                Entry<K,V> y = leftOf(parentOf(parentOf(x)));
              //如果X的叔节点（U） 为红色，此时将父节点和叔节点置为黑色，祖父节点置为红色，以祖父节点为当前节点，继续进行。
                if (colorOf(y) == RED) {
                    //将X的父节点（P）设置为黑色
                    setColor(parentOf(x), BLACK);
                    //将X的叔节点（U）设置为黑色
                    setColor(y, BLACK);
                    //将X的父节点的父节点（G）设置红色
                    setColor(parentOf(parentOf(x)), RED);
                    x = parentOf(parentOf(x));
                }
              //如果X的叔节点（U为黑色）；这里会存在两种情况
                else {
                    //如果X节点为其父节点（P）的左子树，则先进行右旋后进行左旋，旋转中心是其父节点
                    if (x == leftOf(parentOf(x))) {
                        //将X的父节点作为X
                        x = parentOf(x);
                       //右旋转
                        rotateRight(x);
                    }
                    //（情况五）
                    //将X的父节点（P）设置为黑色
                    setColor(parentOf(x), BLACK);
                    //将X的父节点的父节点（G）设置红色
                    setColor(parentOf(parentOf(x)), RED);
                    //以X的父节点的父节点（G）为中心右旋转
                    rotateLeft(parentOf(parentOf(x)));
                }
            }
        }
        //将根节点G强制设置为黑色
        root.color = BLACK;
    }

```
以上是红黑树调整的过程实现，其中坐旋转`rotateLeft`和`rotateRight`的代码实现如下：
```java
// 左旋转
private void rotateLeft(Entry<K,V> p) {
    if (p != null) {
        //获取P的右子节点，
        Entry<K,V> r = p.right;
        //将R的左子树设置为P的右子树
        p.right = r.left;
        //若R的左子树不为空，则将P设置为R左子树的父亲
        if (r.left != null)
            r.left.parent = p;
        //将P的父亲设置R的父亲
        r.parent = p.parent;
        //如果P的父亲为空，则将R设置为跟节点
        if (p.parent == null)
            root = r;
        //如果P为其父节点（G）的左子树，则将R设置为P父节点(G)左子树
        else if (p.parent.left == p)
            p.parent.left = r;
        //否则R设置为P的父节点（G）的右子树
        else
            p.parent.right = r;
        //将P设置为R的左子树
        r.left = p;
        //将R设置为P的父节点
        p.parent = r;
    }
}
```
所谓左旋，就是将新增节点（N）当做其父节点（P），将其父节点P当做新增节点（N）的左子节点。即：G.left ---> N ,N.left ---> P，该过程的动画如图所示（途中的E节点就相当于`rotateLeft`方法的参数p，即为轴点）

![左旋](https://images0.cnblogs.com/blog/94031/201403/270024492492764.gif)

右旋方法定义如下：
```Java
private void rotateLeft(Entry<K,V> p) {
    if (p != null) {
        //获取P的右子节点，其实这里就相当于新增节点N（情况四而言）
        Entry<K,V> r = p.right;
        //将R的左子树设置为P的右子树
        p.right = r.left;
        //若R的左子树不为空，则将P设置为R左子树的父亲
        if (r.left != null)
            r.left.parent = p;
        //将P的父亲设置R的父亲
        r.parent = p.parent;
        //如果P的父亲为空，则将R设置为跟节点
        if (p.parent == null)
            root = r;
        //如果P为其父节点（G）的左子树，则将R设置为P父节点(G)左子树
        else if (p.parent.left == p)
            p.parent.left = r;
        //否则R设置为P的父节点（G）的右子树
        else
            p.parent.right = r;
        //将P设置为R的左子树
        r.left = p;
        //将R设置为P的父节点
        p.parent = r;
    }
}
```
过程与左旋转类似，不再赘述过程，其动画如下：
![右旋](http://img.blog.csdn.net/20140523091755703?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2hlbnNzeQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

除此之外还有红黑树节点的着色方法`setColor`，就是将变更节点颜色，定义如下：
```java
private static <K,V> void setColor(Entry<K,V> p, boolean c) {
   if (p != null)
       p.color = c;
}
```
至此，红黑树的插入操作完成。


### public V remove(Object key)
这个方法用来寻找key值对应的树节点，找到后删除该节点。同样，在分析代码之前，我们先回顾下红黑树的删除操作：

红黑树的删除算法可以分为下面几个步骤：
1. 定位：按照BST删除算法定位到应该被删除的那个节点，注意回顾一下二叉树节点删除算法：当被删除节点是叶子节点的时候，直接删除；当被删除节点只具有右子树或左子树之一的时候，将该节点的父节点的相应指针连接到该节点的那个子树上，然后删除该节点；当该节点兼备左右子树的时候，将该节点和左子树中的节点或右子树中的最小节点做交换，然后问题转换成删除前面两种情况。所以这里需要注意，最终需要被删除的那个节点并不一定是最初树中的位置。
2. 如果最终被删除的节点(n)是红色的，那么不需要调整，红黑树的删除算法结束。因为删除一个红色节点，不影响红黑树的平衡性。
3. 如果最终被删除的节点(n)是黑色的，那么该节点所在的路径上黑色节点总数减少1，红黑树失去平衡，需要调整。这时候需要考虑到其父节点(p)、兄弟节点(w)的情况分别进行处理，假设当前节点(n)是父节点(p)的左子节点，**n是p的右子节点的情况可以用镜像堆成考虑。**
  1. 如果兄弟节点(w)是红色的，此时需要交换父节点(p)和兄弟节点(w)的颜色，然后以父节点为轴进行左旋，完成后，当前节点(n)将具有新的兄弟节点。如下图：
  ![左旋](https://images0.cnblogs.com/blog/405877/201403/101959160429284.jpg)
  之后将变成下面一种情况。
  2. 如果兄弟节点(w)是黑色的，此时需要考虑到兄弟节点的左右节点的颜色
    1. 如果兄弟节点(w)的两个子节点(x和y)都是黑色，那么此时将兄弟节点(w)设定为红色，将父节点(p)作为当前节点，递归匹配情况。
    ![兄弟节点为黑色，两个子节点都是黑色](https://images0.cnblogs.com/blog/405877/201403/101959393504066.jpg)
    2. 如果兄弟节点的左节点(x)为红色，右节点(y)为黑色，则需要将左节点(x)设置为黑色，兄弟节点(w)设置为红色，并以左节点(x)为轴进行右旋转。
    ![右旋](https://images0.cnblogs.com/blog/405877/201403/101959498418258.jpg)
    3. 如果兄弟节点的左节点(x)任意，右节点(y)为红色，则需要右节点(y)设置为黑色，并交换兄弟节点(w)和父节点(p)的颜色，再以兄弟节点(w)为轴进行左旋转。
    ![左旋](https://images0.cnblogs.com/blog/405877/201307/13124731-694f468d0fbf4b618c987094306c8d94.jpg)
    至此调整结束。

> 注意，以上情况是当前节点是父节点的左节点的情况，可用镜像对对称来处理当前节点是父节点的右节点的情况，注意相关操作也成镜像堆成，比如当兄弟节点是黑节点，其做左节点为红色，右节点任意，此时需要将左节点设置为红色，然后交换兄弟节点和父节点的颜色，再以兄弟节点为轴进行右旋转。

`remove`方法的源码如下：
```java
public V remove(Object key) {
    Entry<K,V> p = getEntry(key);
    if (p == null)
        return null;

    V oldValue = p.value;
    deleteEntry(p);
    return oldValue;
}
```
其中`deleteEntry`是关键，定义如下：
```java
private void deleteEntry(Entry<K,V> p) {
    modCount++;      //修改次数 +1
    size--;          //元素个数 -1

    /*
     * 被删除节点的左子树和右子树都不为空，那么就用 p节点的中序后继节点代替 p 节点
     * successor(P)方法为寻找P的替代节点。规则是右分支最左边，或者 左分支最右边的节点
     * ---------------------（1）
     */
    if (p.left != null && p.right != null) {  
        Entry<K,V> s = successor(p);
        p.key = s.key;
        p.value = s.value;
        p = s;
    }

    //replacement为替代节点，如果P的左子树存在那么就用左子树替代，否则用右子树替代
    Entry<K,V> replacement = (p.left != null ? p.left : p.right);

    /*
     * 删除节点，
     * -----------------------（2）
     */
    //如果替代节点不为空
    if (replacement != null) {
        replacement.parent = p.parent;
        /*
         *replacement来替代P节点
         */
        //若P没有父节点，则跟节点直接变成replacement
        if (p.parent == null)
            root = replacement;
        //如果P为左节点，则用replacement来替代为左节点
        else if (p == p.parent.left)
            p.parent.left  = replacement;
      //如果P为右节点，则用replacement来替代为右节点
        else
            p.parent.right = replacement;

        //同时将P节点从这棵树中剔除掉
        p.left = p.right = p.parent = null;

        /*
         * 若P为红色直接删除，红黑树保持平衡
         * 但是若P为黑色，则需要调整红黑树使其保持平衡
         */
        if (p.color == BLACK)
            fixAfterDeletion(replacement);
    } else if (p.parent == null) {     //p没有父节点，表示为P根节点，直接删除即可
        root = null;
    } else {      //P节点不存在子节点，直接删除即可
        if (p.color == BLACK)         //如果P节点的颜色为黑色，对红黑树进行调整
            fixAfterDeletion(p);

        //删除P节点
        if (p.parent != null) {
            if (p == p.parent.left)
                p.parent.left = null;
            else if (p == p.parent.right)
                p.parent.right = null;
            p.parent = null;
        }
    }
}
```
其中`successor`方法是寻找左子树或右子树中的代替被删除节点的节点，定义如下：
```java

static <K,V> TreeMap.Entry<K,V> successor(Entry<K,V> t) {
    if (t == null)
        return null;
    /*
     * 寻找右子树的最左子树
     */
    else if (t.right != null) {
        Entry<K,V> p = t.right;
        while (p.left != null)
            p = p.left;
        return p;
    }
    /*
     * 选择左子树的最右子树
     */
    else {
        Entry<K,V> p = t.parent;
        Entry<K,V> ch = t;
        while (p != null && ch == p.right) {
            ch = p;
            p = p.parent;
        }
        return p;
    }
}
```
从代码中看出，优先选择了右子树中的最小值作为代替。另外在进行节点删除后，需要进行调整和平衡，重点在于`fixAfterDeletion`的实现：
```java
private void fixAfterDeletion(Entry<K,V> x) {
    // 删除节点需要一直迭代，知道 直到 x 不是根节点，且 x 的颜色是黑色
    while (x != root && colorOf(x) == BLACK) {
        if (x == leftOf(parentOf(x))) {      //若X节点为左节点
            //获取其兄弟节点
            Entry<K,V> sib = rightOf(parentOf(x));

            /*
             * 如果兄弟节点为红色
             * 策略：改变W、P的颜色，然后以父节点为轴进行一次左旋转
             */
            if (colorOf(sib) == RED) {     
                setColor(sib, BLACK);     
                setColor(parentOf(x), RED);  
                rotateLeft(parentOf(x));
                sib = rightOf(parentOf(x));
            }

            /*
             * 若兄弟节点的两个子节点都为黑色
             * 策略：将兄弟节点变成红色，将父亲节点作为当前节点
             */
            if (colorOf(leftOf(sib))  == BLACK &&
                colorOf(rightOf(sib)) == BLACK) {
                setColor(sib, RED);
                x = parentOf(x);
            }
            else {
                /*
                 * 如果兄弟节点只有右子树为黑色
                 * 策略：将兄弟节点与其左子树进行颜色互换然后进行右转
                 */
                if (colorOf(rightOf(sib)) == BLACK) {
                    setColor(leftOf(sib), BLACK);
                    setColor(sib, RED);
                    rotateRight(sib);
                    sib = rightOf(parentOf(x));
                }
                /*
                 *策略：交换兄弟节点和父节点的颜色，
                 *同时将兄弟节点右子树设置为黑色，最后左旋转
                 */
                setColor(sib, colorOf(parentOf(x)));
                setColor(parentOf(x), BLACK);
                setColor(rightOf(sib), BLACK);
                rotateLeft(parentOf(x));
                x = root;
            }
        }

        /**
         * X节点为右节点与其为做节点处理过程差不多，这里就不在累述了
         */
        else {
            Entry<K,V> sib = leftOf(parentOf(x));

            if (colorOf(sib) == RED) {
                setColor(sib, BLACK);
                setColor(parentOf(x), RED);
                rotateRight(parentOf(x));
                sib = leftOf(parentOf(x));
            }

            if (colorOf(rightOf(sib)) == BLACK &&
                colorOf(leftOf(sib)) == BLACK) {
                setColor(sib, RED);
                x = parentOf(x);
            } else {
                if (colorOf(leftOf(sib)) == BLACK) {
                    setColor(rightOf(sib), BLACK);
                    setColor(sib, RED);
                    rotateLeft(sib);
                    sib = leftOf(parentOf(x));
                }
                setColor(sib, colorOf(parentOf(x)));
                setColor(parentOf(x), BLACK);
                setColor(leftOf(sib), BLACK);
                rotateRight(parentOf(x));
                x = root;
            }
        }
    }
    setColor(x, BLACK);
}
```
红黑树的删除操作比插入操作更加困难，所以慢慢掌握吧，别着急。

参考
* [红黑树数据结构剖析](http://www.cnblogs.com/fanzhidongyzby/p/3187912.html)
* [treemap原理](http://blog.csdn.net/zhangyuan19880606/article/details/51234420)
