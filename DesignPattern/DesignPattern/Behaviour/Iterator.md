# Iterator

迭代器模式。提供一种方法顺序访问一个聚合对象中各个元素，而又不暴露该对象的内部表示。


迭代器是为容器服务的。容器就是那些能够容纳对象的类型。迭代器模式就是为了解决遍历这些容器中的元素而诞生的。容器只需要管理元素的增加，遍历的任务就交给迭代器。

![迭代器模式](https://ws2.sinaimg.cn/large/006tNbRwly1fx1xwaetujj30n60k4gml.jpg)

有以下角色：
* Iterator抽象迭代器：负责访问和遍历元素的接口，基本上是3个固定的方法：`first()`、`next()`、`isNext()`。
* ConcreteIterator具体迭代器：用来实现迭代器接口，完成容器元素的遍历。
* Aggregate抽象容器：负责提供创建具体迭代器角色的接口，java中是`iterator()`方法。
* ConcreteAggregate具体容器：实现容器接口定义的方法，创建容纳迭代器的对象。

下面是通用源码：
```java
// 抽象迭代器
public iteraface Iterator{
  // 遍历到下一个元素
  public Object next();
  // 是否应遍历到尾部
  public boolean hasNext();
  // 删除当前指向的元素
  public boolean remove();
}
// 具体迭代器
public class ConcreteIterator implements Iterator{
  private List list= new ArrayList();
  // 定义当前游标
  public int cursor = 0;
  @SuppressWarning("uncheck")
  public ConcreteIterator(List list){
    this.list = list;
  }

  // 判断是否到达尾部
  public boolean hasNext(){
    return this.cursor != this.list.size();    
  }
  // 返回下一个元素
  public Object next(){
    Object result = null;
    if(this.hasNext()){
      result = this.list.get(this.cursor++);
    }else {
      result = null;
    }
    return result;
  }
  // 删除当前元素
  public boolean remove(){
    this.list.remove(this.cursor);
    return true;
  }
}
// 抽象容器
public interface Aggregate{
  public void add(Object object);
  public void remove(Object object);
  public Iterator iterator();
}
// 具体容器类
public class ConcreteAggregate implements Aggregate{
  // 容纳对象的容器
  private List list = new ArrayList();

  public void add(Object object){
    this.list.add(object);
  }

  public Iterator iterator(){
    return new ConcreteIterator(this.vector);
  }

  public void remove(Object object){
    this.remove(object);
  }
}
```
这是一种已经没落的模式，基本上没有人会单独写一个迭代器，除非是产品性质的开发。一般来说，Java提供的Iterator已经能满足要求了。

参考
* [JAVA 设计模式 迭代器模式](http://www.cnblogs.com/jingmoxukong/p/4236056.html)
