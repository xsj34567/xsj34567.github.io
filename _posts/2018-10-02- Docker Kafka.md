---
layout: post
title:  "Docker Kafka"
date:   2018-10-02 
categories: Docker Kafka 容器
tags: Docker Kafak 容器
---

* content
{:toc}

## Docker Kafka

   基于Docker，安装kafka。


### 示例步骤

拉去镜像

```shell script

docker pull wurstmeister/zookeeper

docker pull wurstmeister/kafka

```


docker-compose.yml   windows  注意修改相应的IP、主题

```shell script

version: '3'
services:
  zookeeper:
    image: wurstmeister/zookeeper
    ports:
      - "2181:2181"
  kafka:
    image: wurstmeister/kafka
    depends_on: [ zookeeper ]
    ports:
      - "9092:9092"
    environment:
      KAFKA_ADVERTISED_HOST_NAME: 127.0.0.1
      KAFKA_CREATE_TOPICS: "test:1:1"
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
```


```shell script

# 服务打包
docker-compose build


# 启动docker镜像服务
docker-compose up -d


# 进入容器 
docker exec -it kafka_kafka_1 bash


# 启动生产者，可生产数据
bash-4.4# $KAFKA_HOME/bin/kafka-console-producer.sh --topic=test --broker-list kafka_kafka_1:9092
>ni
>haha


# 新建窗口，启动消费者（可查看生产的数据）
bash-4.4# $KAFKA_HOME/bin/kafka-console-consumer.sh --bootstrap-server kafka_kafka_1:9092 --from-beginning --topic test
ni
haha


# 启动后端管理（展示不能展示，Windows下路径）
docker run -itd --name=kafka-manager -p 9000:9000 -e ZK_HOSTS="127.0.0.1:2181" sheepkiller/kafka-manager

```

### 问题

#### 问题1

注： 安装docker-compose (注意版本号：github)

```shell script

curl -L "https://github.com/docker/compose/releases/download/1.27.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose

```

#### 问题2

```
Kafka-manager出现错误，Yikes! Ask timed out on [ActorSelection[Anchor(akka://kafka-manager-system/)


解决办法：启动kafka-manager后台管理时，修改zk 的ip地址 ，将127.0.01 修改为实际IP

docker run -itd --name=kafka-manager -p 9000:9000 -e ZK_HOSTS="127.0.0.1:2181" sheepkiller/kafka-manager

```

参考文档：

[Docker 安装](https://www.cnblogs.com/zhaoxxnbsp/p/13065722.html)

[docker-kafka示例文档](https://www.jianshu.com/p/0edcc3addf3f)

[docker-kafka安装文档](https://blog.csdn.net/sayoko06/article/details/104020621)

[kafka-manager安装文档](https://cloud.tencent.com/developer/article/1141507)

https://github.com/wurstmeister/kafka-docker/blob/master/Dockerfile

