# Base

这是最基础的包，主要包含一些工具类，使得Java语言使用更加便利。按照官方文档，可以大致分为以下三类：

![google guava base](https://ws3.sinaimg.cn/large/006tNc79ly1fvt5klra7tj30cg0scgm4.jpg)


`Ascii`中针对Ox00-Ox7F的字符和字符串提供了一些静态处理方法，从源码中学习到的最关键的方法在于大小写字母之间的一些规律，该类提出了`CASE_MASK = Ox20`，例如大小写的转化就是其中与`CASE_MASK`的异或结果，取得字母c在字母表中的索引的方法是`(char)()(c | CASE_MASK) - 'a')` 。
