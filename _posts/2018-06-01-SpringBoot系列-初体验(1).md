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
	[采用示例](https://gitee.com/xushj/springboot-start.git)  执行如下命令


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

  - Main-Class: org.springframework.boot.loader.JarLauncher 是jar启动的入口
  - Start-Class：com.mzz.springbootstart.SpringbootStartApplication 是我们系统启动的入口

- org.springframework.boot.loader: 需要加载的.class的目录

  - achive： 归档文件
  - ....





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