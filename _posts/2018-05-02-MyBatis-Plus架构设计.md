---

layout: post
title:  "MyBatis-Plus架构"
date:   2018-05-01
categories: Mybatis-Plus 框架
tags: Mybatis 框架
---

* content
{:toc}

## 1. MyBatisPlus架构

  了解MyBatis-Plus主要流程，便于快速分析问题，有大局观，从整体来把控、分析。

mybatis-plus 是基于mybatis增强版。

![2022-03-11_mybatis-plus原理](\image\mybatis\2022-03-11_mybatis-plus原理.png)

[mybatis-plus](https://www.oschina.net/p/mybatis-plus?hmsr=aladdin1e1)



## 2. 常见问题总结

### 2.1 分页插件

> ```
> 返回对象与第一次查询时使用的Mapper有关，建议先将条件组合，通过IN查询（但需要注意列编码需要一致） 
> --- 在线监控报警查询时
> ```

### 2.2 UNION 联合查询

> 可使用视图，同查询表操作类似。



[Gitee](https://gitee.com/baomidou/mybatis-plus)

### 参考

[mybatis-plus](https://www.oschina.net/p/mybatis-plus?hmsr=aladdin1e1)