---
layout: post
title: "JavaWEB之Tomcat"
date: 2017-01-31
categories: Java 容器 性能优化 Tomcat
tags: Java 容器 性能优化
---

* content
{:toc}
## [Tomcat性能优化](https://www.cnblogs.com/dadonggg/p/8668386.html)

连接能力： BIO - > NIO -> APR (多路复用)


### 1  内存优化

	-- 设置JAVA_OPTS参数
	　　-server 启用jdk 的 server 版； 
	　　-Xms java虚拟机初始化时的最小内存； 
	　　-Xmx java虚拟机可使用的最大内存； 
	　　-XX: PermSize 内存永久保留区域 
	　　-XX:MaxPermSize 内存最大永久保留区域 

### 2  并发优化

> 调整连接器connector的并发处理能力；在Tomcat 配置文件 server.xml 中的

	<Connector port="9027"
	　　protocol="HTTP/1.1"
	　　maxHttpHeaderSize="8192"
	　　minProcessors="100"
	　　maxProcessors="1000"
	　　acceptCount="1000"
	　　redirectPort="8443"
	　　disableUploadTimeout="true"/>



### 3. 缓存压缩优化

> 开启浏览器的缓存，这样读取存放在webapps文件夹里的静态内容会更快，大大推动整体性能。

### 4. 安全优化

#### 4.1 降权启动

```java
//以普通用户启动tomcat，降权启动，防止不法分子通过tomcat获得root权限。
```



#### 4.2 修改端口号

```java
//修改tomcat配置文件server.xml中的全球人都知道的http连接器端口号，防止黑客攻击。
```

#### 4.3 更改关闭tomcat指令

```java
//#更改关闭Tomcat的指令
```

#### 4.4 修改管理员账号与密码

```xml
<!--修改tomcat-user.xml中默认的Manager用户名和密码 -->
<?xml version=’1.0’ encoding=’utf-8’?>
<tomcat-users>
    <role rolename=”manager”/>
    <user username=”temobi” password=”temobi8090” roles=”manager”/>
</tomcat-users>
```

#### 4.5 清空站点目录下ROOT下管理页面等文件

```java
// ROOT下有一些站点的管理程序可以查看tomcat的各种信息及配置，因此我们需要清空这些文件或者将站点目录更改。
```

### 5. 数据库优化

```java
// 数据库正常启动或关闭
// Tomcat性能在等待数据库查询被执行期间会降低
```



### 6. 其他优化

#### 6.1 错误页面优雅显示

#### 6.2 隐藏版本号

#### 6.3 禁用DNS

#### 6.4 设置 session过期时间









## 参考资料

[Tomcat优化](https://www.cnblogs.com/dadonggg/p/8668386.html)