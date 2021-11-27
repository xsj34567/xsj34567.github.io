---
layout: post
title:  "高并发之Happens-Before"
date:   2018-08-02 
categories: 高并发 安全性 Happens-Before
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

#### 1. Happens-Before 模型

​	从JDK1.5 开始引入的。<font color='red'>多线程环境下，一个操作对另一个操作可见，必然存在happens-before关系。（可见性与有序性）</font>

##### 1.1 程序顺序规则

> as-if-serial：不管怎么重排序，单线程执行的结果都是不变的。

```java
int a = 10; // A
int b = 18; // B
int c = a * b;  //C
//不管 A、B怎么重排序，C是不允许重排序的，结果都是不变的。
```

##### 1.2 传递性原则

```java
//如同上面的例子：
// A happens-before B
// B happens-before C

//推导出： A happens-before C   （其中A、B可以重排，不影响结果。）
```

##### 1.3 volatile 变量规则

​	对于volatile修饰的变量的<font color='red'>写操作</font>，一定happens-before后续对于volatile变量的<font color='red'>读操作</font>。底层通过<font color='red'>内存屏障机制</font>来防止指令重排。

```java
public class VolatileTest{
    int a = 1;
    volatile boolean flag = false;
    public void write()
    {
        a = 2; //  1
        flag = true; // 2  后volatile写时，前面操作不能重排
    }    
    public void read(){
        if(flag)  // 3  先volatile读时，后面操作不能重排
        {
            int m = a; //4
        }
    }
    public static void main(String[] args) {
        VolatileTest vt = new VolatileTest();
        Thread t1 = new Thread(() -> {
            vt.write();
        });
        Thread t2 = new Thread(() -> {
            vt.read();
        });
//        t1.start();
//        t2.start();
        t2.start();
        t1.start();
    }
}

```

  **是否重排**																						**第二个操作**

| 第一个操作 | 普通读/写 | volatile 读 | volatile写 |
| :--------- | --------- | ----------- | ---------- |
| 普通读/写  |           |             | NO         |
| volatile读 | NO        | NO          | NO         |
| volatile写 |           | NO          | NO         |

<font color='red'>赋值为写,取值为读</font>

##### 1.4 监视器锁规则

​	 <font color='red'>synchronized</font>: 一个线程对于一个锁的释放操作，一定happen-before与后续线程对这个锁的加锁操作。

```java
public class SynchronizedTest{
	int m =10;
    public void test(){
        synchronized(this)  //此处自动加锁
        {
            // m 是共享变量，初始值为20
            if(m < 18)
            {
                this.m = 18;
            }
        }//此处自动释放锁
    }
}
```

- 假设m的初始值是10，线程A执行完代码块后，m的值会变成12，执行完成之后会释放锁。线程B进入代

  码块时，能够看到线程A对m的写操作，也就是B线程能够看到m=12。

##### 1.5 start规则

​     <font color='red'>start</font>：如果线程A执行操作ThreadB.start(),那么线程A的ThreadB.start()之前的操作happens-before线程B中的任意操作。

```java
public class StartTest{
  	int x=0;    
    public static void main(String[] args) {
          Thread t1=new Thread(()->{
           //主线程调用t1.start()之前
           //所有对共享变量的修改，此处皆可见   //此例中，x==20
              System.out.println(x);
          });        
        //修改共享变量的值
        x = 20;
        t1.start();
    }
}
```

##### 1.6 join规则

​	<font color='red'>join</font>规则：如果线程A执行操作ThreadB.join()并成功返回，那么线程B中的任意操作happens-before于线程A从ThreadB.join()操作成功的返回。

```java
public class JoinTest{
    int x =0;
    public static void main(String[] args) {
        Thread t1=new Thread(()->{
         //此处对共享变量x修改
         x=100;
        });
        //例如此处对共享变量修改，
        //则这个修改结果对线程t1可见
        //主线程启动子线程
        t1.start();
        t1.join()
        //子线程所有对共享变量的修改
        //在主线程调用t1.join()之后皆可见//此例中，x==100
    }
}
```



参考：

[GP-Mic](https://www.gupaoedu.com/team.html)

[ConcurrentHashMap线程安全吗](https://mp.weixin.qq.com/s/ZCQPrgW6iv2IP_3RKk016g)

[到底如何保证线程安全，你真的清楚吗？](https://mp.weixin.qq.com/s/VXzDrkR0ix5S9PNBs3SKhA)

