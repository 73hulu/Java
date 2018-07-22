# Pattern

`Pattern`是一个正则表达式经过编译后的表现模式。该类的内容有很多，暂时不纠细节，了解常用的方法即可。

![Pattern](https://ws4.sinaimg.cn/large/006tKfTcgy1ftae4tsfioj30ap05dq3b.jpg)


>Pattern是一个终类，并且构造方法是私有的。

## compile
这个方法将给定的正则表达式编译并赋予给Pattern类。可以指定正则表达式和flag。

```Java
public static Pattern compile(String regex) {
    return new Pattern(regex, 0);
}

public static Pattern compile(String regex, int flags) {
    return new Pattern(regex, flags);
}

private Pattern(String p, int f) {
    pattern = p;
    flags = f;

    // to use UNICODE_CASE if UNICODE_CHARACTER_CLASS present
    if ((flags & UNICODE_CHARACTER_CLASS) != 0)
        flags |= UNICODE_CASE;

    // Reset group index count
    capturingGroupCount = 1;
    localCount = 0;

    if (pattern.length() > 0) {
        compile();
    } else {
        root = new Start(lastAccept);
        matchRoot = lastAccept;
    }
}
```
flag具体是什么含义还没有弄明白，这个参数的取值可以是CASE INSENSITIVE,MULTILINE,DOTALL,UNICODE CASE， CANON EQ，但是一般不常用。


## public Matcher matcher(CharSequence input) {...}
该方法用来生成一个给定命名的Matcher对象。
```Java
public Matcher matcher(CharSequence input) {
    if (!compiled) {
        synchronized(this) {
            if (!compiled)
                compile();
        }
    }
    Matcher m = new Matcher(this, input);
    return m;
}
```

所以Pattern的使用套路经常是：
```Java
String str = "Hello World";
Pattern pattern = Pattern.compile("^\S$");
Matcher matcher = pattern.matcher(str);
```
然后接下来真正匹配的工作就交给Matcher类了。


## public static boolean matches(String regex, CharSequence input) {...}
编译给定的正则表达式并且对输入的字串以该正则表达式为模开展匹配,该方法适合于该正则表达式只会使用一次的情况，也就是只进行一次匹配工作，因为这种情况下并不需要生成一个Matcher实例。
```Java
public static boolean matches(String regex, CharSequence input) {
    Pattern p = Pattern.compile(regex);
    Matcher m = p.matcher(input);
    return m.matches();
}
```
这个方法的效果等同于：
```Java
boolean res = Pattern.compile(regex).matcher(input).matches();
```

## split方法
这个方法将目标字符串按照Pattern里所包含的正则表达式为模进行分割。效果类似于`String`的`split`方法：
```Java
public String[] split(CharSequence input, int limit) {...}
public String[] split(CharSequence input) {...}
```
其中`limit`参数的含义与`String.split`中的参数一致，下面是一个例子：
```Java
import java.util.regex.*;

public class Replacement {
    public static void main(String[] args) throws Exception {
        // 生成一个Pattern,同时编译一个正则表达式
        Pattern p = Pattern.compile("[/]+");
        // 用Pattern的split()方法把字符串按"/"分割
        String[] result = p.split("Kevin has seen《LEON》seveal times,because it is a good film."
               + "/ 凯文已经看过《这个杀手不太冷》几次了，因为它是一部" + "好电影。/名词:凯文。");
       for (int i = 0; i < result.length; i++)
            System.out.println(result[i]);
       }
}
```
程序的输出结果为：
```Java
Kevin has seen《LEON》seveal times,because it is a good film.
凯文已经看过《这个杀手不太冷》几次了，因为它是一部好电影。
名词:凯文。
```
