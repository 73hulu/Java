# SortedMap

`SortedMap`是一个接口，表明map中的映射关系将按照key的自然顺序或指定顺序进行排序，这种排序表现在以迭代器遍历容器的时候。要求，放入容器内的映射关系中的key必须实现`Comparable`接口或者被指定的比较器接受。接口定义如下：

![SortedMap](http://ovn0i3kdg.bkt.clouddn.com/SortedMap.png)

注意到其中`comparator`方法，它用来返回使用的比较器。


实现这个接口的类很多，典型的有`NavigableMap`，而`TreeMap`通过这`NavigableMap`这个接口实现了映射关系的比较。而`TreeMap`的实现是通过红黑树。红黑树是有特殊性质的平衡二叉排序树。这里排序的“序”就是我们所说的比较器。

我们一般采用`TreeSet`来实现映射关系的排序，比如：
```java
public static void main(String[] args) {
    HashMap<String,String> map=new HashMap<String, String>();
    map.put("a", "abc");
    map.put("b","de");
    map.put("c", "kg");
    for (Map.Entry<String,String> entry: map.entrySet()) {
        System.out.println("排序之前:"+entry.getKey()+" 值"+entry.getValue());

    }
    System.out.println("======================================================");
    SortedMap<String,String> sort=new TreeMap<String,String>(map);
    Set<Map.Entry<String,String>> entry1=sort.entrySet();
    Iterator<Map.Entry<String,String>> it=entry1.iterator();
    while(it.hasNext())
    {
        Map.Entry<String,String> entry=it.next();
        System.out.println("排序之后:"+entry.getKey()+" 值"+entry.getValue());
    }
}
```
需要说明，`HashMap`比`SortedMap`快，所以当不需要排序的时候，用`HashMap`，而如果需要排序的时，也先用`HashMap`存储映射关系，然后用`HashMap`的对象来创建`SortedMap`接口对象。就像上面的例子一样。

`SortedMap`在实际的应用中有广泛的应用，一个典型的例子就是生成微信签名，其api原文是：

> 设所有发送或者接收到的数据为集合M，将集合M内非空参数值的参数按照参数名ASCII码从小到大排序（字典序），使用URL键值对的格式（即key1=value1&key2=value2…）拼接成字符串stringA。

这里我们就可以用`SortedMap`来存储映射关系，然后拼接成字符串就行。
