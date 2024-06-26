---
layout: post
title: 'Docker MySQL'
date: 2018-10-02
categories: Docker MySQL 实战
tags: Docker MySQL 容器 实战
---

- content
  {:toc}

## Docker MySQL

[非 Docker 安装 MySQL](https://www.runoob.com/mysql/mysql-install.html)

基于 Docker，安装 MySQL。

### 一、安装步骤

### 1.拉取镜像

```shell
# docker pull meizhangzheng/mysql:8.022
docker pull mysql:8.0.11
```

### 2. 初始化

#### 2.1 配置文件路径

```shell
mkdir -p /usr/local/mysqlData/test/cnf
mkdir -p /usr/local/mysqlData/test/data
vi /usr/local/mysqlData/test/cnf/my.cnf
```

#### 2.2 配置 my.cnf

配置来源与 mysql 8 的 my.ini 文件，修改了其中的路径信息

```shell
[client]

# pipe=
# socket=mysql=mysql=mysql=MYSQL

port=3306

[mysql]
no-beep=

# default-character-set=

# SERVER SECTION
# ----------------------------------------------------------------------
#
# The following options will be read by the MySQL Server. Make sure that
# you have installed the server correctly (see above) so it reads this
# file.=
#
# server_type=3
[mysqld]

# The next three options are mutually exclusive to SERVER_PORT below.
# skip-networking=
# enable-named-pipe=
# shared-memory=

# shared-memory-base-name=MYSQL

# The Pipe the MySQL Server will use
# socket=mysql=mysql=mysql=mysql=MYSQL

# The TCP/IP Port the MySQL Server will listen on
port=3306

# Path to installation directory. All paths are usually resolved relative to this.
# basedir="/usr/local/mysqlData/test/"

# Path to the database root 数据
datadir=/usr/local/mysqlData/test/data

# The default character set that will be used when a new schema or table is
# created and no character set is defined
# character-set-server=

# The default authentication plugin to be used when connecting to the server
default_authentication_plugin=caching_sha2_password

# The default storage engine that will be used when create new tables when
default-storage-engine=INNODB

# Set the SQL mode to strict
sql-mode="STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION"

# General and Slow logging.
log-output=FILE
general-log=0
general_log_file="test-mysql.log"
slow-query-log=1
slow_query_log_file="test-mysql-slow.log"
long_query_time=10

# Binary Logging.
# log-bin=

# Error Logging.
log-error="test-mysq.err"

# Server Id.
server-id=1

# Specifies the on how table names are stored in the metadata.
# If set to 0, will throw an error on case-insensitive operative systems
# If set to 1, table names are stored in lowercase on disk and comparisons are not case sensitive.
# If set to 2, table names are stored as given but compared in lowercase.
# This option also applies to database names and table aliases.
# NOTE: Modify this value after Server initialization won't take effect.
lower_case_table_names=1

# Secure File Priv.   注意该路径的配置
secure-file-priv=/var/lib/mysql

# The maximum amount of concurrent sessions the MySQL server will
# allow. One of these connections will be reserved for a user with
# SUPER privileges to allow the administrator to login even if the
# connection limit has been reached.
max_connections=151

# The number of open tables for all threads. Increasing this value
# increases the number of file descriptors that mysqld requires.
# Therefore you have to make sure to set the amount of open files
# allowed to at least 4096 in the variable "open-files-limit" in
# section [mysqld_safe]
table_open_cache=2000

# Maximum size for internal (in-memory) temporary tables. If a table
# grows larger than this value, it is automatically converted to disk
# based table This limitation is for a single table. There can be many
# of them.
tmp_table_size=16M

# How many threads we should keep in a cache for reuse. When a client
# disconnects, the client's threads are put in the cache if there aren't
# more than thread_cache_size threads from before.  This greatly reduces
# the amount of thread creations needed if you have a lot of new
# connections. (Normally this doesn't give a notable performance
# improvement if you have a good thread implementation.)
thread_cache_size=10

# *** MyISAM Specific options
# The maximum size of the temporary file MySQL is allowed to use while
# recreating the index (during REPAIR, ALTER TABLE or LOAD DATA INFILE.
# If the file-size would be bigger than this, the index will be created
# through the key cache (which is slower).
myisam_max_sort_file_size=100G

# If the temporary file used for fast index creation would be bigger
# than using the key cache by the amount specified here, then prefer the
# key cache method.  This is mainly used to force long character keys in
# large tables to use the slower key cache method to create the index.
myisam_sort_buffer_size=8M

# Size of the Key Buffer, used to cache index blocks for MyISAM tables.
# Do not set it larger than 30% of your available memory, as some memory
# is also required by the OS to cache rows. Even if you're not using
# MyISAM tables, you should still set it to 8-64M as it will also be
# used for internal temporary disk tables.
key_buffer_size=8M

# Size of the buffer used for doing full table scans of MyISAM tables.
# Allocated per thread, if a full scan is needed.
read_buffer_size=0

read_rnd_buffer_size=0

# *** INNODB Specific options ***
# innodb_data_home_dir=

# Use this option if you have a MySQL server with InnoDB support enabled
# but you do not plan to use it. This will save memory and disk space
# and speed up some things.
# skip-innodb=

# If set to 1, InnoDB will flush (fsync) the transaction logs to the
# disk at each commit, which offers full ACID behavior. If you are
# willing to compromise this safety, and you are running small
# transactions, you may set this to 0 or 2 to reduce disk I/O to the
# logs. Value 0 means that the log is only written to the log file and
# the log file flushed to disk approximately once per second. Value 2
# means the log is written to the log file at each commit, but the log
# file is only flushed to disk approximately once per second.
innodb_flush_log_at_trx_commit=1

# The size of the buffer InnoDB uses for buffering log data. As soon as
# it is full, InnoDB will have to flush it to disk. As it is flushed
# once per second anyway, it does not make sense to have it very large
# (even with long transactions).
innodb_log_buffer_size=1M

# InnoDB, unlike MyISAM, uses a buffer pool to cache both indexes and
# row data. The bigger you set this the less disk I/O is needed to
# access data in tables. On a dedicated database server you may set this
# parameter up to 80% of the machine physical memory size. Do not set it
# too large, though, because competition of the physical memory may
# cause paging in the operating system.  Note that on 32bit systems you
# might be limited to 2-3.5G of user level memory per process, so do not
# set it too high.
innodb_buffer_pool_size=8M

# Size of each log file in a log group. You should set the combined size
# of log files to about 25%-100% of your buffer pool size to avoid
# unneeded buffer pool flush activity on log file overwrite. However,
# note that a larger logfile size will increase the time needed for the
# recovery process.
innodb_log_file_size=48M

# Number of threads allowed inside the InnoDB kernel. The optimal value
# depends highly on the application, hardware as well as the OS
# scheduler properties. A too high value may lead to thread thrashing.
innodb_thread_concurrency=17

# The increment size (in MB) for extending the size of an auto-extend InnoDB system tablespace file when it becomes full.
innodb_autoextend_increment=64

# The number of regions that the InnoDB buffer pool is divided into.
# For systems with buffer pools in the multi-gigabyte range, dividing the buffer pool into separate instances can improve concurrency,
# by reducing contention as different threads read and write to cached pages.
innodb_buffer_pool_instances=8

# Determines the number of threads that can enter InnoDB concurrently.
innodb_concurrency_tickets=5000

# Specifies how long in milliseconds (ms) a block inserted into the old sublist must stay there after its first access before
# it can be moved to the new sublist.
innodb_old_blocks_time=1000

# It specifies the maximum number of .ibd files that MySQL can keep open at one time. The minimum value is 10.
innodb_open_files=300

# When this variable is enabled, InnoDB updates statistics during metadata statements.
innodb_stats_on_metadata=0

# When innodb_file_per_table is enabled (the default in 5.6.6 and higher), InnoDB stores the data and indexes for each newly created table
# in a separate .ibd file, rather than in the system tablespace.
innodb_file_per_table=1

# Use the following list of values: 0 for crc32, 1 for strict_crc32, 2 for innodb, 3 for strict_innodb, 4 for none, 5 for strict_none.
innodb_checksum_algorithm=0

# The number of outstanding connection requests MySQL can have.
# This option is useful when the main MySQL thread gets many connection requests in a very short time.
# It then takes some time (although very little) for the main thread to check the connection and start a new thread.
# The back_log value indicates how many requests can be stacked during this short time before MySQL momentarily
# stops answering new requests.
# You need to increase this only if you expect a large number of connections in a short period of time.
back_log=80

# If this is set to a nonzero value, all tables are closed every flush_time seconds to free up resources and
# synchronize unflushed data to disk.
# This option is best used only on systems with minimal resources.
flush_time=0

# The minimum size of the buffer that is used for plain index scans, range index scans, and joins that do not use
# indexes and thus perform full table scans.
join_buffer_size=256K

# The maximum size of one packet or any generated or intermediate string, or any parameter sent by the
# mysql_stmt_send_long_data() C API function.
max_allowed_packet=4M

# If more than this many successive connection requests from a host are interrupted without a successful connection,
# the server blocks that host from performing further connections.
max_connect_errors=100

# Changes the number of file descriptors available to mysqld.
# You should try increasing the value of this option if mysqld gives you the error "Too many open files".
open_files_limit=4161

# If you see many sort_merge_passes per second in SHOW GLOBAL STATUS output, you can consider increasing the
# sort_buffer_size value to speed up ORDER BY or GROUP BY operations that cannot be improved with query optimization
# or improved indexing.
sort_buffer_size=256K

# The number of table definitions (from .frm files) that can be stored in the definition cache.
# If you use a large number of tables, you can create a large table definition cache to speed up opening of tables.
# The table definition cache takes less space and does not use file descriptors, unlike the normal table cache.
# The minimum and default values are both 400.
table_definition_cache=1400

# Specify the maximum size of a row-based binary log event, in bytes.
# Rows are grouped into events smaller than this size if possible. The value should be a multiple of 256.
binlog_row_event_max_size=8K

# If the value of this variable is greater than 0, a replication slave synchronizes its master.info file to disk.
# (using fdatasync()) after every sync_master_info events.
sync_master_info=10000

# If the value of this variable is greater than 0, the MySQL server synchronizes its relay log to disk.
# (using fdatasync()) after every sync_relay_log writes to the relay log.
sync_relay_log=10000

# If the value of this variable is greater than 0, a replication slave synchronizes its relay-log.info file to disk.
# (using fdatasync()) after every sync_relay_log_info transactions.
sync_relay_log_info=10000

# Load mysql plugins at start."plugin_x ; plugin_y".
# plugin_load=

# The TCP/IP Port the MySQL Server X Protocol will listen on.
loose_mysqlx_port=33060

```

### 3.启动服务

```shell
# mysql 8 不推荐使用软链接

docker run --restart=always --privileged=true -itd -p 3307:3306 --name test_mysql -v /usr/local/mysqlData/test/conf/my.cnf:/etc/mysql/my.cnf -v /usr/local/mysqlData/test/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 mysql:8.0.11

# 参数说明：
# –restart=always： 当Docker 重启时，容器会自动启动。
# –privileged=true：容器内的root拥有真正root权限，否则容器内root只是外部普通用户权限
# -v /usr/local/mysqlData/test/log:/var/log/mysql 映射日志文件
# -v /usr/local/mysqlData/test/data:/var/lib/mysql 映射数据目录
# -v /usr/local/mysqlData/test/conf/my.cnf:/etc/mysql/my.cnf 映射配置文件
```

### 4. 配置远程连接

```shell
# 登录到test_mysql 容器
docker exec -it test_mysql bash

# 访问mysql (密码为初始密码)
myql -uroot -p

# 查询用户（可忽略）
select user,host,authentication_string from mysql.user;

# 设置权限（为root分配权限，以便可以远程连接）
update user set host = '%' where user = 'root';

grant all PRIVILEGES on *.* to root@'%' WITH GRANT OPTION;

# 由于Mysql5.6以上的版本修改了Password算法，这里需要更新密码算法，便于使用Navicat连接
grant all PRIVILEGES on *.* to root@'%' WITH GRANT OPTION;

# 更新密码算法（msyql 8.0 以上）
# ALTER user 'root'@'%' IDENTIFIED BY '123456' PASSWORD EXPIRE NEVER;
ALTER user 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456';

# 异常问题1：You must reset your password using ALTER USER statement before executing this statement

alter user 'root'@'localhost' identified by '123456'; #改密码方式一
alter user USER() identified by '123456'; 			   #改密码方式二

# 异常问题2： caching_sha2_password not be loaded:xxxx
 ALTER user 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '123456';

# 提交
FLUSH PRIVILEGES;

# 通过navicate 即可连接

```

### 二、基于安装包安装

```shell

# 安装发布的版本   archives(归档版本，无需再打包，解压、配置即可)
https://downloads.mysql.com/archives/community/

# 文件以 .tar.gz结尾

# 通过日志中查看mysql初始密码
```

```sql
select '自己有才是真的有';
```

### 参考文档

[Docker 部署 Mysql8.0](https://blog.csdn.net/xsj34567/article/details/80940238#comments_13664456)
