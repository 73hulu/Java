# Memento
备忘录模式。在不破坏封闭的前提下，捕获一个对象的内部状态，并在该对象之外保存这个状态。这样以后就可将该对象恢复到原先保存的状态。


![备忘录模式](https://ws4.sinaimg.cn/large/006tNbRwly1fx325g2x91j30qq06a3z5.jpg)

有以下角色：
* Oirginator（发起人角色）：记录当前时刻的内部状态，负责定义哪些属于备份范畴的状态，负责创建和恢复备忘录数据
* Memento（备忘录）：负责存储Originator发起人对象内部的状态，在需要的时候提供发起人需要的内部装填。
* Caretaker（备忘录管理员角色）：对备忘录进行关系，保存和提供备忘录。存在的主要目录是避免客户端直接Memeto进行操作，符合迪米特法则。


```java
// 发起人角色
public class Originator{
  // 内部角色
  private String state = "";
  public String getState(){
    return state;
  }
  public void setState(String state){
    this.state = state;
  }
  // 创建一个备忘录
  public Memeto createMememto(){
    return new Memento(this.state);
  }
  // 恢复一个备忘录
  public void restoreMemento(Memento _memento){
    this.setState(_memento.getState());
  }
}
// 备忘录角色
public class Memento({
  // 发起人的内部状态
  private String state = "";
  // 构造函数传递参数
  public Memento(String _state){
    this.state = _state;
  }
  public String getState(){
    return state;
  }
  public void setState(String state){
    this.state = state;
  }
}
// 备忘录管理员角色
public class Caretaker{
  // 备忘录对象
  private Memento memento;
  public Memento getMemento(){
    return memento;
  }
  public void setMemento(Memento memento){
    this.memento = memento;
  }
}
// 场景类
public class Client{
  public static void main(String[] args){
    // 定义 发起人
    Originator originator = new Originator();
    // 定义备忘录管理员
    Caretaker caretaker = new Caretaker();
    // 创建一个备忘录
    caretaker.setMemento(originator.createMememto());
    // 恢复一个备忘录
    originator.restoreMemento(caretaker.getMemento());
  }
}
```

备忘录就是给了一颗后悔药，让你能保存当前的状态，在以后能重置回滚。使用的典型场景有：
* 需要保存和回复数据的相关状态场景。
* 提供一个可回滚的操作，比如word中的ctrl + Z 组合，浏览器的回退。
* 需要监控的副本场景。
* 数据库链接的事务管理。

建立备忘录需要注意备忘录的生命周期和生成频率，不要给内存造成很大的负担。

备忘录模式有很多变种。

例如，从上面的代码中可以看到，`Originator`和`Memento`的内部结构实际上差不多，那么我们能否唯一呢？此时备忘录管理类也可以去掉，让`Originator`自主备份。
```java
public class Originator implements Cloneable{
  private Originator backup;
  // 内部状态
  private String state = "";
  public String getState(){
    return state;
  }
  public void setState(String state){
    this.state = state;
  }
  // 创建一个备忘录
  public void createMememto(){
    this.backup = this.clone();
  }
  // 恢复一个备忘录
  public void restoreMemento(){
    // 在此之前应该进行断言，防止空指针
    this.setState(this.backup.getState());
  }
  // 克隆当前对象
  @Override
  protected Originator clone(){
    try{
      return (Originator)super.clone();
    }catch (CloneNotSupportedException e){
      e.printStackTrace();

    }
  }
}
```
注意到，以上使用了“原型模式”，利用`clone`快速复制对象，并且自动备份。但是这似乎不太符合备忘录的模式，因为它的定义是“在该对象之外保存这个状态”，但是以上代码却将其保存在了发起人内部。说的没错，这种Clone方法的备忘录模式，可以使用在比较简单的场景或者比较单一的场景中，尽量不要与其他对的对象产生严重的耦合关系。

在《设计模式之禅》中还介绍了“多状态的备忘录模式”、“多备份的备忘录”、“带有访问权限的备忘录”都非常有启发性。

参考

* [java设计模式之备忘录模式](https://www.cnblogs.com/liaoweipeng/p/5791064.html)
