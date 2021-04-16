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


### 一、示例步骤

#### 1.拉去镜像

```

docker pull jenkins/jenkins

```

#### 2.环境变量配置

```
#本文的挂载目录是home下
mkdir -p /usr/meizhangzheng/jenkins

```

#### 3.启动命令

```
#运用镜像启动容器命令

docker run -d -p 8080:8080 -v /usr/meizhangzheng/jenkins:/var/jenkins_home --name jenkins --restart always --privileged=true  -u root jenkins/jenkins

```

#### 4.完善镜像资源路径

```

# 修改hudson.model.UpdateCenter.xml 配置信息
sed -i  's/https:\/\/updates.jenkins.io\/update-center.json/https:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins\/updates\/update-center.json/g' /usr/meizhangzheng/jenkins/hudson.model.UpdateCenter.xml

# 修改updates/default.json 资源路径
sed -i  's/https:\/\/updates.jenkins.io\/download/https:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins/g' /usr/meizhangzheng/jenkins/updates/default.json

# 修改updates/default.json 资源路径
sed -i  's/http:\/\/www.google.com/http:\/\/www.baidu.com/g' /usr/meizhangzheng/jenkins/updates/default.json


```


#### 5.插件

```

# 选择插件 安装 SSH 、Git、Maven ...

# 配置Github、Gitee 账户信息

# 选择自由项目

```

### 参考文档

[github-jenkins自动构建](https://www.cnblogs.com/weschen/p/6867885.html)

[gitee-jenkins自动构建](https://gitee.com/help/articles/4193)

[jenkins/jenkins](https://www.cnblogs.com/dreammer/p/13670222.html)

[docker-jenkins](https://www.cnblogs.com/nhdlb/p/12576273.html)