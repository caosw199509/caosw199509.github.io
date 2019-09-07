---
layout:     post
title:      BenchmarkSQL5.0数据库TPC-C基准测试
subtitle:   BenchmarkSQL是一个同时支持Oracle、MySQL等关系型数据库的基准测试工具，通过使用BenchmarkSQL对数据库进行TPC-C标准测试，即模拟多种事务处理：新订单、支付操作、订单状态查询、发货、库存状态查询等，从而获得最终的TpmC值。
author:     caosw
header-img: img/BenchmarkSQL.jpg
catalog: true
tags:
    - Oracle
    - BenchmarkSQL
---

# BenchmarkSQL5.0数据库TPC-C基准测试
***
### 1、TPC-C介绍    

TPC（Transaction Processing-Performance Council）是事务处理性能委员会的缩写。该组织的主要功能是指定商用应用基准程序(Benchmark)的标准规范、性能和价格度量，并管理测试结果的发布。
<br>其中TPC-C是在线事务处理（OLTP）的基准程序。TPC-C 会模拟一个批发商的货物管理环境，旨在模拟仓储、订单、配送等OLTP业务的过程。通过模拟这套复杂的业务流程，收集测试中的指标，从而得出最终的TpmC值，即每分钟可处理的事务数。

### 2、BenchmarkSQL工具介绍

BenchmarkSQL是一个同时支持Oracle、MySQL等关系型数据库的基准测试工具，通过使用BenchmarkSQL对数据库进行TPC-C标准测试，即模拟多种事务处理：新订单、支付操作、订单状态查询、发货、库存状态查询等，从而获得最终的TpmC值。

### 3、BenchmarkSQL安装
##### 3.1、配置Java环境变量

    [root@localhost opt]# tar zxvf jdk-8u65-linux-x64.tar.gz
    [root@localhost opt]# ln -s /opt/jdk1.8.0_65/ /usr/local/java
    [root@localhost opt]# vi /etc/profile
    export JAVA_HOME=/usr/local/java
    export PATH=$PATH:$JAVA_HOME/bin
    [root@localhost opt]# source /etc/profile
    [root@localhost opt]# java -version
    openjdk version "1.8.0_222"
    OpenJDK Runtime Environment (build 1.8.0_222-b10)
    OpenJDK 64-Bit Server VM (build 25.222-b10, mixed mode

##### 3.2、安装BenchmarkSQL并编译

    [root@localhost opt]# unzip benchmarksql-5.0.zip
    [root@localhost opt]# yum install -y ant
    [root@localhost opt]# cd /opt/benchmarksql-5.0
    [root@localhost benchmarksql-5.0]# ant

##### 3.3、添加jdbc驱动jar包

将需要压测的数据库的驱动jar包放置benchmarksql-5.0的lib目录下

### 4、实际测试
`这里的测试环境是某项目上的数据库服务器，测试了国产数据HighGo4.3.2和Oracle11.2.0.4的情况对比。在实际测试过程中例如测试HighGo的时候会将Oracle的服务停掉，同理测试Oracle的时候会将HighGo的服务停掉。另外某些配置参数需要调成相近，例如HighGo中的shared_buffers参数为设置内存占用参数调成物理内存的70%，Oracle的SGA大小也要调成70%占用。最后这种压测也只是基准测试，不能完全代表实际项目中的情况。`
##### 4.1、服务器硬件配置
    操作系统： CentOS Linux release 7.6.1810 (Core)
    CPU： Intel(R) Xeon(R) CPU E7-4820 v2 @ 2.00GHz 64core
    内存：128G
    硬盘：1T

##### 4.2、数据版本以及压测工具版本
    Oracle版本：11.2.0.4
    HighGo版本：4.3.2

##### 4.3、props文件编写

jdbc连接部分改成对应的数据库的JDBC连接，这里以Oracle为例:
<br>创建文件props.ora文件，文件存放在run目录下。
<br>其中压测的所连接的用户或者库需要手动创建。

    db=oracle
    driver=oracle.jdbc.driver.OracleDriver
    conn=jdbc:oracle:thin:@localhost:1521:orcl
    user=benchmarksql
    password=11111

    // 设定仓库数
    warehouses=20
    loadWorkers=4

    // 并发数
    terminals=64

    runTxnsPerTerminal=0
    // 压测时间
    runMins=10
    //每分钟的事务
    limitTxnsPerMin=0

    //The following five values must add up to 100
    newOrderWeight=45
    paymentWeight=43
    orderStatusWeight=4
    deliveryWeight=4
    stockLevelWeight=4

    // Directory name to create for collecting detailed result data.
    // Comment this out to suppress.
    resultDirectory=my_result_%tY-%tm-%td_%tH%tM%tS
    osCollectorScript=./misc/os_collector_linux.py
    osCollectorInterval=1
    //osCollectorSSHAddr=user@dbhost
    //指定网卡名称以及磁盘
    osCollectorDevices=net_eth0 blk_sda

##### 4.4、准备测试数据

需要执行部分脚本构建测试数据

    [root@localhost run]# ./runSQL.sh props.ora ./sql.common/tableCreates.sql
    [root@localhost run]# ./runLoader.sh props.ora numWAREHOUSES 20
    [root@localhost run]# ./runSQL.sh props.ora ./sql.common/indexCreates.sql

##### 4.5、压测拓扑

![](https://github.com/caosw199509/caosw199509.github.io/blob/master/work_img/2019-09-06/benchmarksql.png?raw=true)

##### 4.6、压测描述

按照BenchmarkSQL官方文档中描述，其测试并发压力一般为CPU个数的2-8倍，故挑选了以下三种测试用例。
    <br>用例一：测试HighGo、Oracle在BenchmarkSQL工具64线程20仓库下的TpmC；
    <br>用例二：测试HighGo、Oracle在BenchmarkSQL工具128线程20仓库下的TpmC；
    <br>用例三：测试HighGo、Oracle在BenchmarkSQL工具64线程40仓库下的TpmC；
##### 4.7、用例一测试结果
###### 4.7.1、HighGo

![](https://github.com/caosw199509/caosw199509.github.io/blob/master/work_img/2019-09-06/highgo01.png?raw=true)

在20仓64并发终端下，HighGo数据库Tpmc值为96514.2

###### 4.7.2、Oracle

![](https://github.com/caosw199509/caosw199509.github.io/blob/master/work_img/2019-09-06/oracle01.png?raw=true)

在20仓64并发终端下，Oracle数据库Tpmc值为84668.78

##### 4.8、用例二测试结果
###### 4.8.1、HighGo

![](https://github.com/caosw199509/caosw199509.github.io/blob/master/work_img/2019-09-06/highgo02.png?raw=true)

在20仓128并发终端下，HighGo数据库Tpmc值为90814.89

###### 4.8.2、Oracle

![](https://github.com/caosw199509/caosw199509.github.io/blob/master/work_img/2019-09-06/oracle02.png?raw=true)

在20仓128并发终端下，Oracle数据库Tpmc值为108959.27

##### 4.9、用例三测试结果
###### 4.9.1、HighGo

![](https://github.com/caosw199509/caosw199509.github.io/blob/master/work_img/2019-09-06/highgo03.png?raw=true)

在40仓64并发终端下，HighGo数据库Tpmc值为61125.29

###### 4.9.2、Oracle

![](https://github.com/caosw199509/caosw199509.github.io/blob/master/work_img/2019-09-06/oracle03.png?raw=true)

在40仓64并发终端下，Oracle数据库Tpmc值为64919.13


##### 4.10、清楚测试环境
同样的有脚本可以进行清除

    [root@localhost run]# ./runDatabaseDestroy.sh props.ora