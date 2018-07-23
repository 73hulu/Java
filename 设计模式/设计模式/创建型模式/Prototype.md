# Prototype
原型模式： 用原型实例指定创建对象的种类，并且通过拷贝这些原型创建新的对象

![Prototype](http://ovn0i3kdg.bkt.clouddn.com/prototype.png)


## 原型模式
举个例子。批量发送账单：

```java
// 广告信模板类
public class AdvTemplate{
  // 广告信名称
  private String advSubject = "xxx银行国庆信用卡抽奖活动";

  // 广告信内容
  private String advContext = "国庆抽奖活动通知，只要刷卡就送你一百万！....";

  // setter & getter
}
```

```java
// 邮件类
public class Mail{
  // 收件人
  private String receiver;
  // 邮件名称
  private String subject;
  // 称谓
  private String appellation;
  // 邮件内容
  private String context;
  // 邮件的尾部，一般都是加上"xxx版权所有"等信息
  private String tail;
  // 构造函数
  public Mail(AdvTemplate advTemplate){
    this.context = advTemplate.getAdvContext();
    this.subject = advTemplate.getAdvSubject();
  }
  // getter & setter
}
```
场景类：
```java
public class Client {
  // 发送账单的数量，这个值是从数据中获取的
  private static int MAX_COUNT = 6;
  public static void main(String[] args){
    // 模拟发送邮件
    int i = 0;
    // 把模板定义出来，这个是从数据库中取得的
    Mail mail = new Mail(new AdvTemplate());
    mail.setTail("XXX银行所有")；
    while (i < MAX_COUNT){
      // 以下是每封邮件不同的地方
      mail.setAppellation(getRandString(5) + " 先生（女士）");
      mail.setReveiver(getRandString(5) + "@" + getRandString(5) + ".com");
      // 发送邮件
      sendMail(mail);
      i ++;
    }
  }
}
// 省略发送邮件方法sendMail
// 省略获得指定长度的随机字符串方法getRandString
```
这个方法的问题在于：
①如果是单线程。大量邮件需要发送耗费很长时间。
②如果是多线程，会有线程安全问题。即产生第一封邮件对象，放到线程1中运行，还没有发送出去，线程2也启动了，直接把邮件对象mail的收件人改掉了，线程不安全了。

> 其实第二点有点不太明白，为什么线程2会把邮件的收件人改掉？这个线程只包含mail过程么？如果是这样，为什么不能从一开始就将所有的处理（包括定义mail）就包含在不同的线程中呢？

通过对象的赋值功能能解决这个问题。

```java
// 邮件类
public class Mail implements Cloneabel{
  // 收件人
  private String receiver;
  // 邮件名称
  private String subject;
  // 称谓
  private String appellation;
  // 邮件内容
  private String context;
  // 邮件的尾部，一般都是加上"xxx版权所有"等信息
  private String tail;
  // 构造函数
  public Mail(AdvTemplate advTemplate){
    this.context = advTemplate.getAdvContext();
    this.subject = advTemplate.getAdvSubject();
  }
  // getter & setter

  @Override
  public Mail clone(){
    Mail mail = null;
    try{
      mail = (Mail) super.clone();
    }catch (CloneNotSupportException e){
      // todo
      e.printStackTrace();
    }
    return mail;
  }
}
```
然后发送邮件的客户端改成这样：
```java
public class Client {
  // 发送账单的数量，这个值是从数据中获取的
  private static int MAX_COUNT = 6;
  public static void main(String[] args){
    // 模拟发送邮件
    int i = 0;
    // 把模板定义出来，这个是从数据库中取得的
    Mail mail = new Mail(new AdvTemplate());
    mail.setTail("XXX银行所有")；
    while (i < MAX_COUNT){
      // 以下是每封邮件不同的地方
      Mail cloneMail = mail.clone();
      cloneMail.setAppellation(getRandString(5) + " 先生（女士）");
      cloneMail.setReveiver(getRandString(5) + "@" + getRandString(5) + ".com");
      // 发送邮件
      sendMail(mail);
      i ++;
    }
  }
}
```
以上就是原型模式的使用方法：原型实例实现`Cloneabel`接口，然后重写`clone`方法。在时候的是使用调用该方法就能快速复制拷贝出新对象。

在重写`clone`方法的时候，要特别注意深拷贝和浅拷贝。
`super.clone()`是`Object`中的方法，它只能将“值”拷贝下来，这对于一些基本类型（这里包括String类型，它没有clone方法，通过线程池在需要的时候才在内存中创建新字符串）是适用的，但是对于引用类型，这个“值”指的是对象的地址。只是增加了一个指向该对象的指针，这叫做浅拷贝。深拷贝需要自己写。怎么写呢？自己new呗。别入被拷贝的对象中有一个属性是ArrayList，那么在`clone`的时候，除了调用`super.clone()`，还需要自己new一个ArrayList出来。

另外还要注意，如果被拷贝的对象中有final属性，clone就会出错。所以在应用原型模式的时候，不能出现final字段。clone和final是冲突的。

## 原型模型的优点
* 性能优良：原型模式是在内存**二进制流**的拷贝，比直接new一个对象性能要好的多，特别是在**一个循环体内产生大量的对象时，非常适合用原型模型。**
* 逃避构造函数的约束：既是优点也是缺点，直接在内存中拷贝，**构造函数是不会执行的**。




参考
* [[设计模式]原型模式](http://www.cnblogs.com/jingmoxukong/p/4218556.html)
