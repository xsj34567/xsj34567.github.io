---
layout: post
title: "MySQL之InnoDB架构"
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

#### 1.1 核心结构及特点

​	`聚簇索引`、`修改缓冲区(chage buffer)`、`自适应hash索引 AHI`、`MVCC(多版本控制)`、`多缓冲区池(减少磁盘IO)`、`事务（数据安全性）`、`行级锁颗粒(并发控制)`、`外键`、`支持更多复制特性（主从/集群）`、`支持热备`、`自动故障恢复`、`双写机制 double write`

#### 1.2 常用字符集

>  ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci
>
> 兼容emoji编码支持

[参考文档](https://blog.csdn.net/qq_37054881/article/details/90023611)

### 2. 核心模块

> 一条查询SQL语句如何执行的？

![2021-11-13_MySQL查询执行流程](\image\db\2021-11-13_MySQL查询执行流程.png)

> 一条update 语句如何执行？

=》查询数据流程如上图，具体更新过程：

![2021-11-13_MySQL更新执行流程](\image\db\2021-11-13_MySQL更新执行流程.png)



### 3. 主从配置

#### 3.1 开启binlog

```sql
[mysqld]
log-bin=mysql-bin
server_id=100  -- 注意主从都需配置，并且不同
```



#### 3.2 开启同步

```sql
# 在slave 上启动线程：
start slave;

# 查看状态 在 slave 上执行命令：

show slave status\G;

```



集群也是类似的：基于binlog  	<柏美迪康>

参考

https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html)

```mysql
-- 发展的眼光、对比看待 
```

