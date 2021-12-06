---
layout: post
title:  "高并发之Synchronized"
date:   2018-08-02 
categories: 高并发 安全性 Synchronized
tags: 高并发
---

* content
{:toc}
### 线程的安全性问题

​       大家都知道,线程会存在安全性问题,那接下来我们从原理层面去了解线程为什么会存在安全性问题,并且我们应该怎么去解决这类的问题。
​      其实线程安全问题可以总结为:` 原子性`、`可见性`、`有序性`这几个问题,我们搞懂了这几个问题并且知道怎么解决,那么多线程安全性问题也就不是问题了。

常见的解决安全性问题方案：

```text
原子性： Synchronized、AtomicXXX、Lock (CAS)
可见性： Synchroinzed、volatile
有序性： Synchroinzed、volatile  => happens-before模型
```

#### 1. Synchronized

​	Synchronized减重的过程，通常被称为锁膨胀或是锁升级的过程。
主要步骤是：

- 先是通过偏向锁来获取锁，解决了虽然有同步但无竞争的场景下锁的消耗。
- 再是通过对象头的Mark Word来实现的轻量级锁，通过轻量级锁如果还有竞争，那么继续升级。
- 升级为自旋锁，如果达到最大自旋次数了，那么就直接升级为重量级锁，所有未获取锁的线程都阻塞等待。



#### 参考：

[到底如何保证线程安全，你真的清楚吗？](https://mp.weixin.qq.com/s/VXzDrkR0ix5S9PNBs3SKhA)

