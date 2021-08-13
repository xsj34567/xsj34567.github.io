---
layout: post
title: "ETL之Kettle"
date: 2020-03-01
categories: 大数据 ETL 工具 Kettle
tags: 大数据 ETL
---

* content
{:toc}

## [一. ETL](https://baike.baidu.com/item/ETL/1251949?fr=aladdin)

```
Extract-Transform-Load: 抽取->转换->加载
抽取(Extract)：一般抽取过程需要连接到不同的数据源，以便为随后的步 骤提供数据。这一部分看上去简单而琐碎，实际上它是 ETL 解决方案的成 功实施的一个主要障碍。

转换(Transform)：任何对数据的处理过程都是转换。这些处理过程通常包
括（但不限于）下面一些操作：
移动数据
根据规则验证数据
数据内容和数据结构的修改
将多个数据源的数据集成
根据处理后的数据计算派生值和聚集值

加载(Load)：将数据加载到目标系统的所有操作。
```

### [1. Kettle项目示例](http://www.kettle.net.cn/blog/category/demo/)

>  [项目demo](https://gitee.com/xushj/etl-demo)

### 2. 常见问题

#### 2.1 编码问题处理方式

```
	<1. 启动时设置编码格式 UTF-8;
	<2. 检查数据库格式是否为UTF-8;
	<3. 工具平台是否为UTF-8；
	<4. 数据抽取时设置编码格式： characterEncoding UTF-8
	<5. 通过外部程序处理（Java、Python等处理数据格式）
```

#### 2.2 换行问题： \r\n

​		<1. 程序抽取式处理

#### 2.3 kettle 传参问题

- 基于Spoon  (右键 =》新建注释)

> 转换
> <1. 转换属性=》命名参数： DATA_DATE2 (启动时可传)
> <2. SQL语句中使用 '${DATA_DATE2}'

> 作业
>
> <1. 作业属性=》命名参数： DATA_DATE2 (启动时可传)
> <2. SQL语句中使用 '${DATA_DATE2}'
> <3. 编辑作业入口 =》 转换 =》 命名参数： DATA_DATE2  （从作业到转换传参）

#### 2.4 同步问题

> 作业： 等待SQL

> 调度：阻塞数据等待数据完成

![2021-08-05_etl](\image\etl\kettle\2021-08-05_etl.png)



### 参考资料

[kettle源码](https://github.com/pentaho/pentaho-kettle)

[Kettle中文网](http://www.kettle.net.cn/category/demo)

[阿里-DataX](https://github.com/alibaba/DataX)

[Datax-Web](https://github.com/WeiYe-Jing/datax-web)

[ETL工具对比](https://www.cnblogs.com/DataPipeline2018/p/11131723.html)  偏向性较强

[Kettle 组件文档]

链接：https://pan.baidu.com/s/1DAOzqGHJId1KWR1Eb58RFQ 
提取码：95md 

