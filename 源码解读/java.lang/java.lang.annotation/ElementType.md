# ElementType

这个枚举类型的常量提供了Java程序中注解可能出现的句法位置的简单分类。这些常量用在`java.lang.annotation.Target`元注释中以指定写入给定类型的注释的合法位置。
注释可能出现的语法位置被分解成声明上下文，注释应用于声明，类型上下文，注释应用于声明和表达式中使用的类型。

常量`ANNOTATION_TYPE`，`CONSTRUCTOR`，`FIELD`，`LOCAL_VARIABLE`，`METHOD`，`PACKAGE`，`PARAMETER`，`TYPE`和`TYPE_PARAMETER`对应于JLS 9.6.4.1中的声明上下文。

有以下常量：


| 常量 | 含义 |
| :------------- | :------------- |
|TYPE |  类、接口（包括注释类型）或枚举声明    |
|FIELD   |  字段声明（包括枚举常量） |
|METHOD   |  方法声明  |
|PARAMETER   | 常规参数声明  |
|CONSTRUCTOR   |  构造方法声明  |
|LOCAL_VARIABLE   | 局部变量声明   |
|ANNOTATION_TYPE   | 注释类型声明  |
|PACKAGE   |  包声明 |
|TYPE_PARAMETER   | Type parameter declaration（since 1.8）  |
|TYPE_USE   | 类型声明（包括注释类型声明）和参数类型声明|




例如，类型被@Target（ElementType.TYPE_USE）元注释的注释可以被写在字段的类型上（或者在字段的类型内，如果它是嵌套的，参数化的或数组类型的），并且也可以作为类声明的修饰符出现。

TYPE_USE常量包括类型声明和类型参数声明，以方便设计者为赋予注释类型语义的类型检查器。例如，如果注释类型NonNull使用@Target（ElementType.TYPE_USE）进行元注释，则类型检查器可以将@NonNull类C {...}处理为指示类C的所有变量都为非null ，同时仍然允许其他类的变量是否为非null或非非null，这取决于@NonNull是否出现在变量的声明中。
