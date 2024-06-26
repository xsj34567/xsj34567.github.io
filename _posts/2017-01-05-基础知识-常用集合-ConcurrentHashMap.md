---
layout: post
title: "常用集合-ConcurrentHashMap"
date: 2017-01-05
categories: 集合 ConcurrentHashMap
tags: Java 集合ConcurrentHashMap
---

{:toc}

## 常用集合

介绍常用集合 Map 的特点、线程安全问题、实现线程安全的方式。

### ConcurrentHashMap

#### 不安全的子类

- HashMap

  > **Note that this implementation is not synchronized.** If multiple threads access a hash map concurrently, and at least one of the threads modifies the map structurally, it _must_ be synchronized externally. (A structural modification is any operation that adds or deletes one or more mappings; merely changing the value associated with a key that an instance already contains is not a structural modification.) This is typically accomplished by synchronizing on some object that naturally encapsulates the map. If no such object exists, the map should be "wrapped" using the [`Collections.synchronizedMap`](https://docs.oracle.com/javase/8/docs/api/java/util/Collections.html#synchronizedMap-java.util.Map-) method. This is best done at creation time, to prevent accidental unsynchronized access to the map:

  ```java
     Map m = Collections.synchronizedMap(new HashMap(...));//方法块加锁
     ConcurrenTHashMap
  ```

```java
/**
 * @author  Doug Lea
 * @author  Josh Bloch
 * @author  Arthur van Hoff
 * @author  Neal Gafter
 * @see     Object#hashCode()
 * @see     Collection
 * @see     Map
 * @see     TreeMap
 * @see     Hashtable
 * @since   1.2
 */
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {

    /**
     * The default initial capacity - MUST be a power of two.
     */
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

    /**
     * The maximum capacity, used if a higher value is implicitly specified
     * by either of the constructors with arguments.
     * MUST be a power of two <= 1<<30.
     */
    static final int MAXIMUM_CAPACITY = 1 << 30;

    /**
     * The load factor used when none specified in constructor.
     */
    static final float DEFAULT_LOAD_FACTOR = 0.75f;

    /**
     * The bin count threshold for using a tree rather than list for a
     * bin.  Bins are converted to trees when adding an element to a
     * bin with at least this many nodes. The value must be greater
     * than 2 and should be at least 8 to mesh with assumptions in
     * tree removal about conversion back to plain bins upon
     * shrinkage.
     */
    static final int TREEIFY_THRESHOLD = 8;

    /**
     * The bin count threshold for untreeifying a (split) bin during a
     * resize operation. Should be less than TREEIFY_THRESHOLD, and at
     * most 6 to mesh with shrinkage detection under removal.
     */
    static final int UNTREEIFY_THRESHOLD = 6;

    /**
     * The smallest table capacity for which bins may be treeified.
     * (Otherwise the table is resized if too many nodes in a bin.)
     * Should be at least 4 * TREEIFY_THRESHOLD to avoid conflicts
     * between resizing and treeification thresholds.
     */
    static final int MIN_TREEIFY_CAPACITY = 64;
	...
}
```

- 技术难点：
  - 基本数据结构（数组+链表（1.7）+[1.8 红黑树](https://baike.baidu.com/item/%E7%BA%A2%E9%BB%91%E6%A0%91/2413209?fr=aladdin)) ---> 什么情况下会链表与红黑树转换？
  - 高效使用的使用 Map（Entry<key,value> 、Lambda 表达式；
  - 理解容量、负载因子、最大容量、转红黑树阈值、取消红黑树阈值，在什么时候会用到；
  - 什么线程不安全？扩容 resize、rehash --> 环形链表的产生（高并发环境下，一个在交换，一个读取原来的值。 A->B->A)
  - 为什么说红黑树提高了查询效率？[红黑树原理及算法实现](https://www.cnblogs.com/skywang12345/p/3245399.html) o(lg(n)) ，链表查询效率 0(n);
  - 红黑树何时会左旋与右旋（添加/移除节点） -> 保证红黑树特性（红黑相间））

参考文档：

[https://docs.oracle.com/javase/8/docs/api/ 官方文档](https://docs.oracle.com/javase/8/docs/api/)

[https://github.com/crossoverJie/JCSprout/blob/master/MD/HashMap.md HashMap 原理](https://github.com/crossoverJie/JCSprout/blob/master/MD/HashMap.md)

[https://blog.csdn.net/hhx0626/article/details/54024222](https://blog.csdn.net/hhx0626/article/details/54024222) 环形链路如何产生

[HashMap连环问答](https://zhuanlan.zhihu.com/p/77899892)
