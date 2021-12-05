---
layout: post
title:  "Dockerfile常用命令"
date:   2018-10-01 
categories: Docker 容器 Dockerfile
tags: Docker 容器 Dockerfile
---

* content
{:toc}

## [Dockerfile命令](https://www.runoob.com/docker/docker-dockerfile.html)

   容器运用比较广泛，微服务、CI。

   基础： docker version 、docker info 、docker-compose version ...

   [Docker中文](http://www.docker.org.cn/)    

### 1. 什么是 Dockerfile？

​	Dockerfile 是一个用来构建镜像的文本文件，文本内容包含了一条条构建镜像所需的指令和说明。



### 2. Dockerfile各指令

#### 2.1 FROM 和 RUN/CMD

- FROM:初始镜像来源
- RUN：构建docker build 时使用
- CMD: 构建容器时使用

<font color='red'>如果 Dockerfile 中如果存在多个 CMD 指令，仅最后一个生效。</font>



#### 2.2 COPY 与 ADD

- COPY：将构建镜像是文件或目录拷贝到目的地（官方推荐）。
- ADD：同COPY类似，针对压缩文件，能自动解压，可能会出现异常，在不解压的前提下，无法复制 tar 压缩文件。



#### 2.3 ENDPOINT 和 RUN

- ENDPOINT：命令行参数会被当作参数送给 ENTRYPOINT 指令指定的程序。

  ```dockerfile
  ENTRYPOINT ["<executeable>","<param1>","<param2>",...]
  # Dockerfile 中存在多个ENDPOINT 时，只有一个生效
  ENTRYPOINT ["nginx", "-c"] # 定参
  CMD ["/etc/nginx/nginx.conf"] # 变参 
  ```

#### 2.4 ENV 和AGR

- ENV：设置环境变量（设置环境变量，定义了环境变量，那么在后续的指令中，就可以使用这个环境变量。）
- AGR：只在dockerfile内有效（<font color='red'>docker build时有效，构建好的镜像内无效</font>）



#### 2.5 VOLUME卷

定义匿名数据卷。在启动容器时忘记挂载数据卷，会自动挂载到匿名卷。

作用：

- 避免重要的数据，因容器重启而丢失，这是非常致命的。
- 避免容器不断变大。



#### 2.6 EXPOSE

仅仅只是声明端口。

作用：

- 帮助镜像使用者理解这个镜像服务的守护端口，以方便配置映射。

- 在运行时使用随机端口映射时，也就是 docker run -P 时，会自动随机映射 EXPOSE 的端口。

  

#### 2.7 WORKDIR

指定工作目录。用 WORKDIR 指定的工作目录，会在构建镜像的每一层中都存在。（WORKDIR 指定的工作目录，必须是提前创建好的）。

docker build 构建镜像过程中的，每一个 RUN 命令都是新建的一层。只有通过 WORKDIR 创建的目录才会一直存在。



#### 2.8 ONBUILD

- 延迟构建（本次不构建，下次构建）



### 3. [DOCKER 镜像优化](https://blog.csdn.net/weixin_45727359/article/details/104243279)

- 基础镜像小 （Alpine)
- 层级尽量少
- 去除不必要
- 复用镜像层
- 分阶段构建





### 参考文档

[docker文档](https://docs.docker.com/)

[菜鸟教程-dockerfile](https://www.cnblogs.com/bakari/p/10443484.html)

 [DOCKER 镜像优化](https://blog.csdn.net/weixin_45727359/article/details/104243279)
