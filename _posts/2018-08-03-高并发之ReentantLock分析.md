---
layout: post
title:  "高并发之ReentrantLock分析"
date:   2018-08-03
categories: 高并发 Lock
tags: 高并发
---

* content
{:toc}
### 1.1 AQS

​	  AbstractQueuedSynchronizer： 互斥、共享

![2021-11-27_Locks类型](\image\并发\JUC\2021-11-27_Locks类型.png)

### 1.2 ReentrantLock设计思路

> 满足线程的互斥特性；意味着同一个时刻，只允许一个线程进入到加锁的代码中。-> 多线程环境下，线程的顺序访问。

- 1. 一定会涉及到锁的抢占：

  ```java
  //需要有一个标记来实现互斥。全局变量（0,1)   ==> CAS
  ```

- 2. 抢占到了锁，怎么处理   

     ```java
     //不需要处理,直接执行。
     ```

  3. 没抢占到锁，怎么处理

     > 需要等待

     ```java
     //让处于排队中的线程，如果没有抢占到锁，则直接先阻塞->释放CPU资源）。
     ```

     - 如何等待？
       - wait/notify(线程通信的机制，无法指定唤醒某个线程)
       - <font color='red'>LockSupport.park/unpark（阻塞一个指定的线程，唤醒一个指定的线程）</font>
       - Condition (await/notify/notifyAll)

     > 需要排队

     ```java
     //允许有N个线程被阻塞，此时线程处于活跃状态
     ```

     - 通过一个数据结构，把这N个排队的线程存储起来。

       ```java
       //阻塞队列(双向链表)
       ```

  4. 抢占到锁，如何释放？

     ```java
     // LockSupport.unpark(Thread)  -> 唤醒处于队列中指定的线程。
     ```

  5. 锁抢占的公平性（是否允许插队）

     1. 公平锁
     2. 非公平锁

  6. 重入的特性

     ```java
     //	识别是否是同一个ThreadID
     ```

     

### 1.3 Lock 与 Condition

![2021-11-29_Lock与Condition图解](\image\并发\JUC\2021-11-29_Lock与Condition图解.png)



#### 参考：

[使用Atomic](https://www.liaoxuefeng.com/wiki/1252599548343744/1306581083881506)
