# Flyweight

Flyweight模式翻译过来是“享元模式”，名字有点怪，但是我们实际上一直在用这种模式，只是不知道这叫做享元模式。

> Flyweight是拳击比赛中的特有名词，意思是“较轻量级”。

![享元模式](https://ws4.sinaimg.cn/large/006tNbRwly1fx26dn4xnlj30k50bbq3u.jpg)


享元技模式是池技术的重要实现方式，其定义如下：使用**共享对象**可有效得支持大量**细粒度的对象**。

享元模式的主要目的是实现对象的共享，即共享池，当系统中对象多的时候可以减少内存的开销，通常与工厂模式一起使用。


> 对象池（Object Pool）的实现有很多开源工具，比如Apache的common-pool就是一个不错的池工具。

```JAVA
public class ConnectionPool {  

    private Vector<Connection> pool;  

    /*公有属性*/  
    private String url = "jdbc:mysql://localhost:3306/test";  
    private String username = "root";  
    private String password = "root";  
    private String driverClassName = "com.mysql.jdbc.Driver";  

    private int poolSize = 100;  
    private static ConnectionPool instance = null;  
    Connection conn = null;  

    /*构造方法，做一些初始化工作*/  
    private ConnectionPool() {  
        pool = new Vector<Connection>(poolSize);  

        for (int i = 0; i < poolSize; i++) {  
            try {  
                Class.forName(driverClassName);  
                conn = DriverManager.getConnection(url, username, password);  
                pool.add(conn);  
            } catch (ClassNotFoundException e) {  
                e.printStackTrace();  
            } catch (SQLException e) {  
                e.printStackTrace();  
            }  
        }  
    }  

    /* 返回连接到连接池 */  
    public synchronized void release(conn) {  
        pool.add(conn);  
    }  

    /* 返回连接池中的一个数据库连接 */  
    public synchronized Connection getConnection() {  
        if (pool.size() > 0) {  
            Connection conn = pool.get(0);  
            pool.remove(conn);  
            return conn;  
        } else {  
            return null;  
        }  
    }  
}  
```

Java API中的享元模式应用，比如`String.intern()`方法。
