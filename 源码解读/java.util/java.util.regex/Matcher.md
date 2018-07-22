# Matcher

Matcher类是正则表达式的匹配器类。定义如下：

![Matcher](https://ws3.sinaimg.cn/large/006tKfTcgy1ftaeovw0dsj30bh0rvjtu.jpg)
![Matcher](https://ws2.sinaimg.cn/large/006tKfTcgy1ftaepdd0ioj30bh04agls.jpg)

## public final class Matcher implements MatchResult
类声明，被final修饰，说明不可被继承。

## 构造方法
Matcher类的两个构造方法都是protected的，在`Pattern`中被调用，我们需要通过`Pattern`的静态方法`compile`来创建`Matcher`实例：
```Java
Matcher() {
}

Matcher(Pattern parent, CharSequence text) {
    this.parentPattern = parent;
    this.text = text;

    // Allocate state storage
    int parentGroupCount = Math.max(parent.capturingGroupCount, 10);
    groups = new int[parentGroupCount * 2];
    locals = new int[parent.localCount];

    // Put fields into initial states
    reset();
}
```
这里有几个变量的含义需要说明一下：
```Java
// 创建此对象的模式匹配。
Pattern parentPattern;

// 匹配的目的字符串。
CharSequence text;

// 组使用的存储。如果在匹配过程中跳过一个组，它们可能包含无效的值。存储的是当前匹配的各捕获组的first和last信息。
// groups[0]存储的是组零的first，groups[1]存储的是组零的last，groups[2]存储的是组1的first，groups[3]存储的是组1的last
int[] groups;

int[] locals;
```
最后，调用`reset`进行复位，将类的一些变量进行赋值，最终各个变量的结果如下：
![Matcher初始化结果](https://ws1.sinaimg.cn/large/006tKfTcgy1ftaezr5wu7j314w0lgdil.jpg)

## public Pattern pattern() {...}
返回parentPattern，即构造器传入的Pattern对象。
```Java
public Pattern pattern() {
    return parentPattern;
}
```

## public int groupCount(){...}
返回匹配器中的捕获组个数。零组表示不在捕获组中，不计入个数
```Java
public int groupCount() {
    return parentPattern.capturingGroupCount - 1;
}
```
示例：
```Java
Pattern p = Pattern.compile("(\\w+)%(\\d+)");
Matcher m = p.matcher("ab%12-cd%34");
System.out.println(m.groupCount());// 2
```

## start 和 end
重载了三种start方法和end方法，含义一一对应，这里只解释start方法。
```Java
// 返回当前匹配的子串的第一个字符在目标字符串中的索引位置 。
public int start() {
    if (first < 0)
        throw new IllegalStateException("No match available");
    return first; // 可知start()方法返回的是匹配器的状态first
}

// 返回当前匹配的指定组中的子串的第一个字符在目标字符串中的索引位置 。
public int start(int group) {
    if (first < 0)
        throw new IllegalStateException("No match available");
    if (group < 0 || group > groupCount())
        throw new IndexOutOfBoundsException("No group " + group);
    return groups[group * 2];
}

// 返回当前指定的命名捕获组中的子串的一个字符在目标字符串中的索引位置
public int start(String name) {
    return groups[getMatchedGroupIndex(name) * 2];
}
```

## public boolean find() {...}
在目标字符串里查找下一个匹配子串。如果匹配成功，则可以通过 start、end 和 group 方法获取更多信息。

```Java
public boolean find() {
   int nextSearchIndex = last;
   if (nextSearchIndex == first)
       nextSearchIndex++;

   // If next search starts before region, start it at region
   if (nextSearchIndex < from)
       nextSearchIndex = from;

   // If next search starts beyond region then it fails
   if (nextSearchIndex > to) {
       for (int i = 0; i < groups.length; i++)
           groups[i] = -1;
       return false;
   }
   return search(nextSearchIndex);
}
```
使用示例：
```Java
Pattern p = Pattern.compile("(\\w+)%(\\d+)");
Matcher m = p.matcher("ab%12-cd%34");
while (m.find()) {
    System.out.println("group():" + m.group());
    System.out.println("start():" + m.start());
    System.out.println("end():" + m.end());
    System.out.println("group(1):" + m.group(1));
    System.out.println("start(1):" + m.start(1));
    System.out.println("end(1):" + m.end(1));
    System.out.println("group(2):" + m.group(2));
    System.out.println("start(2):" + m.start(2));
    System.out.println("end(2):" + m.end(2));
    System.out.println();
}
```
执行结果为：
```Java
group():ab%12
start():0
end():5
group(1):ab
start(1):0
end(1):2
group(2):12
start(2):3
end(2):5

group():cd%34
start():6
end():11
group(1):cd
start(1):6
end(1):8
group(2):34
start(2):9
end(2):11
```
find()方法匹配了两个子串：ab%12和cd%34；每个子串有2组。

## group
返回当前查找而获得的与组匹配的所有子串内容。
```Java
// 可知group()实际调用了group(int group)方法，参数group为0。组零表示整个模式
public String group() {
    return group(0);
}
// 返回当前查找而获得的与组匹配的所有子串内容
public String group(int group) {
    if (first < 0)
        throw new IllegalStateException("No match found");
    if (group < 0 || group > groupCount())
        throw new IndexOutOfBoundsException("No group " + group);
    if ((groups[group*2] == -1) || (groups[group*2+1] == -1))
        return null;
    return getSubSequence(groups[group * 2], groups[group * 2 + 1]).toString();
}
// 返回当前指定名称的捕获组而获得的与组匹配的所有子串内容
public String group(String name) {
    int group = getMatchedGroupIndex(name);
    if ((groups[group*2] == -1) || (groups[group*2+1] == -1))
        return null;
    return getSubSequence(groups[group * 2], groups[group * 2 + 1]).toString();
}
```




参考
* [java之Matcher类详解](https://www.cnblogs.com/SQP51312/p/6134324.html)
