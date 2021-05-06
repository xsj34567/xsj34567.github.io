---
layout: post
title: "Spark DataFrame实战"
date: 2021-04-30
categories: 大数据 Spark DataFrame
tags: 大数据 Spark DataFrame
updateTime: 2021-04-29
---

* content
{:toc}
## Spark DataFrame

## 一、Spark DataFrame SQL 编程

### 1.1 什么是 Spark DataFrame & SQL？

    Spark DataFrame 是一个带有列名称的分布式数据集，类似于关系型数据库中的一张表，可以通过结构化的数据文件，Hive中的表，外部数据库以及已经存在的RDD得到。
    
    Spark SQL 是使用 SQL 或 HiveQL 语法编写 SQL 语句，来执行计算任务。

### 1.2 Spark DataFrame & SQL 引入动机？

    1）Spark DataFrame & SQL 执行效率相比 RDD 更高
    
    2）Spark DataFrame & SQL 代码更简洁
    
    3）Spark DataFrame & SQL 带有自动优化程序引擎
### 1.3 Spark DataFrame 与 RDD区别

```
DataFrame = RDD[Row] + shcema
```



## 二、Spark DataFrame SQL 示例

### 1.1 DataFrame 读取数据

```scala

// json 文件路径

val path = "file:///home/mezhangzhenng/hdoop/tmp/people.json"
val people = spark.read.json(path)

// 打印 DataFrame 的元信息

people.printSchema()

// 查看前五条数据

people.show(5)


```


-- people.json 文件内容

```json
{"name":"mzz"}
{"name":"test1","age":20}
{"name":"test2","age":30}
{"name":"test2","age":40}


```

#### 1.1.1 将people.json 文件上传到hadoop 的 /data/tmp/people.json 文件下

```shell

    cd  /home/meizhangzheng/hadoop/tmp 

    vim people.json
    
    hadoop fs -put people.json /data/tmp/people.json


```

#### 1.1.2 读取文件（spark2.0 以后，不使用sqlContext而使用spark）

```shell
    
    scala> val people = spark.read.json("/data/tmp/people.json")
    people: org.apache.spark.sql.DataFrame = [age: bigint, name: string] 

```

#### 1.1.3 操作

```shell

# 打印元信息

scala> people.printSchema()
root
 |-- age: long (nullable = true)
 |-- name: string (nullable = true)


# 查看前五条数据

scala> people.show(5)
+----+-----+                                                                    
| age| name|
+----+-----+
|null|  mzz|
|  20|test1|
|  30|test2|
|  40|test2|
+----+-----+


```

### 1.2 DataFrame/SQL 计算结果

```shell

val people = spark.read.json("/data/tmp/people.json")

// 得到 年龄在 13 ~ 19 之间的人的姓名
val teensOne = people.where("age >= 20 and age <=30").select("name")
teensOne.show(3)

// 将 DataFrame 注册为临时表

people.registerTempTable("people")

// 通过 SQL 语句来计算任务

val teensTwo = spark.sql("SELECT name FROM people WHERE age >= 20 AND age <= 30")

teensTwo.show(3)


//按照年龄分组

people.groupBy("name").agg(max("age")).show()
+-----+--------+                                                                
| name|max(age)|
+-----+--------+
|test1|      20|
|test2|      40|
|  mzz|    null|
+-----+--------+


//语句：

spark.sql("select name,max(age) as age from people where age>20 and age<= 30 group by name").show()

```

### 1.3 RDD 与 DataFrame 相互转化

people.txt内容：
test1,20
test2,40
test2,30

cd  /home/meizhangzheng/hadoop/tmp 

vi people.txt  #输入people.txt内容

hadoop fs -put people.txt /data/tmp/people.txt

#### 1.3.1RDD => DataFrame 
定义people类

```shell

scala> case class Person(name:String,age:Int)
defined class Person

# 得到一个RDD
scala> val people = sc.textFile("/data/tmp/people.txt")
people: org.apache.spark.rdd.RDD[String] = /data/tmp/people.txt MapPartitionsRDD[1] at textFile at <console>:24

scala> people.map(_.split(","))
res2: org.apache.spark.rdd.RDD[Array[String]] = MapPartitionsRDD[2] at map at <console>:26

scala> people.map(_.split(",")).map(p=>Person(p(0),p(1).trim.toInt))
res3: org.apache.spark.rdd.RDD[Person] = MapPartitionsRDD[4] at map at <console>:28

# RDD已经转化为DataFrame
scala> people.map(_.split(",")).map(p=>Person(p(0),p(1).trim.toInt)).toDF()
res4: org.apache.spark.sql.DataFrame = [name: string, age: int]

scala> val p2 =  people.map(_.split(",")).map(p=>Person(p(0),p(1).trim.toInt)).toDF()
p2: org.apache.spark.sql.DataFrame = [name: string, age: int]

scala> p2.show()
+-----+---+                                                                     
| name|age|
+-----+---+
|   t1| 20|
|test2| 40|
|test2| 30|
+-----+---+

```

#### 1.3.2 DataFrame => RDD

```shell

scala> p2.rdd
res6: org.apache.spark.rdd.RDD[org.apache.spark.sql.Row] = MapPartitionsRDD[17] at rdd at <console>:26

scala> val p3=p2.rdd
p3: org.apache.spark.rdd.RDD[org.apache.spark.sql.Row] = MapPartitionsRDD[17] at rdd at <console>:26


# 显示RDD 内容
scala> p3.collect
res9: Array[org.apache.spark.sql.Row] = Array([t1,20], [test2,40], [test2,30]) 

```