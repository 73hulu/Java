# Interpreter

解释器模型。定义一个语言，定义它的文法的一种表示。并定义一个解释器，这个解释器使用该表示来解释语言中的句子。

![解释器模型](https://ws1.sinaimg.cn/large/006tNbRwly1fx30imjtroj30q00feq3v.jpg)

有以下角色：
* AbstractExpression（抽象解释器）：具体的解释任务由各个实现类完成，具体的解释器分别为`TerminalExpression`和`NoneterminalExpression`完成。
* TerminalExpression（终结符表达式）：实现与文法中的元素相关联的解释操作，通常一个解释器模式中只有一个终结符表达式。
* NoneterminalExpression（非终结符表达式）：文法中每条规则对应于一个非终结表达式。非终结符表达式根据逻辑的复杂程度而增加，原则上每个文法规则都对应一个非终结符表达式。
* Context（环境角色）：比如HashMap

```java
// 抽象表达式
public abstract class Expression{
  // 每个表达式都必须有一个解析任务
  public abstract Object interpeter(Context ctx);
}
// 终结符表达式，比较简单，主要是处理场景元素和数据的转换
public class TerminalExpression extends Expression{
  // 通常终结符表达式只要一个，但有多个独享
  public Object interpreter(Context ctx){
    return null;
  }
}
// 非终结符表达式，每个都代表了一个文法规则，并且每个文法规则都只关心自己周边的文法规则的结果（注意是结果），因此这就产生了每个非终结符表达式调用自己周边的非终结符表达式。
public class NoneterminalExpression extends Expression{
  // 每个非终结符表达式都会对其他表达式产生依赖
  public NoneterminalExpression(Expression ... expression){

  }
  public Object interpreter(Context ctx){
    // 进行文法处理
    return null;
  }
}
// 客户类
public class Client{
  public static void main(String[] args){
    Context ctx = new Context();
    // 通常定一个语法容器，容纳一个具体的具体的表达式，通常为ArrayList,LinkList等；
    LinkedList<Expression> stack = new LinkedList<Expression>();
    for(;;){
      // 进行语法判断，并产生递归调用
    }
    // 产生一个完成的语法书。由各个具体的语法分析进行解析
    Expression exp = stack.pop();
    // 具体元素进入场景
    exp.interpreter(ctx);
  }
}
```

解析器模式是一个简单的语法分析工具，最显著的特点是扩展性，修改语法规则只需要修改响应的非终结符表达式就可以了，若扩展语法，则只要增加非终结符类就可以了。

缺点在于在规则非常多的情况下会引起类膨胀，并且由于递归的使用比较难以调试及引起性能问题。

较少用到。

参考
* [JAVA 设计模式 解释器模式](http://www.cnblogs.com/jingmoxukong/p/4236961.html)
