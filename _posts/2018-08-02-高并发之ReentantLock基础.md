---
layout: post
title:  "高并发之ReentrantLock基础"
date:   2018-08-02 
categories: 高并发 Lock
tags: 高并发 Lock
---

* content
{:toc}
### 1. ReentrantLock

​	锁的作用为了<font color='green'>保证线程间同步是安全的</font>。java.util.concurrent包提供了Lock操作，保证了原子性。其主要利用了<font color='red'>AQS队列、线程间切换（LockSupport.park/unpark）、CAS(锁状态）</font>。

#### 1.1 为什么要引入ReentrantLock？

​		`synchronized`关键字用于加锁，但这种锁<font color='red'>一是很重，二是获取时必须一直等待，没有额外的尝试机制。基于JVM层面的锁</font>，不需要考虑异常。

```java
public class Counter {
    private int count;
    public void add(int n) {
        synchronized(this) {
            count += n;
        }
    }
}
```

​        `ReentrantLock` ：基于<font color='green'>Java代码层面实现的锁，我们必须要获取锁，然后在finally中释放锁。</font>与synchronized不同的是，reentrantLock可以先尝试获取锁：

```java
//尝试获取锁的时候，最多等待1秒。如果1秒后仍未获取到锁，tryLock()返回false，程序就可以做一些额外处理，而不是无限等待下去
if (lock.tryLock(1, TimeUnit.SECONDS)) {
    try {
        ...
    } finally {
        lock.unlock();
    }
}
```

```java
public class Counter {
    private final Lock lock = new ReentrantLock();
    private int count;
    public void add(int n) {
        lock.lock();
        try {
            count += n;
        } finally {
            lock.unlock();
        }
    }
}
```

使用`ReentrantLock`比直接使用`synchronized`更安全，线程在`tryLock()`失败的时候不会导致死锁。**[来源](https://www.liaoxuefeng.com/wiki/1252599548343744/1306580960149538)**

#### 1.2 `ReentrantLock`我们怎么编写`wait`和`notify`的功能呢？

​		使用`ReentrantLock`比直接使用`synchronized`更安全，可以替代`synchronized`进行线程同步。但是，`synchronized`可以配合`wait`和`notify`实现线程在条件不满足时等待，条件满足时唤醒，用`ReentrantLock`我们怎么编写`wait`和`notify`的功能呢？

答案是<font color='red'>使用`Condition`对象来实现`wait`和`notify`的功能。</font>

##### 1.2.1  基于synchronized的阻塞队列

```java
class TaskQueue {
    Queue<String> queue = new LinkedList<>();

    public synchronized void addTask(String s) {
        this.queue.add(s);
        this.notifyAll();//唤醒全部线程，如果使用notify可能唤醒了其他线程
    }

    public synchronized String getTask() {
        while (queue.isEmpty()) {
            //等待/阻塞,避免死循环，CPU飙高。释放this锁
            this.wait();
            //重新获取this锁
        }
        return queue.remove();
    }
}
```

##### 1.2.2 基于ReentrantLock实现阻塞队列

```java
class TaskQueue {
    private final Lock lock = new ReentrantLock();
    private final Condition condition = lock.newCondition(); //必须基于该Lock对象中创建Condition
    private Queue<String> queue = new LinkedList<>();

    public void addTask(String s) {
        lock.lock();
        try {
            queue.add(s);
            condition.signalAll();
        } finally {
            lock.unlock();
        }
    }
    public String getTask() {
        lock.lock();
        try {
            while (queue.isEmpty()) {
                condition.await();
            }
            return queue.remove();
        } finally {
            lock.unlock();
        }
    }
}
```

分析：

| 名称       | Condition   | Synchronized | 解释                                                         |
| ---------- | ----------- | ------------ | ------------------------------------------------------------ |
| 阻塞锁     | await()     | wait()       | 会释放当前锁，线程进入等待状态<br/>和`tryLock()`类似，`await()`可以在等待指定时间后，<br/>如果还没有被其他线程通过<br/>`signal()`或`signalAll()`唤醒，可以自己醒来 |
| 唤醒锁     | signal()    | notify()     | 会唤醒某个等待线程                                           |
| 唤醒全部锁 | signalAll() | notifyAll()  | 会唤醒所有等待线程                                           |

```java
if (condition.await(1, TimeUnit.SECOND)) {
    // 被其他线程唤醒
} else {
    // 指定时间内没有被其他线程唤醒
}
```

#### 1.3 ReentrantReadWriteLock

> ReentrantLock只保证一个线程可以执行临界代码。保护过头了，并发性能不高，不能同时读。

```java
public class Counter {
    private final Lock lock = new ReentrantLock();
    private int[] counts = new int[10];

    public void inc(int index) {
        lock.lock();
        try {
            counts[index] += 1;
        } finally {
            lock.unlock();
        }
    }

    public int[] get() {
        lock.lock();
        try {
            return Arrays.copyOf(counts, counts.length);
        } finally {
            lock.unlock();
        }
    }
}
```

<font color='red'>实际上我们想要的是：允许多个线程同时读，但只要有一个线程在写，其他线程就必须等待：</font>

使用`ReadWriteLock`可以解决这个问题，它保证：

- 只允许一个线程写入（其他线程既不能写入也不能读取）；
- 没有写入时，多个线程允许同时读（提高性能）。

```java
public class Counter {
    private final ReadWriteLock rwlock = new ReentrantReadWriteLock();
    private final Lock rlock = rwlock.readLock();
    private final Lock wlock = rwlock.writeLock();
    private int[] counts = new int[10];

    public void inc(int index) {
        wlock.lock(); // 加写锁
        try {
            counts[index] += 1;
        } finally {
            wlock.unlock(); // 释放写锁
        }
    }

    public int[] get() {
        rlock.lock(); // 加读锁(可以多个同时读，但是如果还未获取到读锁，这时有写入的时候，读锁会等待)
        try {
            return Arrays.copyOf(counts, counts.length);
        } finally {
            rlock.unlock(); // 释放读锁
        }
    }
}
```

##### 1.3.1 应用场景

​		<font color='green'>同一个数据，有大量线程读取，但仅有少数线程修改。</font>如：论坛帖子、博客等，读多写少的情况下。

- `ReadWriteLock`只允许一个线程写入；
- `ReadWriteLock`允许多个线程在没有写入时同时读取（<font color='red'>如果同时有读写的时候，如果读先执行，则写等待。如果写先执行，则读等待。</font> ）；
- `ReadWriteLock`适合读多写少的场景。

#### 1.4 StampedLock

> 如果我们深入分析`ReadWriteLock`，会发现它有个潜在的问题：如果有线程正在读，写线程需要等待读线程释放锁后才能获取写锁，即读的过程中不允许写，这是一种悲观的读锁。

​		 要进一步提升并发执行效率，Java 8引入了新的读写锁：`StampedLock`。

​		`stampedLock`和`ReadWriteLock`相比，改进之处在于：<font color='green'>读的过程中也允许获取写锁后写入！这样一来，我们读的数据就可能不一致，所以，需要一点额外的代码来判断读的过程中是否有写入，这种读锁是一种乐观锁。</font>

​		<font color='red'>乐观锁的意思就是乐观地估计读的过程中大概率不会有写入，因此被称为乐观锁。反过来，悲观锁则是读的过程中拒绝有写入，也就是写入必须等待。显然乐观锁的并发效率更高，但一旦有小概率的写入导致读取的数据不一致，需要能检测出来，再读一遍就行。</font>

```java
public class Point {
    private final StampedLock stampedLock = new StampedLock();
    private double x;
    private double y;

    public void move(double deltaX, double deltaY) {
        long stamp = stampedLock.writeLock(); // 获取写锁
        try {
            x += deltaX;
            y += deltaY;
        } finally {
            stampedLock.unlockWrite(stamp); // 释放写锁
        }
    }

    public double distanceFromOrigin() {
        long stamp = stampedLock.tryOptimisticRead(); // 获得一个乐观读锁
        // 注意下面两行代码不是原子操作
        // 假设x,y = (100,200)
        double currentX = x;
        // 此处已读取到x=100，但x,y可能被写线程修改为(300,400)
        double currentY = y;
        // 此处已读取到y，如果没有写入，读取是正确的(100,200)
        // 如果有写入，读取是错误的(100,400)
        if (!stampedLock.validate(stamp)) { // 检查乐观读锁后是否有其他写锁发生
            stamp = stampedLock.readLock(); // 获取一个悲观读锁
            try {
                currentX = x;
                currentY = y;
            } finally {
                stampedLock.unlockRead(stamp); // 释放悲观读锁
            }
        }
        return Math.sqrt(currentX * currentX + currentY * currentY);
    }
}
```

和`ReadWriteLock`相比，写入的加锁是完全一样的，不同的是读取。注意到首先我们通过`tryOptimisticRead()`获取一个乐观读锁，并返回<font color='red'>版本号</font>。接着进行读取，读取完成后，我们通过`validate()`去验证版本号，如果在读取过程中没有写入，版本号不变，验证成功，我们就可以放心地继续后续操作。如果在读取过程中有写入，版本号会发生变化，验证将失败。在失败的时候，我们再通过获取悲观读锁再次读取。由于写入的概率不高，程序在绝大部分情况下可以通过乐观读锁获取数据，极少数情况下使用悲观读锁获取数据。

可见，`StampedLock`把读锁细分为乐观读和悲观读，能进一步提升并发效率。但这也是有代价的：一是代码更加复杂，二是`StampedLock`是不可重入锁，不能在一个线程中反复获取同一个锁。

`StampedLock`还提供了更复杂的将悲观读锁升级为写锁的功能，它主要使用在if-then-update的场景：即先读，如果读的数据满足条件，就返回，如果读的数据不满足条件，再尝试写。

##### 1.4.1 应用场景

- `StampedLock`提供了乐观读锁，可取代`ReadWriteLock`以进一步提升并发性能；
- `StampedLock`是不可重入锁。

#### 来源：

[廖雪峰博客之使用Condition](https://www.liaoxuefeng.com/wiki/1252599548343744/1306581033549858)

[廖雪峰博客之使用ReadWriteLock](https://www.liaoxuefeng.com/wiki/1252599548343744/1306581002092578)
