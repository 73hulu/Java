# java.util.regex

`java.util.regex`是一个用正则表达式所订制的模式来对字符串进行匹配工作的类库包。

它主要包括两个类：`Pattern`和`Matcher` 。

一个`Pattern`是一个正则表达式经编译后的表现模式，一个`Matcher`对象是一个状态机器，它依据Pattern对象做为匹配模式对字符串展开匹配检查。

首先一个Pattern实例订制了一个所用语法与PERL（还没学过……）的类似的正则表达式经编译后的模式，然后一个Matcher实例在这个给定的Pattern实例的模式控制下进行字符串的匹配工作。
