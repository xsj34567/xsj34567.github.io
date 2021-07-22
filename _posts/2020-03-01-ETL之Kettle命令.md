---
layout: post
title: "ETL之Kettle命令"
date: 2020-03-01
categories: 大数据 ETL 工具 Kettle
tags: 大数据 ETL
---

* content
{:toc}

## [ETL](https://baike.baidu.com/item/ETL/1251949?fr=aladdin)

[项目demo](https://gitee.com/xushj/etl-demo)

### 1. Kettle  JOB 

```shell
#!/bin/bash
# j_kettle_run.sh
# 运行示例： 注意路径  /opt/data_warehouse/file/ads/j_kettle_run.sh /opt/data_warehouse/file/ads/ j_kettle_ads_wlh_premises_data_crm_log 2021-07-20 
# desc：根据传入的job名称、tran路径以及跑数日期 
# author:admin
# dev_date：2021-07-22
# note :注意变量 与脚本中的路径并相应调整
source ~/.bash_profile
cd "$KETTLE_HOME"
tran_name=${2}
tran_dir=${1}
param_date=${3//-/}
param_date2=${3}
data_level=ads

#判断Log_dir 是否存在,不存在则创建目录
function checkmkdir()
{
if [ ! -d "${log_dir}" ]; then
  mkdir -p ${log_dir}
 if [ $? -eq 0 ];then
  echo "create dir ${log_dir} success" 
else 
 echo "create dir ${log_dir}  fail"
exit 1 
  fi
 else 
echo "log dir is ${log_dir}"
fi
}

#判断日期格式是否正确
function checkdate()
{
year=${param_date:0:4}
month=${param_date:4:2}
day=${param_date:6:2}
if [ ${#param_date} -ne 8 ];     
then 
	echo "Usage: bash $0 yyyymmdd format"
    exit 2
fi
if echo $day|grep -q '^0'
then
    day=`echo $day |sed 's/^0//'`
fi
if cal $month $year >/dev/null 2>/dev/null
then
    daym=`cal $month $year|egrep -v "$year|Su"|grep -w "$day"`
    if [ "$daym" != "" ]
    then
	log_dir=/opt/data_warehouse/log/${param_date}/${data_level}
    log_name=${tran_name}_${param_date}.log
        echo ok
		checkmkdir
    else
        echo "Error: Please input a right date."
    exit 2
    fi
else
    echo "Error: Please input a right date."
    exit 3
fi	
}

# 执行kettle 转换,根据传入的tran名称、tran路径以及跑数日期
function execjob()
{ 
echo “running the ${tran_name}”
./kitchen.sh  -file=${tran_dir}${tran_name}.kjb -level:Detailed  -param:DATA_DATE1=${param_date} -param:DATA_DATE2=${param_date2}  >>${log_dir}/${log_name} 2>&1
if [ $? -eq 0 ];then
 echo "the job run sucess....see the log info ${log_dir}/${tran_name}_${param_date}.log"
 else 
  echo " the job run fail,check the log info   ${log_dir}/${tran_name}_${param_date}.log" 
exit 3
fi
}

#主函数
function main()
{
checkdate ${tran_name} ${tran_dir} ${param_date}
execjob ${tran_name} ${tran_dir} ${param_date}
}
main ${tran_name} ${tran_dir} ${param_date}

```



### 2. Kettle Transform

```shell
# 核心： 启动命令差异
./pan.sh  -file=${tran_dir}${tran_name}.ktr -level:Detailed -param:DATA_DATE1=${param_date} -param:DATA_DATE2=${param_date2}  >>${log_dir}/${log_name} 2>&1
```



### 3. Kettle Java 

```shell
#!/bin/bash
# modify_desc:调用jar shell param_1:jar所在的路径 param_2:jar名称 param_3:运行日期
# modify_author:admin
# modify_date：2021-07-22
# note :注意变量 与脚本中的路径并相应调整
source ~/.bash_profile

jar_path_name=${1}
jar_name=${2}
param_date=${3//-/}
data_level_1=ods
data_level_2=dwd
data_level_3=dws
data_level_4=dm
data_level_5=ads
data_level=""

# 判断Log_dir 是否存在
function checkLogDir()
{
if [ ! -d "${log_dir}" ]; then
  mkdir -p ${log_dir}
 if [ $? -eq 0 ];then
  echo "create dir ${log_dir} success" 
 else 
  echo "create dir ${log_dir}  fail"
exit 1 
  fi
else 
 echo "log dir is ${log_dir}"
fi
}

# 判断jar包所处于的数仓层级,用于初始化data_level
function checkDataLevel()
{
if [[ ${jar_name} == *$data_level_1* ]]
 then
   data_level=${data_level_1}
elif [[ ${jar_name} == *$data_level_2* ]]
 then 	
 data_level=${data_level_2}
elif [[ ${jar_name} == *$data_level_3* ]]
 then 	
 data_level=${data_level_3}
 elif [[ ${jar_name} == *$data_level_4* ]]
 then 	
data_level=${data_level_4}
 elif [[ ${jar_name} == *$data_level_5* ]]
 then 	
 data_level=${data_level_5}
fi 
}

#判断日期格式是否正确
function checkDate()
{
year=${param_date:0:4}
month=${param_date:4:2}
day=${param_date:6:2}
if [ ${#param_date} -ne 8 ];     
then 
	echo "Usage: bash $0 yyyymmdd format"
    exit 2
fi
if echo $day|grep -q '^0'
then
    day=`echo $day |sed 's/^0//'`
fi
if cal $month $year >/dev/null 2>/dev/null
then
    daym=`cal $month $year|egrep -v "$year|Su"|grep -w "$day"`
    if [ "$daym" != "" ]
    then
	checkDataLevel
	log_dir=/opt/data_warehouse/log/${param_date}/${data_level}
    log_name=${jar_name}_${param_date}.log
        echo ok
		checkLogDir
    else
        echo "Error: Please input a right date."
    exit 2
    fi
else
    echo "Error: Please input a right date."
    exit 3
fi	
}

# 检查输入参数合法性
function checkParams(){
if [ $# != 3 ] ;
 then 
echo "need have 3 prameter,the first  is jar name,the second  is jar dir,the third is run_date" 
exit 4 
fi
}

# exec jar job
function execJarJob()
{
echo “start running the ${jar_name}”
jar_full_name=${jar_name}.jar
java -jar ${jar_path_name}${jar_full_name}>>${log_dir}/${log_name}
if [ $? -eq 0 ]; then
run_time=`date '+%Y-%m-%d %H:%M:%S'`
  echo "${run_time} INFO: 调用${jar_name}成功">>${log_dir}/${log_name}
  echo "the job run sucess....see the log info ${log_dir}/${log_name}"
else 
run_time=`date '+%Y-%m-%d %H:%M:%S'`
  echo "${run_time} error: 调用${jar_name}错误">>${log_dir}/${log_name}
  echo "the job run fail....see the log info ${log_dir}/${log_name}"
  exit 3  
fi
}

# 主方法
function main()
{
checkParams ${jar_path_name} ${jar_name} ${param_date}
checkDate ${jar_path_name} ${jar_name} ${param_date}
execJarJob ${jar_path_name} ${jar_name} ${param_date}
}
main ${jar_path_name} ${jar_name} ${param_date}

```

### 4. Kettle Python

```shell
python ..
```



#### 参考资料

[kettle源码](https://github.com/pentaho/pentaho-kettle)

[Kettle中文网](http://www.kettle.net.cn/category/demo)

[阿里-DataX](https://github.com/alibaba/DataX)

[Datax-Web](https://github.com/WeiYe-Jing/datax-web)

[ETL工具对比](https://www.cnblogs.com/DataPipeline2018/p/11131723.html)  偏向性较强

[Kettle 组件文档]

链接：https://pan.baidu.com/s/1DAOzqGHJId1KWR1Eb58RFQ 
提取码：95md 
