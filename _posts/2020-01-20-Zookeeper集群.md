---
layout: post
title: "Zookeeper集群"
date: 2021-04-23
categories: 大数据 Zookeeper
tags: 大数据 Zookeeper
updateTime: 2020-04-29
---

* content
{:toc}
## Zookeeper集群

ZooKeeper是一个[分布式](https://baike.baidu.com/item/分布式/19276232)的，开放源码的[分布式应用程序](https://baike.baidu.com/item/分布式应用程序/9854429)协调服务，是[Google](https://baike.baidu.com/item/Google)的Chubby一个[开源](https://baike.baidu.com/item/开源/246339)的实现，是Hadoop和[Hbase](https://baike.baidu.com/item/Hbase/7670213)的重要组件。它是一个为分布式应用提供一致性服务的软件，提供的功能包括：配置维护、域名服务、分布式同步、组服务等。

### [Zookeeper](https://baike.baidu.com/item/zookeeper/4836397?fr=aladdin) 安装

[zookeeper 3.6.0 下载地址](https://downloads.apache.org/zookeeper/zookeeper-3.6.0/apache-zookeeper-3.6.0-bin.tar.gz)

Master 节点操作：

   ```sh
cd /home/meizhangzheng

wget https://downloads.apache.org/zookeeper/zookeeper-3.6.0/apache-zookeeper-3.6.0-bin.tar.gz
   ```

#### 1. 解压 zookeeper 安装包  

```sh
#目录： /home/meizhangzheng

tar -zxvf apache-zookeeper-3.6.0-bin.tar.gz -C ~/data/

cd /home/meizhangzheng/data 

mv apache-zookeeper-3.6.0-bin zookeeper-3.6.0
```

#### 2. 配置zookeeper 
```sh
cd /home/meizhangzheng/data/zookeeper-3.6.0/conf

vi zoo.cfg

dataDir=/home/meizhangzheng/data/zk_data
dataLogDir=/home/meizhangzheng/data/zk_log
server.1=master:2888:3888
server.2=slave1:2888:3888
server.3=slave2:2888:3888
```
#### 3. 填写myid,master ,slave1,slave2 分别填写1，2，3  （每台主机填写一个）
```sh
cd /home/meizhangzheng

mkdir -p /home/meizhangzheng/data/zk_data
mkdir -p /home/meizhangzheng/data/zk_log

cd /home/meizhangzheng/data/zk_data

sudo vim myid
```

#### 4.复制到其他节点
```sh
scp -r ~/data/zookeeper-3.6.0/ meizhangzheng@slave1:~/data

scp -r ~/data/zookeeper-3.6.0/ meizhangzheng@slave2:~/data

scp -r ~/data/zk_data/ meizhangzheng@slave1:~/data

scp -r ~/data/zk_data/ meizhangzheng@slave2:~/data
```

#### 5. 启动校验
```sh
#5.1 启动
zkServer.sh start

#5.2 查看状态
zkServer.sh status

#5.3 查看当前进程
jps

#5.4 查看日志
cat zookeeper.out
zkServer.sh start-foreground

#5.5 停止
zkServer.sh stop
```
#### 6.注意事项

-- 日志信息
（1）mkdir: cannot create directory ‘/home/meizhangzheng/data/zookeeper-3.6.0/bin/../logs’: Permission denied

```sh
#设置日志环境变量
dataLogDir=/home/meizhangzheng/data/zk_log
```



（2） 无权限问题  : cannot create directory ‘/home/meizhangzheng/data/zookeeper-3.6.0/bin/../logs’: Permission denied

```shell
#设置拥有者/创建者 、 用户组
[meizhangzheng@master ~]$ sudo chown -R meizhangzheng data/
[meizhangzheng@master ~]$ sudo chgrp -R meizhangzheng data/
```

（3）不要乱用sudo 系统root身份执行



### 参考文档

[zookeeper 3.6.0 集群安装](https://www.cnblogs.com/renzhongpei/p/12748970.html)