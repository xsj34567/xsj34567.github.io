---
layout: post
title:  "SpringBoot系列-核心组件"
date:   2018-06-01
categories: SpringBoot
tags: SpringBoot
---

* content
{:toc}

## 一、SpringBoot 核心组件
### 1. SpringBoot**核心**通过Maven继承依赖关系快速整合第三方框架

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.2.3.RELEASE</version>    
</parent>
<dependencies>
    <!-- SpringBoot 整合SpringMVC -->
    <!-- 我们依赖spring-boot-starter-web能够帮我整合Spring环境 原理通过Maven子父工程 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

- springboot 通过引用spring-boot-starter-web依赖，整合SpingMVC框架。当你添加了相应的starter模块，就相当于添加了相应的所有必须的依赖包，包括spring-boot-starter（这是**Spring** **Boot**的**核心**启动器，包含了自动配置、日志和YAML）；

- spring**-**boot-starter-test（支持常规的测试依赖，包括JUnit、Hamcrest、Mockito以及spring-test模块）；

- spring**-**boot-starter-web （支持全栈式Web开发，包括Tomcat和spring-webmvc）等相关依赖。

### 2  基于SpringMVC无配置文件（纯Java）完全注解化实现SpringBoot框架，Main函数启动。

```java
@SpringBootApplication 
public class Application {
    //方式一
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
    //方式二
    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(MySpringConfiguration.class);
        app.run(args);
    }
    //方式三
    public static void main(String[] args) {
        new SpringApplicationBuilder()
            .sources(Parent.class)
            .child(Application.class)
            .run(args);
    }
}
```

springboot有三种方式启动，都会在没有web.xml配置文件的情况，通过java代码操作整个SpringMVC的初始化过程，java代码最终会生成class文件,内置Tomcat就会加载这些class文件，当所有程序加载完成后，项目就可以访问了。

### 3. [核心注解](https://zhuanlan.zhihu.com/p/57689422)

- **@SpringBootApplication**

- **@EnableAutoConfiguration**

- **@Configuration**

- ### @ComponentScan

  ... ...

### 参考资料

​	[https://spring.io/guides/gs/spring-boot/](https://spring.io/guides/gs/spring-boot/)

​	[方志鹏博客](https://blog.csdn.net/forezp/article/details/70341651)

​	[Jar规范-官网](https://docs.oracle.com/javase/8/docs/technotes/guides/jar/jar.html)

​	[小马哥](https://github.com/mercyblitz)