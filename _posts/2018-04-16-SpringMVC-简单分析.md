---
layout: post
title:  "SpringMVC-简单分析"
date:   2018-04-16 
categories: Spring
tags: Spring SpringMVC
---

* content
{:toc}

## 一、SpringMVC工作机制
   主要介绍SpringMVC核心机制，请页面请求，到服务端经历哪些主要过程然后返回信息。



​    在容器初始化时会建立所有 url 和 Controller 的对应关系,保存到 `Map<url,Controller>` 中.Tomcat 启动时会通知 Spring 初始化容器(加载 Bean 的定义信息和初始化所有单例 Bean),然后 SpringMVC 会遍历容器中的 Bean,获取每一个 Controller 中的所有方法访问的 url,然后将 url 和 Controller 保存到一个 Map 中; 

​	这样就可以根据 Request 快速定位到 Controller,因为最终处理 Request 的是 Controller 中的 方法,Map 中只保留了 url 和 Controller 中的对应关系,所以要根据 Request 的 url 进一步确认 Controller 中的 Method,这一步工作的原理就是拼接 Controller 的 url(Controller 上 @RequestMapping 的值)和方法的 url(Method 上@RequestMapping 的值),与 request 的 url 进行匹 配,找到匹配的那个方法; 

确定处理请求的 Method 后,接下来的任务就是参数绑定,把 Request 中参数绑定到方法的形式参数 上,这一步是整个请求处理过程中最复杂的一个步骤。SpringMVC提供了两种Request参数与方法形参 的绑定方法:
	 1 通过注解进行绑定,@RequestParam 

​	 2 通过参数名称进行绑定. 使用注解进行绑定,我们只要在方法参数前面声明@RequestParam("a"),就可以将 Request 中参数 a 的值绑定到方法的该参数上.使用参数名称进行绑定的前提是必须要获取方法中参数的名称,Java 反射只提供了获取方法的参数的类型,并没有提供获取参数名称的方法.SpringMVC 解决这个问题的方 法是用 asm 框架读取字节码文件,来获取方法的参数名称.asm 框架是一个字节码操作框架,

​	关于 asm 更 多介绍可以参考它的官网。个人建议,使用注解来完成参数绑定,这样就可以省去 asm 框架的读取字节码 的操作。 



## 二、SpringMVC 流程分析

​     根据工作机制中三部分来分析 SpringMVC 的源代码。  

 其一,ApplicationContext 初始化时建立所有 url 和 Controller 类的对应关系(用 Map 保存);

 其二,根据请求 url 找到对应的 Controller,并从 Controller 中找到处理请求的方法;

 其三,request 参数绑定到方法的形参,执行方法处理请求,并返回结果视图 。

## 三、[SpringBoot使用过滤器、拦截器、切面(AOP)，及其之间的区别和执行顺序](https://www.cnblogs.com/java-spring/p/12742984.html)

> 范围：Filter > Servlet > Interceptor > Aspect > Controller  
>
> 顺序：Filter ——> Servlet ——> Interceptor ——> Aspect（环绕前、前置、环绕后、后置） ——> Controller  ——> Aspect ——> Interceptor ——> Servelet ——> Filter  (前置、后置)