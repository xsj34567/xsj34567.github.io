---
layout: post
title:  "系统安全"
date:   2013-01-02 10:29:54
categories: windows/Linux/Mac
tags: windows Linux Mac
creattime: 2019-05-16 10:29:54
---

* content
{:toc}
系统安全：杀毒软件、基本安全防护、WEB安全防护等


## Linux 安全

杀毒软件、登录、注销、软件 软件更新等等

### 基本防护

更新软件、系统参数配置

#### 更新系统软件

(1)内核修复
```shell
yum update kernel
yum update kernel-devel
yum update kernel-headers
yum update kernel-tools
yum update kernel-tools-libs
yum update python-perf
```

(2)glibc 安全
```shell
yum update glibc
yum update glibc-common
yum update glibc-devel
yum update glibc-headers
yum update nscd
```

(3)bash 修复
```yum update bash```


(4)NetworkManager 和 libnl3 安全和BUG修复更新

```shell
yum update libnl3
yum update NetworkManager
yum update NetworkManager-libnm
yum update NetworkManager-tui
yum update NetworkManager-wifi
```

(5)wget 安全 
```yum update wget```

(6)procps-ng 安全更新
```yum update procps-ng ```

(7)systemd 安全更新
```shell
yum update libgudev1
yum update systemd
yum update systemd-libs
yum update systemd-sysv
```

(8)openssh 安全更新
```shell
yum update openssh
yum update openssh-clients
yum update openssh-server
```

(9)binutils 安全
```yum update binutils```


(10)sudo 安全和BUG修复更新
```yum update sudo```


#### 系统参数设置

(1)确保SSH MaxAuthTries设置为3到6之间 | 服务配置
```
描述
设置较低的Max AuthTrimes参数将降低SSH服务器被暴力攻击成功的风险。
加固建议
在/etc/ssh/sshd_config中取消MaxAuthTries注释符号，设置最大密码尝试失败次数3-6，建议为4：MaxAuthTries 4
```


(2) 密码复杂度检查 | 身份鉴别
```
加固建议
编辑/etc/security/pwquality.conf，把minlen（密码最小长度）设置为9-32位，把minclass
（至少包含小写字母、大写字母、数字、特殊字符等4类字符中等3类或4类）设置为3或4。如： minlen=10 minclass=3
```

(3) SSHD强制使用V2安全协议 | 服务配置
```
SSHD强制使用V2安全协议
编辑 /etc/ssh/sshd_config 文件以按如下方式设置参数： Protocol 2
```

(4) 设置SSH空闲超时退出时间 | 服务配置
```
加固建议
编辑/etc/ssh/sshd_config，将ClientAliveInterval 设置为300到900，即5-15分钟，将ClientAliveCountMax设置为0-3。
ClientAliveCountMax 2
```

(5) 确保SSH LogLevel设置为INFO | 服务配置
```
描述
确保SSH LogLevel设置为INFO,记录登录和注销活动
加固建议
编辑 /etc/ssh/sshd_config 文件以按如下方式设置参数(取消注释): LogLevel INFO
```

#### [系统操作监控配置](https://www.landui.com/help/show-2714.html)

```
  *** 可在 /tmp/dbasky/root 目录下查看 IP+时间段 操作记录 
```

#### [clamav完整查杀linux病毒实战](https://www.cnblogs.com/k98091518/p/6895078.html)

#### [阿里云mail发送邮件](https://blog.csdn.net/cbuy888/article/details/88287883) 163、126、QQ邮箱类似

```
  可以将clamav查询的结果（其他任务的结果），发送到邮箱
```



参考：

阿里云标准-CentOS Linux 7安全基线检查

[CentOS通过日志反查入侵](https://www.landui.com/help/show-2714.html)

[clamav完整查杀linux病毒实战](https://www.cnblogs.com/k98091518/p/6895078.html)

[yum 命令](https://www.runoob.com/linux/linux-yum.html) 检查软件待更新清单、更新、移除实例

[阿里云mail发送邮件](https://blog.csdn.net/cbuy888/article/details/88287883)

[linux下使用自带mail发送邮件](https://blog.csdn.net/qq_39680564/article/details/82746812)