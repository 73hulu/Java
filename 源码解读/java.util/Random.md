# Random

学到`Math`的`random`方法时候，想起来`Random`也可以实现随机数。但是`Random`类中实现的随机算法是伪随机，也就是有规则的随机。在进行随机时，随机算法的起源数字称为种子数(seed)，在种子数的基础上进行一定的变换，从而产生需要的随机数字。

**相同种子数的Random对象，相同次数生成的随机数字是完全相同的。**也就是说，两个种子数相同的Random对象，第一次生成的随机数字完全相同，第二次生成的随机数字也完全相同。这点在生成多个随机数字时需要特别注意。

![Random](http://ovn0i3kdg.bkt.clouddn.com/Random_1.png)
![Random](http://ovn0i3kdg.bkt.clouddn.com/Random_2.png)

以上是`Random`类的结构，方法很多，但是一样是个工具类，了解常用的方法就行。

### 构造方法
重载了2个构造方法。
#### public Random(){...}
无参数构造方法，使用一个和当前系统时间对应的相对时间有关的数字作为种子数，然后使用这个种子数构造Random对象。
```java
public Random() {
    this(seedUniquifier() ^ System.nanoTime());
}
```

#### public Random(long seed){...}
通过制定一个种子数进行创建。具体的实现细节不重要省略不讲。使用示例：
```java
Random r1 = new Random(10);
```
再次强调：种子数只是随机算法的起源数字，**和生成的随机数字的区间无关**。


### 常用方法
`Random`作为工具类，只需要知道一些常用的方法就行。总结如下：

| 方法 | 含义 |
| :------------- | :------------- |
| public boolean nextBoolean() | 生成一个随机的boolean值，生成true和false的值几率相等，也就是都是50%的几率。    |
|public double nextDouble()   |  生成一个随机的double值，数值介于[0,1.0)之间。与`Math.random()`方法实现效果一样|
|public int nextInt()   |生成一个随机的int值，该值介于int的区间，也就是-2^31到2^31-1之间。如果需要制定区间，需要做数学变换  |
|public int nextInt(int n)   |  生成一个随机的int值，该值介于[0,n)的区间。如果需要制定区间，需要做数学变换 |
|  public void setSeed(long seed) | 重新设置Random对象中的种子数。设置完种子数以后的Random对象和相同种子数使用new关键字创建出的Random对象相同。【所以程序里无论要生成多少个随机数，使用同一个`Random`对象即可】  |

以下是`Random`类的一些使用示例：
```java
Random r = new Random();
```
1. 生成[0,1.0)区间的小数
```java
double d = r.nextDouble();
```
2. 生成[0,5.0)区间的小数
```java
double d = r.nextDouble() * 5;
```
3. 生成[1,2.5)区间的小数
```java
double d = r.nextDouble() * 1.5 + 1;
```
4. 生成任意整数
```java
int i = r.nextInt();
```
5. 生成[0,10)区间的整数
```java
int i1 = r.nextInt(10);
//或者
int i2 = Math.abs(r.nextInt() % 10);
```
6. 生成[0,10]区间的整数
```java
int i1 = r.nextInt(11);
//或者
int i2 = Math.abs(r.nextInt() % 11);
```
7. 生成[-3,15)区间的整数
```java
int i1 = r.nextInt(18) - 3;
//或者
int i2 = Math.abs(r.nextInt() % 18) - 3;
```
8. 几率实现问题

  `nextInt(int n)`方法中生成的数字是均匀的，也就是说该区间内部的每个数字生成的几率是相同的。那么如果生成一个[0,100)区间的随机整数，则每个数字生成的几率应该是相同的，而且由于该区间中总计有100个整数，所以每个数字的几率都是1%。按照这个理论，可以实现程序中的几率问题。

  例如：代码实现”随机生成一个整数，该整数以55%的几率生成1，以40%的几率生成2，以5%的几率生成3。"

  代码实现为：
  ```java
  int n5 = r.nextInt(100);
  int m; //结果数字
  if(n5 < 55){ //55个数字的区间，55%的几率
    m = 1;
  }else if(n5 < 95){//[55,95)，40个数字的区间，40%的几率
    m = 2;
  }else{
    m = 3;
  }
  ```

  因为每个数字的几率都是1%，则任意55个数字的区间的几率就是55%，为了代码方便书写，这里使用[0,55)区间的所有整数，后续的原理一样。

  当然，这里的代码可以简化，因为几率都是5%的倍数，所以只要以5%为基础来控制几率即可，下面是简化的代码实现：
  ```java
  int n6 = r.nextInt(20);
  int m1;
  if(n6 < 11){
      m1 = 1;
  }else if(n6 < 19){
      m1= 2;
  }else{
      m1 = 3;
  }
  ```
  > 真是机智，打开新思路！

参考:
* [Random类 (java.util)](http://www.cnblogs.com/Fskjb/archive/2009/08/29/1556417.html)
