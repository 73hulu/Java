# Observable

`Observable`用来实现“观察者模式”中的被观察者的“通知功能”。在使用时候常常与[`Observer`](./Observer.md)来实现此设计模式。

该接口的结构如下：

![java.util.Observable](https://ws2.sinaimg.cn/large/006tNbRwly1fx2x8orixyj30fq0fe74e.jpg)


使用示例：
```Java
// 使用了Observable的被观察者
public class HanFeiZi extends Observable, IHanFeiZi{
  // 韩非子要吃饭了
  public void havaBreakfast(){
    System.out.println("韩非子:开始吃饭了...")
    super.setChange();
    // 继承自Observable的接口方法
    super.notifyObservers("韩非子在吃饭")
  }
  // 韩非子开始娱乐了
  public void haveFun(){
    System.out.println("韩非子：开始娱乐了...");
    super.setChange();
    this.notifyObservers("韩非子在娱乐");
  }
}


// 使用了Observer的观察者
public class LiSi implements Observer{
  // 首先李斯是一个观察者，一旦韩非子有所活动，他就知道，他就要向老板汇报
  public void update(Observable observable, Object obg){
    System.out.println("李斯：观察到李斯的活动，开始像老板汇报....");
    this.reportToQuShiHuang(obj.toString());
    System.out.println("李斯：汇报完毕...\n");
  }
  // 汇报给秦始皇
  private void reportToQuShiHuang(String reportContext){
    System.out.println("李斯：报告，秦老板，韩非子有动静了--->" + reportContext);
  }
}
```
