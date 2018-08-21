# Proxy

> 代理技术是整个Java中最重要的技术，不弄懂它，看一些框架例如Spring的源代码会有问题的。

**动态代理技术，注意不是静态代理，是用来产生一个独对象的代理对象的**。有点拗口，也就是说，如果A是B的代理，我们需要通过B来产生A。类比于B是刘德华，那么A就是刘德华的经纪人。

这就奇怪了，为什么不找刘德华而去找刘德华的经纪人，这个当然了，刘德华的腕儿大么，一些事宜都是经纪人说了算法。那为什么不随便指定一个人呢，这不废话么，刘德华就制定了一个经纪人，随便一个人谁也不认啊。我们需要刘德华唱歌跳舞的，那么就要告诉经纪人：我们邀请经纪人来唱歌跳舞，经纪人需要和刘德华一样，同样会唱歌跳舞（虽然这个例子在现实生活中并不现实，因为经纪人并不需要唱歌跳舞的技能，但是在程序设计中，代理对象和被代理的对象需要有相同的方法）。

所以，由以上的例子就可以明确代理对象的两个概念了：
**1. 代理对象存在的价值在于拦截对真实业务对象（即被代理的对象，上例中的刘德华）**的访问。
**2. 代理对象应该具有和真实业务对象相同的方法。（上例中经济人需要和刘德华一样会唱歌跳舞）**


怎么生成代理对象呢？`Proxy`就是提供产生代理对象的方法类。类结构如下：

![Proxy](https://ws2.sinaimg.cn/large/006tNc79gy1ftr74wjwf6j30hw0mkjv7.jpg)


该类中其他可以不用管，但是`newProxyInstance`这个方法是最最最重要的方法，一定要知道，这个方法用来创建一个对象的代理类，方法签名如下：
```java
public static Object newProxyInstance(ClassLoader loader,
                                          Class<?>[] interfaces,
                                          InvocationHandler h){

                                          }
```

这里有三个参数：
1. `ClassLoader loader`用来指定用哪个类装载器生成代理对象。一般来说，都使用被代理对象的类装载器。
2. `Class<?> [] interface`用来指明生成哪个对象的代理对象。注意，这是通过接口来指定的。
3. `InvocationHandler h` 是用来指明产生的这个代理对象要做什么事情。也就是这个代理对象怎么产生目标对象。因此，一个非常重要的步骤就是实现`InvocationHandler`接口的类。


下面就是使用动态代理模式的例子：

首先定义一个对象的行为接口
```JAVA
package cn.gacl.proxy;
 2
 3 /**
 4 * @ClassName: Person
 5 * @Description: 定义对象的行为
 6 * @author: 孤傲苍狼
 7 * @date: 2014-9-14 下午9:44:22
 8 *
 9 */
10 public interface Person {
11
12     /**
13     * @Method: sing
14     * @Description: 唱歌
15     * @Anthor:孤傲苍狼
16     *
17     * @param name
18     * @return
19     */
20     String sing(String name);
21     /**
22     * @Method: sing
23     * @Description: 跳舞
24     * @Anthor:孤傲苍狼
25     *
26     * @param name
27     * @return
28     */
29     String dance(String name);
30 }
```

接着定义这个接口的目标业务对象类：
```JAVA
package cn.gacl.proxy;
 2
 3 /**
 4 * @ClassName: LiuDeHua
 5 * @Description: 刘德华实现Person接口，那么刘德华会唱歌和跳舞了
 6 * @author: 孤傲苍狼
 7 * @date: 2014-9-14 下午9:22:24
 8 *
 9 */
10 public class LiuDeHua implements Person {
11
12     public String sing(String name){
13         System.out.println("刘德华唱"+name+"歌！！");
14         return "歌唱完了，谢谢大家！";
15     }
16     
17     public String dance(String name){
18         System.out.println("刘德华跳"+name+"舞！！");
19         return "舞跳完了，多谢各位观众！";
20     }
21 }
```

下面就是要看如何生成代理对象的代理类：
```JAVA
1 package cn.gacl.proxy;
2
3 import java.lang.reflect.InvocationHandler;
4 import java.lang.reflect.Method;
5 import java.lang.reflect.Proxy;
6
7 /**
8 * @ClassName: LiuDeHuaProxy
9 * @Description: 这个代理类负责生成刘德华的代理人
10 * @author: 孤傲苍狼
11 * @date: 2014-9-14 下午9:50:02
12 *
13 */
14 public class LiuDeHuaProxy {
15
16     //设计一个类变量记住代理类要代理的目标对象
17     private Person ldh = new LiuDeHua();
18     
19     /**
20     * 设计一个方法生成代理对象
21     * @Method: getProxy
22     * @Description: 这个方法返回刘德华的代理对象：Person person = LiuDeHuaProxy.getProxy();//得到一个代理对象
23     * @Anthor:孤傲苍狼
24     *
25     * @return 某个对象的代理对象
26     */
27     public Person getProxy() {
28         //使用Proxy.newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h)返回某个对象的代理对象
29         return (Person) Proxy.newProxyInstance(LiuDeHuaProxy.class
30                 .getClassLoader(), ldh.getClass().getInterfaces(),
31                 new InvocationHandler() {
32                     /**
33                      * InvocationHandler接口只定义了一个invoke方法，因此对于这样的接口，我们不用单独去定义一个类来实现该接口，
34                      * 而是直接使用一个匿名内部类来实现该接口，new InvocationHandler() {}就是针对InvocationHandler接口的匿名实现类
35                      */
36                     /**
37                      * 在invoke方法编码指定返回的代理对象干的工作
38                      * proxy : 把代理对象自己传递进来
39                      * method：把代理对象当前调用的方法传递进来
40                      * args:把方法参数传递进来
41                      *
42                      * 当调用代理对象的person.sing("冰雨");或者 person.dance("江南style");方法时，
43                      * 实际上执行的都是invoke方法里面的代码，
44                      * 因此我们可以在invoke方法中使用method.getName()就可以知道当前调用的是代理对象的哪个方法
45                      */
46                     @Override
47                     public Object invoke(Object proxy, Method method,
48                             Object[] args) throws Throwable {
49                         //如果调用的是代理对象的sing方法
50                         if (method.getName().equals("sing")) {
51                             System.out.println("我是他的经纪人，要找他唱歌得先给十万块钱！！");
52                             //已经给钱了，经纪人自己不会唱歌，就只能找刘德华去唱歌！
53                             return method.invoke(ldh, args); //代理对象调用真实目标对象的sing方法去处理用户请求
54                         }
55                         //如果调用的是代理对象的dance方法
56                         if (method.getName().equals("dance")) {
57                             System.out.println("我是他的经纪人，要找他跳舞得先给二十万块钱！！");
58                             //已经给钱了，经纪人自己不会唱歌，就只能找刘德华去跳舞！
59                             return method.invoke(ldh, args);//代理对象调用真实目标对象的dance方法去处理用户请求
60                         }
61
62                         return null;
63                     }
64                 });
65     }
66 }
```

在动态代理技术中，**由于不管用户调用了代理对象的什么方法，都是调用开发人员编写的处理器的invoke方法，相当于invoke方法拦截到了代理对象的方法调用。**，并且，开发人员能通过invoke方法的参数，还可以在拦截的同事，知道用户调用了什么方法，因此利用这两个属性，就可以实现一些特殊需求，比如：拦截用户的访问请求，查看权限，动态添加额外功能。

参考
* [java中Proxy(代理与动态代理)](https://blog.csdn.net/pangqiandou/article/details/52964066)
