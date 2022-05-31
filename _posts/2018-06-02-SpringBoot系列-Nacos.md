---
layout: post
title:  "SpringBoot系列-Nacos"
date:   2018-06-01
categories: SpringBoot Nacos
tags: SpringBoot Nacos
---

* content
{:toc}

## 一、[Nacos](https://nacos.io/zh-cn/docs/what-is-nacos.html)

  介绍Nacos,主要包括服务发现、配置管理、服务治理、UI、社区活跃。

###  1.  基础架构

​	[基础架构及功能图](https://nacos.io/zh-cn/docs/architecture.html)

### 2. [快速开始](https://nacos.io/zh-cn/docs/quick-start.html)

```shell
# 下载并编译
git clone https://github.com/alibaba/nacos.git
cd nacos/
mvn -Prelease-nacos -Dmaven.test.skip=true clean install -U  
ls -al distribution/target/

// change the $version to your actual path
cd distribution/target/nacos-server-$version/nacos/bin

# 启动 （非集群稳定版本 ）
sh startup.sh -m standalone

# 访问路径
# http://127.0.0.1:8848/nacos
```

[来源](https://nacos.io/zh-cn/docs/quick-start.html)

[支持优秀开源项目RuoYi](https://gitee.com/y_project/RuoYi)





### 参考

[Nacos 官方文档](https://nacos.io/zh-cn/docs/what-is-nacos.html)

[小马哥](https://github.com/mercyblitz)

