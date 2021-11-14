---
layout: post
title: "MySQL之InnoDB"
date: 2014-01-5 17:17:21
categories: DB MySQL
tags: MySQL InnoDB
updateTime: 2021-11-14
---

* content
{:toc}
### 1. [InnoDB 内存结构与磁盘结构](https://dev.mysql.com/doc/refman/8.0/en/innodb-architecture.html)

![2021-09-09_innodb-architecture](\image\db\innodb-architecture.png)

[图片来自官网](https://dev.mysql.com/doc/refman/8.0/en/innodb-architecture.html)

核心特点：`聚簇索引`、`修改缓冲区(chage buffer)`、`自适应hash索引 AHI`、`MVCC(多版本控制)`、`多缓冲区池(减少磁盘IO)`、`事务（数据安全性）`、`行级锁颗粒(并发控制)`、`外键`、`支持更多复制特性（主从/集群）`、`支持热备`、`自动故障恢复`、`双写机制 double write`

### 2. 核心模块

> 一条查询SQL语句如何执行的？

![2021-11-13_MySQL查询执行流程](\image\db\2021-11-13_MySQL查询执行流程.png)

> 一条update 语句如何执行？

=》查询数据流程如上图，具体更新过程：

![2021-11-13_MySQL更新执行流程](\image\db\2021-11-13_MySQL更新执行流程.png)



参考

https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html)

```mysql
-- 发展的眼光、对比看待 
```

