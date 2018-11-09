# Factory Method

学习抽象工厂的时候知道，抽象工厂的缺点是一旦有新的需求，工厂需要是生产新的产品，那么我们就需要修改抽象工厂的接口，这就意味着，实现这个接口的所有实现工程都需要修改，这不符合“开闭原则”。

## 工厂方法的概念
工程方法克服了这个缺点，它将类的实例化（具体产品的创建）延迟到了工厂类的子类（具体工厂）中完成，即由具体的工厂来决定应该创建哪一个产品。


![Factory Method](http://img.blog.csdn.net/20160828082911344)


假设这样的情况：工厂A生产产品a，按照抽象工厂的方法，它应该写成这样：
```java
// 抽象工厂接口
interface Factory{
  Product productA();
}
// 抽象产品接口
interface Product{
  void printInfo();
}

//具体工厂
public Class FactoryA implements Factory{
  @Override
  public Product productA{
    return new ProductA();
  }
}
```
那么如果现在新的需求来了，工厂还需要生产产品B，那要怎么办？如果抽象方法，那么只能在接口Factory中增加创建B产品的方法，造成的后果是工厂的实现类也跟着需要改变：
```java
// 抽象工厂接口
interface Factory{
  Product productA();
  Product productB();
}
// 抽象产品接口
interface Product{
  void printInfo();
}

public class ProductA implements Product{

  @Override
  public void printInfo(){
    System.out.print("a");
  }
}

public class productB implements Product{
  @Override
  public void printInfo(){
    System.out.print("b");
  }
}
//具体工厂
public Class FactoryA implements Factory{
  @Override
  public Product productA{
    return new ProductA();
  }

  @Override
  public Product productB{
    return new productB();
  }
}

public class AbstactFactoryTest(){
  public static void main(String[] args){
    Factory factory = new FactoryA();
    ProductA productA = factory.productA();
    productA.printInfo();
    ProductB productB = factory.productB();
    productB.printInfo();
  }
}
```

总是，抽象工厂方法是创建了一个接口，艺高人胆大，定义了工厂能具有的所有功能。能造什么不能造什么全部都约定清楚了。


但是工厂方法不一样，它很宽容，只是限定了工厂它能造东西，至于造什么东西，你们自便。仍然是上面这个例子，如果按照工厂方法的模式来写，应该是这样：
```java
//抽象接口
interfact Factory{
  Product product();
}

// 抽象产品接口
interface Product{
  void printInfo();
}

// 具体产品ProductA类
public class ProductA implements Product{
  @Override
  public void printInfo(){
    System.out.print("a");
  }
}

//能创建A产品的工厂
public class FactoryA implements Factory{
  @Override
  public Product product(){
    return new ProductA();
  }
}
```
以上就是一个能造A产品的工厂A，如果现在来了新需求，需要造B怎么办？不能改变A，那么再创建一个工厂B
```java
// 抽象工厂接口
interface Factory{
  Product productA();
  Product productB();
}
// 抽象产品接口
interface Product{
  void printInfo();
}

public class ProductA implements Product{

  @Override
  public void printInfo(){
    System.out.print("a");
  }
}

public class productB implements Product{
  @Override
  public void printInfo(){
    System.out.print("b");
  }
}

//能创建A产品的工厂
public class FactoryA implements Factory{
  @Override
  public Product product(){
    return new ProductA();
  }
}

// 能创建B产品的工厂
public class FactoryB implements Factory{
  @Override
  public Product product(){
    return new ProductB;
  }
}

// 测试方法
public class FactoryMethodTest{
  public static void main(String[] args){
    //获取客户需要的A产品
    Factory factoryA = new FactoryA();
    Product productA = factoryA.product();
    productA.printInfo(); //a

    //获取客户需要的B产品
    Factory factoryB = new FactoryB();
    Product productB = factoryB.product();
    productB.printInfo(); //b
  }
}
```

> 以上这种方法叫做多工厂方法类，即每个产品单独对应了一个工厂实例。在复杂的应用中一般采用多工厂的方法，然后再增加一个协调类，避免调用者与各个子工厂交流。协调类的目的是封装子工厂类，对高层模块提供统一的访问接口。

所以，面对不同的需求，以后只需要新增不同的工厂就可以了，可就是说可能需要工厂有n种功能，那么我们就需要创建n个工厂。这样每个工厂就各司其职，符合单一原则，有改动时候不需要改动原来的类，符合开闭原则。能够拥抱变化

当然这些有点换个角度想想也是缺点。

* 添加新产品时，除了增加新产品类外，还要提供与之对应的具体工厂类，系统类的个数将成对增加，在一定程度上增加了系统的复杂度；同时，有更多的类需要编译和运行，会给系统带来一些额外的开销。
* 由于考虑到系统的可扩展性，需要引入抽象层，在客户端代码中均使用抽象层进行定义，增加了系统的抽象性和理解难度，且在实现时可能需要用到DOM、反射等技术，增加了系统的实现难度。
* 虽然保证了工厂方法内的对修改关闭，但对于使用工厂方法的类，如果要更换另外一种产品，仍然需要修改实例化的具体工厂类；
* 一个具体工厂只能创建一种具体产品

## 工厂方法实践
### 代替单例模式
通过犯罪社可以创建单例。
```Java
publci class SingletonFactory{
  private static Singleton singleton;

  static {
    try{
      Class cl = Class.forName(Singleton.class.getName());
      Constructor concurrent = cl.getDeclaredConstructor();
      constructor.setAccessible(true);
      singleton =(Singleton)constructor.newInstance();
    }catch(Exception e){
      // 异常处理
    }
  }

  public static Singleton getSingleton(){
    return singleton;
  }
}
```

### 延迟初始化
什么是延迟初始化？一个对象被消费完毕之后，并不立即释放，工厂类保持起初始状态，等待再次被使用。
```Java
public class ProductFactory{
  private static final Map<String, Product> prMap = new HashMap();

  public static synchroniezed Product createProduct(String type) throws Exception {
    Product product = null;
    if (prMap.containsKey(type)) {
      product = prMap.get(type);
    }else{
      if(type.equals("Product1")){
        product = new ConcreteProduct1();
      }else {
        product = new ConcreteProduct2();
      }
      // 同时放到缓存中
      prMap.put(type, product);
    }
    return product;
  }
}
```
代码非常简单，利用map来做缓存，注意这个使用synchronized保证线程安全，一个性能改进的办法是使用`ConcurrentHashMap`的`putIfAbsent`方法。

延迟框架是可以扩展的，例如限制某一个产品类的最大实例化话数量，可以通过判断Map中已有的对象数量来实现。例如JDBC连接数据库，都会要设置一个MaxConnections最大连接树龄，该数量就是内存中最大实例化的数量。



参考
* [[设计模式]工厂方法模式](http://www.cnblogs.com/jingmoxukong/p/4016173.html)
* [工厂方法模式（Factory Method）-最易懂的设计模式解析](http://blog.csdn.net/carson_ho/article/details/52343584)
