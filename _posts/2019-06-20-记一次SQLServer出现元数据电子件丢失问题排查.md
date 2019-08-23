---
layout:     post
title:      记一次SQLServer出现元数据电子件丢失问题排查
subtitle:   项目上出现了数据丢失，但是确认了非人为操作，需要进一步排查到底哪里出现的删除操作，具体排查步骤和过程。
date:       2019-06-20
author:     caosw
header-img: img/home-bg-o.jpg
catalog: true
tags:
    - SQLServer
---

# 记一次SQLServer出现元数据电子件丢失问题排查
***
### 背景：
某项目业务系统于6月19日下午4点出现元数据电子件被删除，导致业务系统附件信息中部分电子附件不显示。对于一家网络公司而言丢数据是算是一种灾难，尽管项目组已经通过之前的备份进行了还原处理，但是还是需要深入排查到数据丢失的具体原因。

### 操作系统以及数据库版本：
    windows Server 2008R2
    SQLServer2008R2

### 排查过程：
1、通过ApexSQLlog工具查询6月19日12点至4点的所有和表ZHGL_ScanFileInfo相关的删除操作，并查找相关的SPID信息。

![](https://github.com/caosw199509/caosw199509.github.io/blob/master/work_img/2019-06-20/1.png?raw=true)

2、通过SPID的值查询到执行的机器地址以及对应的端口号。

![](https://github.com/caosw199509/caosw199509.github.io/blob/master/work_img/2019-06-20/2.png?raw=true)

3、远程登陆至查询到的机器并更具查询到的端口号查询具体执行的应用程序。

![](https://github.com/caosw199509/caosw199509.github.io/blob/master/work_img/2019-06-20/3.png?raw=true)

### 总结：
总的来说这次算是比较幸运的，能够通过SPID查询到实际操作地址和端口号，并且应用程序还没有断开，如果应用程序断开了，可能真的就死无对证了。