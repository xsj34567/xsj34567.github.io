---
layout: post
title: "SparkDataSet实战"
date: 2021-04-30
categories: 大数据 Spark DataSet
tags: 大数据 Spark DataSet
updateTime: 2021-04-29
---

* content
{:toc}
## Spark DataSet

## 一、Spark DataSet 编程

### 1.1 什么是 Spark DataSet？

    Dataset是一个分布式的数据集。
    Dataset是Spark 1.6开始新引入的一个接口，它结合了RDD API的很多优点（包括强类型，支持lambda表达式等），以及Spark SQL的优点（优化后的执行引擎）。Dataset可以通过JVM对象来构造，然后通过transformation类算子（map，flatMap，filter等）来进行操作。Scala和Java的API中支持Dataset，但是Python不支持Dataset API。不过因为Python语言本身的天然动态特性，Dataset API的不少feature本身就已经具备了（比如可以通过row.columnName来直接获取某一行的某个字段）。R语言的情况跟Python也很类似。
    
    Dataframe就是按列组织的Dataset。
    在逻辑概念上，可以大概认为Dataframe等同于关系型数据库中的表，或者是Python/R语言中的data frame，但是在底层做了大量的优化。Dataframe可以通过很多方式来构造：比如结构化的数据文件，Hive表，数据库，已有的RDD。Scala，Java，Python，R等语言都支持Dataframe。在Scala API中，Dataframe就是Dataset[Row]的类型别名。在Java中，需要使用Dataset<Row>来代表一个Dataframe。



## 二、Spark DataSet 示例

### 1.1 DataSet的创建

#### 1.1.1 创建Spark Session

```scala

//创建SparkSession
val spark = SparkSession.builder()
  .master("local[*]")
  .appName("dataset")
  .enableHiveSupport()  //支持hive，如果代码中用不到hive的话，可以省略这一条
  .getOrCreate()

```

#### 1.1.2 序列创建DataSet

```json

//1、产生序列dataset
val numDS = spark.range(5, 100, 5)
numDS.orderBy(desc("id")).show(5)  //降序排序，显示5个
numDS.describe().show()  //打印numDS的摘要

+---+
| id|
+---+
| 95|
| 90|
| 85|
| 80|
| 75|

```

#### 1.1.3 集合创建DataSet

```json

//样例类
case class Person(name: String, age: Int, height: Int)
case class People(age: Int, names: String)
case class Score(name: String, grade: Int)

//然后定义隐式转换
import spark.implicits._

//定义集合，创建dataset
//2、集合转成dataset
val seq = Seq(Person("xzw1", 21, 171), Person("zm", 21, 192), Person("mm", 36, 168))
val dstmp = spark.createDataset(seq)
dstmp.show()

```

### 1.2 DataSet基础函数

```json
//1、DataSet存储类型
val seq1 = Seq(Person("xzw1", 21, 171), Person("zm", 21, 192), Person("mm", 36, 168))
val ds1 = spark.createDataset(seq1)
ds1.show()
ds1.checkpoint()
ds1.cache()
ds1.persist()
ds1.count()
ds1.unpersist(true)

//2、DataSet结构属性
ds1.columns
ds1.dtypes
ds1.explain()

//3、DataSet rdd数据互换
val rdd1 = ds1.rdd
val ds2 = rdd1.toDS()
ds2.show()
val df2 = rdd1.toDF()
df2.show()

//4、保存文件
df2.select("name", "age", "height").write.format("csv").save("./save")
```

### 1.3 DataSet Action

### 1.4 DataSet Transfrom

```sql
--当你的才华还撑不起你的野心时,就应该静下心来学习！
```



### 参考

[Spark DataSet API](http://spark.apache.org/docs/2.1.0/api/java/org/apache/spark/sql/Dataset.html)

[Spark DataSet介绍](https://blog.csdn.net/gdkyxy2013/article/details/89513268)