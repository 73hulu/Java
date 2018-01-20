# ConcurrentHashMap

`ConcurrentHashMap`是实现高并发、高吞吐量的线程安全的`HashMap`，这个类服从与之相同的功能规范`Hashtable`（`Hashtable`是也是线程安全的），并且包括与每个方法相对应的方法的版本`Hashtable`。但是，即使所有操作都是线程安全的，检索操作也不需要锁定，并且也不支持以阻止所有访问的方式锁定整个表。这个类Hashtable在依赖于线程安全性的程序中是完全可互操作的，但是不依赖于它的同步细节。

![ConcurrentHashMap](http://ovn0i3kdg.bkt.clouddn.com/ConcurrentHashMap_1.png?imageView/2/w/300)
![ConcurrentHashMap](http://ovn0i3kdg.bkt.clouddn.com/ConcurrentHashMap_2.png?imageView/2/w/300)
![ConcurrentHashMap](http://ovn0i3kdg.bkt.clouddn.com/ConcurrentHashMap_3.png?imageView/2/w/300)
![ConcurrentHashMap](http://ovn0i3kdg.bkt.clouddn.com/ConcurrentHashMap_4.png?imageView/2/w/300)
![ConcurrentHashMap](http://ovn0i3kdg.bkt.clouddn.com/ConcurrentHashMap_5.png?imageView/2/w/300)
