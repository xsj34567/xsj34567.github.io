---
layout: post
title: "Zookeeper架构"
date: 2021-04-23
categories: 大数据 Zookeeper
tags: 大数据 Zookeeper
updateTime: 2021-11-11
---

* content
{:toc}
## Zookeeper架构

ZooKeeper是一个[分布式](https://baike.baidu.com/item/分布式/19276232)的，开放源码的[分布式应用程序](https://baike.baidu.com/item/分布式应用程序/9854429)协调服务，是[Google](https://baike.baidu.com/item/Google)的Chubby一个[开源](https://baike.baidu.com/item/开源/246339)的实现，是Hadoop和[Hbase](https://baike.baidu.com/item/Hbase/7670213)的重要组件。它是一个为分布式应用提供一致性服务的软件，提供的功能包括：配置维护、域名服务、分布式同步、组服务等。

### 1. Zookeeper数据模型

![2021-11-14_Zookeeper数据模型](\image\zookeeper\2021-11-14_Zookeeper数据模型.png)

> 自客户端的每个更新请求，ZooKeeper 都会分配一个全局唯一的递增编号

### 2. Zookeeper 选举模型

![2021-11-14_Zookeeper选举模型](\image\zookeeper\2021-11-14_Zookeeper选举模型.png)

> Leader ：既可以为客户端提供写服务又能提供读服务,Follower、Observer 提供只读服务	
>
> Follower：参与Leader投票选举，对外提供服务
>
> Observer：不选与Leader选举投票过程，只同步leader状态，及对外提供服务。
>
> Client：请求发起方



### 3. 数据一致性协议





### 参考

[如果你还不知道Apache Zookeeper？你凭什么拿大厂Offer！！](https://mp.weixin.qq.com/s/Vy2bXY5tFjiL508SvmrIkQ)

[ZooKeeper概念详解，最全整理](https://blog.csdn.net/jiahao1186/article/details/82633588)

