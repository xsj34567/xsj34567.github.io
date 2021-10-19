---
layout: post
title:  "Docker Jenkins"
date:   2018-10-03 
categories: Docker Jenkins 容器 实战
tags: Docker Jenkins 容器 实战
---

* content
{:toc}

## Docker Jenkins

   基于Docker，安装Jenkins。

   建议直接安装（非Docker)


### 一、示例步骤

#### 1. 拉去镜像(lts:发行稳定版本)

```sh

docker pull jenkins/jenkins:lts

```

#### 2. 环境变量配置

```sh
#本文的挂载目录是home下
mkdir -p /usr/meizhangzheng/jenkins

```

#### 3. 启动命令

```sh
#运用镜像启动容器命令

docker run -d -p 8080:8080 -p 50000:50000 -v /usr/meizhangzheng/jenkins:/var/jenkins_home --name jenkins --restart always --privileged=true  -u root jenkins/jenkins:lts

# 进入容器，查看密码
docker exec -it jenkins /bin/bash
cat /var/jenkins_home/secrets/initialAdminPassword
```

#### 4. 完善镜像资源路径

```sh

# 修改hudson.model.UpdateCenter.xml 配置信息
sed -i  's/https:\/\/updates.jenkins.io\/update-center.json/https:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins\/updates\/update-center.json/g' /usr/meizhangzheng/jenkins/hudson.model.UpdateCenter.xml

# 修改updates/default.json 资源路径
sed -i  's/https:\/\/updates.jenkins.io\/download/https:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins/g' /usr/meizhangzheng/jenkins/updates/default.json

# 修改updates/default.json 资源路径
sed -i  's/http:\/\/www.google.com/http:\/\/www.baidu.com/g' /usr/meizhangzheng/jenkins/updates/default.json


```

- 手动修改密码

  ```sh
  #项目路径
  /usr/meizhangzheng/jenkins/users/
  
  # 外层路径
  /var/jenkins_home/users/  
  
  #密码修改为：123456
  <passwordHash>#jbcrypt:$2a$10$LxMm9HqAI/R4z7gL57qTouW/Mrz8uSaBpCGKvKc7K6dK.g/0yk/uq</passwordHash>
  ```

  

#### 5. 插件

```sh

# 选择插件 安装 SSH 、Git、Maven ...

# 配置Github、Gitee 账户信息

# 选择自由项目

```

#### 6. 事例分析

背景：构建前端项目失败;

![2021-08-09_jenkins](\image\测试\jenkins\2021-08-09_jenkins.png)

分析：联系上下文分析，具体分析走到哪一步，是否项目代码或业务问题；据分析代码拉去没问题，是引用插件问题。

### 参考文档

[github-jenkins自动构建](https://www.cnblogs.com/weschen/p/6867885.html)

[gitee-jenkins自动构建](https://gitee.com/help/articles/4193)

[jenkins/jenkins](https://www.cnblogs.com/dreammer/p/13670222.html)

[docker-jenkins](https://www.cnblogs.com/nhdlb/p/12576273.html)

[手动修改密码](https://blog.csdn.net/weixin_39773337/article/details/109035933)