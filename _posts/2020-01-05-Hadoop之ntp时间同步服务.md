---
layout: post
title: "Hadoop之ntp时间同步服务"
date: 2020-01-05
categories: 大数据 Hadoop NTP
tags: 大数据
updateTime:2021-04-19
---

* content
{:toc}
## Hadoop集群

熟悉时间同步服务，xxl-job 等调度时可能会出现异常。

### 一、ntp 时间同步

【2019-12-31】

说明：

    1. 以master为主 （master是按照网络时间为主： ntp.org）
    2. slave1、slave2 是以master时间为准

#### 1.master节点

    sudo yum install ntp 
    
    sudo systemctl start ntpd
    
    sudo systemctl status ntpd

#### 2.从节点

    1.修改ntp.conf配置
    
    sudo vi /etc/ntp.conf
    
    2.把配置文件中默认的配置注释掉
    
    #server 0.centos.pool.ntp.org iburst
    #server 1.centos.pool.ntp.org iburst
    #server 2.centos.pool.ntp.org iburst
    #server 3.centos.pool.ntp.org iburst
    
    3.在ntp.conf 新增配置 (以master时间为准)
    
    server master iburst
    
    4.重启ntp服务
    
    sduo systemctl start ntpd

#### 3.查看时间同步结果(在salve1、slave2中)

    watch ntpq -p 