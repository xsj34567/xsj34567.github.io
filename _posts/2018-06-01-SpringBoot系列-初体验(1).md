---
layout: post
title:  "SpringBoot系列-初体验(1)"
date:   2018-06-01
categories: SpringBoot
tags: SpringBoot
---

* content
{:toc}

## SpringBoot 示例



### SpringBoot 初体验

#### 示例项目

​	快速构建项目：https://start.spring.io/  或
	采用[示例](https://gitee.com/xushj/springboot-start.git)  执行如下命令


```java
git clone https://gitee.com/xushj/springboot-start.git
mvn clean compile springboot-start
java -jar /target/springboot-start-0.0.1-SNAPSHOT.jar
```

​	也可参考[官网示例](https://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#getting-started-first-application)

启动结果如下：

```java
2018-08-09 11:49:06.982  INFO 14061 --- [           main] c.m.s.SpringbootStartApplication         : Started SpringbootStartApplication in 4.985 seconds (JVM running for 7.065)
```

## SpringBoot 分析

### [基本特征](https://docs.spring.io/spring-boot/docs/2.0.4.RELEASE/reference/htmlsingle/)

- 应用分为两个方面：功能性、非功能性

  - 功能性：系统所设计的业务范畴
  - 非功能性：安全、性能、监控、数据指标（CPU利用率、网卡使用率）

  Spring Boot 规约大于配置，大多数组件，不需要自行配置，而是自动组装！简化开发，大多数情况，使用默认即可！

  production-ready 就是非功能性范畴！

  

- [嵌入式容器：](https://docs.spring.io/spring-boot/docs/2.0.4.RELEASE/reference/htmlsingle/#howto-embedded-web-servers) 独立Spring 应用，不需要外部依赖，依赖容器（Tomcat）

  - `Tomcat`、 `Jetty`、`Undertow`

    

- [外部配置](https://docs.spring.io/spring-boot/docs/2.0.4.RELEASE/reference/htmlsingle/#boot-features-external-config)：

  - 启动参数

  - 配置文件

  - 环境变量

  - ...

    

- 外部应用：

  - Servlet 应用

  - Spring Web MVC

  - Spring Web Flux

  - [WebSocket](https://docs.spring.io/spring-boot/docs/2.0.4.RELEASE/reference/htmlsingle/#boot-features-websockets)

  - WebService

  - ...

    

- SQL：

  - JDBC、JPA、ORM ...

    

- [NoSQL（Not Only SQL）：](https://docs.spring.io/spring-boot/docs/2.0.4.RELEASE/reference/htmlsingle/#boot-features-nosql)

  - Redis、ElasticSearch、Hbase ...



- Reactive
  - Mono : 0 - 1 元素，Optional
  - Flux：0 - N 个元素，类似于 Iterable 或者 Collection

Req -> WebFlux -> 1 - N 线程执行任务执行函数式任务

它是推的方式！

Java 9 里面API 称之为 Flow（流）

​	Publisher -> publish(1)

​	Subscription（1）：订阅消息

​	Subs(A)#onNext() -> Subs(B)#onNext() -> Subs(C)#onNext()



​	Reactive 是推模式（Push）

​	Iterator 是拉模式（Pull）

### 项目如何启动

​	1. mvn install 之后，会生成`springboot-start-0.0.1-SNAPSHOT.jar`、`springboot-start-0.0.1-SNAPSHOT.jar.original`包

```java
springboot-start-0.0.1-SNAPSHOT.jar  -->spring boot maven插件生成的jar包，里面包含了应用的依赖，以及spring boot相关的类
springboot-start-0.0.1-SNAPSHOT.jar.original -->默认的maven-jar-plugin生成的包
```



2.分析springboot-start-0.0.1-SNAPSHOT.jar 目录结构

```java
├── BOOT-INF
|   └──classes
|	   └──com
|		  └──mzz
|			└──springbootstart
|       		└──SpringbootStartApplication.class
|		
│   ├── lib
│   ├── aopalliance-1.0.jar
│   ├── spring-beans-4.2.3.RELEASE.jar
│   ├── ...  	
│       	
├── META-INF
│   ├── MANIFEST.MF
├── application.properties
|
└── org
    └── springframework
        └── boot
            └── loader
                ├── ExecutableArchiveLauncher.class
                ├── JarLauncher.class
                ├── JavaAgentDetector.class
                ├── LaunchedURLClassLoader.class
                ├── Launcher.class
                ├── MainMethodRunner.class
                ├── achive  
                ├── data
                ├── jar
                ├── util  
                ├── ...   
```

​	

- BOOT-INFO：

  - classes: 编译文件

  - lib: 项目所需要的依赖包

    

- META-INF：

  ```java
  Manifest-Version: 1.0
  Implementation-Title: springboot-start
  Implementation-Version: 0.0.1-SNAPSHOT
  Built-By: xushijian
  Implementation-Vendor-Id: com.mzz
  Spring-Boot-Version: 2.0.4.RELEASE
  Main-Class: org.springframework.boot.loader.JarLauncher
  Start-Class: com.mzz.springbootstart.SpringbootStartApplication
  Spring-Boot-Classes: BOOT-INF/classes/
  Spring-Boot-Lib: BOOT-INF/lib/
  Created-By: Apache Maven 3.5.3
  Build-Jdk: 1.8.0_101
  Implementation-URL: https://projects.spring.io/spring-boot/#/spring-bo
   ot-starter-parent/springboot-start
  ```

  - Main-Class: `org.springframework.boot.loader.JarLauncher` 是jar启动的入口
  - Start-Class：`com.mzz.springbootstart.SpringbootStartApplication` 是我们系统启动的入口

- `org.springframework.boot.loader`: 需要加载的.class的目录

  - achive： 归档文件
  - ....

* 除了 jar 或者 war 启动的方式，还有目录启动方式

  * 目录启动方式可以帮助解决老旧的jar 不支持 Spring Boot 新方式，比如老版本的 MyBatis
    * 如果是 jar 包，解压后，跳转解压目录，并且执行`java`命令启动，启动类是 org.springframework.boot.loader.JarLauncher
    * 如果是 war包，解压后，跳转解压目录，并且执行`java`命令启动类是org.springframework.boot.loader.WarLauncher



###  启动流程

- 判断是否存在web环境  Note: `SpringBoot 2` 与 `SpringBoot 1.X` 不同 

  ```java
  private WebApplicationType deduceWebApplicationType() {
          if (ClassUtils.isPresent("org.springframework.web.reactive.DispatcherHandler", (ClassLoader)null) && !ClassUtils.isPresent("org.springframework.web.servlet.DispatcherServlet", (ClassLoader)null) && !ClassUtils.isPresent("org.glassfish.jersey.server.ResourceConfig", (ClassLoader)null)) {
              return WebApplicationType.REACTIVE;
          } else {
              String[] var1 = WEB_ENVIRONMENT_CLASSES;
              int var2 = var1.length;
  
              for(int var3 = 0; var3 < var2; ++var3) {
                  String className = var1[var3];
                  if (!ClassUtils.isPresent(className, (ClassLoader)null)) {
                      return WebApplicationType.NONE;
                  }
              }
  
              return WebApplicationType.SERVLET;
          }
      }
  ```

  - 如果是reactive，则为`WebApplicationType.REACTIVE`

  - 如果是都不包含，则为`WebApplicationType.NONE`

  -   否则就是`WebApplicationType.SERVLET`

    ```java
    //org.springframework.boot.SpringApplication
    protected ConfigurableApplicationContext createApplicationContext() {
            Class<?> contextClass = this.applicationContextClass;
            if (contextClass == null) {
                try {
                    switch(this.webApplicationType) {
                    case SERVLET:
                        contextClass = Class.forName("org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext");
                        break;
                    case REACTIVE:
                        contextClass = Class.forName("org.springframework.boot.web.reactive.context.AnnotationConfigReactiveWebServerApplicationContext");
                        break;
                    default:
                        contextClass = Class.forName("org.springframework.context.annotation.AnnotationConfigApplicationContext");
                    }
                } catch (ClassNotFoundException var3) {
                    throw new IllegalStateException("Unable create a default ApplicationContext, please specify an ApplicationContextClass", var3);
                }
            }
        return (ConfigurableApplicationContext)BeanUtils.instantiateClass(contextClass);
    }
    
    ```

- Jar 、War运行过程

  - JarLauncher、WarLauncher

    - ExecutableArchiveLauncher

      ```java
      protected String getMainClass() throws Exception {
              Manifest manifest = this.archive.getManifest();
              String mainClass = null;
              if (manifest != null) {
                  mainClass = manifest.getMainAttributes().getValue("Start-Class");
              }
      
              if (mainClass == null) {
                  throw new IllegalStateException("No 'Start-Class' manifest entry specified in " + this);
              } else {
                  return mainClass;
              }
          }
      ```

      获取主类信息 (启动类信息）

  - MainMethodRunner

    ```java
    public void run() throws Exception {
            Class<?> mainClass = Thread.currentThread().getContextClassLoader().loadClass(this.mainClassName);
            Method mainMethod = mainClass.getDeclaredMethod("main", String[].class);
            mainMethod.invoke((Object)null, this.args);
        }
    ```

    通过`Thread.currentThread().getContextClassLoader().loadClass(this.mainClassName);` 获取主类，再通过反射启动该类。

## 总结

​	本文简单介绍了SpringBoot如何启动，详细启动，可结合Spring启动做比较。 



### 参考资料

​	[https://spring.io/guides/gs/spring-boot/](https://spring.io/guides/gs/spring-boot/)

​	[方志鹏博客](https://blog.csdn.net/forezp/article/details/70341651)

​	[Jar规范-官网](https://docs.oracle.com/javase/8/docs/technotes/guides/jar/jar.html)

​	小马哥