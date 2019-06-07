---
title: HashMap和Hashtable的区别
tags: ['Java']
category: java
---

HashMap和Hashtable的比较是Java面试中的常见问题，主要用来考研程序员是否能够正确使用集合类以及是否可以随机应变使用多种思路解决问题。

前不久面试时正好也被问到了这个问题，今天突然想起来了，索性就整理一下，写个总结。

HashMap和Hashtable都实现了Map 接口，但是具体要使用哪一个，需要先了解它们存在怎样的区别，然后再根据具体的情况做出选择。

## HashMap和Hashtable的比较

- 1) 线程安全性

　　首先，HashMap是非synchronized的，而Hashtable是synchronized的。这说明Hashtable是线程安全的，而且多个线程可以共享一个Hashtable；而HashMap如果没有正确的同步的话，是不能被多个线程所共享的。但是，Java 5中为我们提供了ConcurrentHashMap，它是Hashtable的替代，而且比Hashtable的扩展性更好。

　　其次，HashMap的迭代器（Iterator）是fail-fast迭代器，而Hashtable的迭代器（enumerator）却不是fail-fast的。因此，当有其它线程改变了HashMap的结构（删除或插入新的元素一个），将会抛出ConcurrentModificationException异常，但是迭代器本身的remove()方法移除元素或者其它线程通过set()方法更改集合对象是允许的（但是，如果已经从结构上进行了修改，在调用set()方法，将会抛出IllegalArgumentException异常），因为这并没有更改集合的“结构”。然而，这并不是一个一定会发生的行为，要看JVM。当然这个行为也是Enumeration和Iterator的区别。

- 2) 同步和速度

　　由于Hashtable是线程安全的，也是synchronized的，所以在单线程环境下比HashMap要慢。如果你不需要同步且只需要单一线程的话，那么使用HashMap性能要比Hashtable好一些。此时，HashMap是个不错的选择。

- 3) 容纳数据

　　HashMap几乎可以等价于Hashtable，除了HashMap是非synchronized的，并可以接受null值（HashMap可以存在null的键值(key)和值(value),但是Hashtable是不可以的)。

- 4) 次序

　　HashMap不能保证随着时间的推移Map中的元素次序是不变的。要保持元素顺序不变，除非是LinkedHashMap。

注：如何实现HashMap的同步呢？

```Java
HashMap hashMap = new HashMap();
Map map = Collections.synchronizeMap(hashMap);
```

## 总结

- HashMap和Hashtable的主要区别在于：线程安全和速度。

- 尽量只在你需要完全的线程安全的时候选择使用Hashtable。

- 如果你使用的是Java 5+的话，尽量使用ConcurrentHashMap。
