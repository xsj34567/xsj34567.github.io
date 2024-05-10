---
layout: post
title: 'kkFileView安装'
date: 2023-06-02
categories: kkFileView 安装
tags: kkFileView 安装
updateTime: 2023-06-02
---

- content
  {:toc}

## kkFileView

[kkFileView 官网](http://kkfileview.keking.cn/zh-cn/index.html)](https://help.fanruan.com/finereport/)

### 1.下载地址

> # Linux
>
> https://kkfileview.keking.cn/kkFileView-4.1.0.tar.gz

> # wins
>
> https://kkfileview.keking.cn/kkFileView-4.1.0.zip

### 2. 安装部署

[部署指南](https://kkfileview.keking.cn/zh-cn/docs/production.html)

#### 2.1 wins 安装

> 下载对应的版本，并解压，注意需要安装 JDK 及及其配置，解压之后进入 bin 目录，
>
> 运行 startup.bat 即可。

#### 2.2 linux 安装

[参考](https://blog.csdn.net/qq_42872034/article/details/129402164)

```shell

# 进入目录（如果没有，则手动创建)
 cd /opt/file-preivew

# 拉取安装包
wget https://kkfileview.keking.cn/kkFileView-4.1.0.tar.gz

# 解压 并重命名
tar -zxvf kkFileView-4.1.0.tar.gz && mv kkFileView-4.1.0 kkFileView

# 安装依赖组件
cd kkFileView/bin/
./install.sh

# 查看office 版本
 ./soffice --version

# 启动服务
cd /opt/file-preivew/kkFileView-4.1.0-SNAPSHOT/bin

 ./shutdown.sh

# 查看日志
cat /opt/file-preivew/kkFileView-4.1.0-SNAPSHOT/log/kkFileView.log


```

#### 2.3 docker 安装

```shell
# 1.拉取指定版本文件预览服务
docker pull keking/kkfileview:4.1.0
# 2. 运行
docker run -it -p 8012:8012 keking/kkfileview:4.1.0
```

访问地址：127.0.0.1:8012/index

<font color='red'>如果需要修改指定页面参数，可以通过 rar 压缩包方式打开，或者解压，修改之后，再覆盖</font>

### 参考

[kkFileView 官网](http://kkfileview.keking.cn/zh-cn/index.html)

[kkFileView 安装](https://blog.csdn.net/codeSmart/article/details/127918844)
