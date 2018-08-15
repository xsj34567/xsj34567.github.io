---
layout: post
title:  "SpringBoot系列-SpringApplication(2)"
date:   2018-06-01
categories: SpringBoot
tags: SpringBoot SpringApplication
---

* content
{:toc}

## SpringApplication

  介绍SpringApplication启动,用Spring实现类似的功能。



### 自定义 SpringApplication  (Spring)

​	介绍 `SpringApplication ` `SpringApplicationBuilder `API 调整

> The `SpringApplication` class provides a convenient way to bootstrap a Spring application that is started from a `main()` method. In many situations, you can delegate to the static `SpringApplication.run` method

[来源](https://docs.spring.io/spring-boot/docs/2.0.4.RELEASE/reference/htmlsingle/#boot-features-spring-application)  `SpringApplication` 提供了简便的方式来启动Spring 应用。 SpringBootApplication源码：

```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = {
		@Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
    ...
}
```
其中`@SpringBootConfiguration`(作为配置项SpringBoot注解)源码：

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration
public @interface SpringBootConfiguration {
}

@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage   //自动配置包
@Import({EnableAutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration { // 激活自动装配 `@Enable` -> `@Enable` 开头的
	...
}

@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE})
@Documented
@Repeatable(ComponentScans.class)//允许重复包
public @interface ComponentScan {  // ----> Spring Framework 3.1 （IDEA 快捷键可查看）
	...
}

--> ComponentScans
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE})
@Documented
public @interface ComponentScans {
    ComponentScan[] value(); 
}
```



相当于`@SpringBootApplication` 等价于`@ComponentScan`+`@EnableAutoConfiguration`+`@Configuration` 。

`@ComponentScan`:扫描指定路径下的包

`@EnableAutoConfiguration`:应用所声明的依赖对Spring框架自动配置。如：

添加`spring-boot-starter-web`就添加了`Tomcat`和`SpringMVC`的依赖，类似于子项目自动添加父项目的依赖。

#### [`@Component` 的“派生性”](https://github.com/spring-projects/spring-framework/wiki/Spring-Annotation-Programming-Model)

> `@Component` is a generic stereotype for any Spring-managed component. Any component annotated with `@Component` is a candidate for component scanning. Similarly, any component annotated with an annotation that is itself meta-annotated with `@Component` is also a candidate for component scanning. For example, `@Service` is meta-annotated with `@Component`.

> Core Spring provides several stereotype annotations out of the box, including but not limited to: `@Component`, `@Service`, `@Repository`, `@Controller`, `@RestController`, and `@Configuration`. `@Repository`, `@Service`, etc. are specializations of `@Component`.

`@Component` -> `@ComponentScan`  阅读源码

处理类 -> `ConfigurationClassParser`

扫描类 -> 

* `ClassPathBeanDefinitionScanner`

  * `ClassPathScanningCandidateComponentProvider`

    ```java
    	protected void registerDefaultFilters() {
    		this.includeFilters.add(new AnnotationTypeFilter(Component.class));
        	...
    	}
    ```

##### Spring 注解驱动示例(自定义实现)

注解驱动上下文 `AnnotationConfigApplicationContext` ， Spring Framework 3.0 开始引入的

```java
@Configuration
public class SpringAnnotationDemo {

    public static void main(String[] args) {

        //   XML 配置文件驱动       ClassPathXmlApplicationContext
        // Annotation 驱动
        // 找 BeanDefinition
        // @Bean @Configuration
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
        // 注册一个 Configuration Class = SpringAnnotationDemo
        context.register(SpringAnnotationDemo.class);
        // 上下文启动
        context.refresh();

        System.out.println(context.getBean(SpringAnnotationDemo.class));

    }
}
```

##### SprigBoot基本写法

```java
SpringApplication springApplication = new SpringApplication(MicroservicesProjectApplication.class);
        Map<String,Object> properties = new LinkedHashMap<>();
        properties.put("server.port",0);
        springApplication.setDefaultProperties(properties);
        springApplication.run(args);
```
##### `SpringApplicationBuilder`

```java
new SpringApplicationBuilder(MicroservicesProjectApplication.class) // Fluent API
    // 单元测试是 PORT = RANDOM
    .properties("server.port=0")  // 随机向 OS 要可用端口
    .run(args);
```

#### Spring Boot 引导示例

- SpringBoot 与Spring 关联性  (如：获取应用上下文 `Context`等)

```java
@SpringBootApplication
public class MicroservicesProjectApplication {
    public static void main(String[] args) {
        SpringApplication springApplication = new SpringApplication(MicroservicesProjectApplication.class);
        //配置系统参数信息
        Map<String,Object> properties = new LinkedHashMap<>();
        properties.put("server.port",0);//随机端口
        springApplication.setDefaultProperties(properties);
        //设置为非WEB应用
        springApplication.setWebApplicationType(WebApplicationType.NONE);

        //配置类
        ConfigurableApplicationContext context = springApplication.run(args);

        //是否有异常？ --> 是否已初始化
        System.out.println(context.getBean(MicroservicesProjectApplication.class));

        //输出SpringBoot 应用的Application的类名
        System.out.println("当前Spring 应用上下文的类:" + context.getClass().getName()); 
    }
}
```


### 配置Spring Boot源

​	理解Spring Boot 配置  --> 类比Spring配置

##### ConfigFileApplicationListener

管理配置文件，比如：`application.properties` 以及 `application.yaml`

`application-{profile}.properties`:

profile  = dev 、test

1. `application-{profile}.properties`
2. application.properties


### SpringAppliation 类型推断（SpringBoot 2.X)

​	判断Web 应用类型（`REACTIVE`、`NONE`、`SERVLET`）   源码阅读：

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

### Spring Boot 事件

​	介绍 Spring Boot 事件与 Spring Framework 事件之间的差异和联系



参考：[GP-小马哥](https://www.gupaoedu.com/team.html)