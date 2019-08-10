---
layout:     post
title:      Oracle11g Data Guard 搭建
subtitle:   Oracle11g Data Guard 搭建
date:       2018-12-25
author:     caosw
header-img: img/post-bg-debug.jpg
catalog: true
tags:
    - Oracle
    - DG
---

# Oracle11g Data Guard 搭建

### 1、 环境说明
系统环境以及hosts配置

    [oracle@primary ~]$ cat /etc/redhat-release 
	CentOS Linux release 7.4.1708 (Core)
	[oracle@primary ~]$ cat /etc/hosts
	127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
	::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
	192.168.1.20 primary
	192.168.1.21 standby
oracle版本信息

    SQL> select * from v$version;
    BANNER
    --------------------------------------------------------------------------------
	Oracle Database 11g Enterprise Edition Release 11.2.0.4.0 - 64bit Production
	PL/SQL Release 11.2.0.4.0 - Production
	CORE    11.2.0.4.0      Production
	TNS for Linux: Version 11.2.0.4.0 - Production
	NLSRTL Version 11.2.0.4.0 - Production

### 2、 主库启动force logging

    SQL> alter database force logging;
	Database altered.
	SQL> select force_logging from v$database;
	FORCE_LOGGING
	------------------------------
	YES

###3、 启动数据库归档模式
创建存放归档日志的目录

    [oracle@primary ~]$ mkdir -p /u01/archive
开启归档模式

    --查看数据库是否已开启归档
    SQL> archive log list
    Database log mode              No Archive Mode
    Automatic archival             Disabled
    Archive destination            USE_DB_RECOVERY_FILE_DEST
    Oldest online log sequence     5
    Current log sequence           7
    --确认未开启归档，关闭数据库
    SQL> shutdown immediate
    Database closed.
    Database dismounted.
    ORACLE instance shut down.
    --将数据库启动到mount模式
    SQL> startup mount
    ORACLE instance started.
    Total System Global Area  972898304 bytes
    Fixed Size                  2259160 bytes
    Variable Size             616564520 bytes
    Database Buffers          348127232 bytes
    Redo Buffers                5947392 bytes
    Database mounted.
    --开启归档，设置归档路径并打开数据库
    SQL> alter database archivelog;
    Database altered.
    SQL> alter system set log_archive_dest_1='location=/u01/archive/' scope=spfile;
    System altered.
    SQL> alter database open;
    Database altered.
    --验证归档开启
    SQL> archive log list
    Database log mode              Archive Mode
    Automatic archival             Enabled
    Archive destination            /u01/archive/
    Oldest online log sequence     5
    Next log sequence to archive   7
    Current log sequence           7 

### 4、 在主库中添加standby redo logfile
查看primary库中的日志信息

    SQL> select group#,members,bytes/1024/1024 from v$log;
    GROUP#    MEMBERS BYTES/1024/1024
    ---------- ---------- ---------------
         1          1              50
         2          1              50
         3          1              50
    SQL> select member from v$logfile;
    MEMBER
    --------------------------------------------------
    /u01/app/oracle/oradata/orcl/redo03.log
    /u01/app/oracle/oradata/orcl/redo02.log
    /u01/app/oracle/oradata/orcl/redo01.log
添加standby redo logfile

    SQL> alter database add standby logfile '/u01/app/oracle/oradata/orcl/stdredo01.log' size 200M;
    Database altered.
    SQL> alter database add standby logfile '/u01/app/oracle/oradata/orcl/stdredo02.log' size 200M;
    Database altered.
    SQL> alter database add standby logfile '/u01/app/oracle/oradata/orcl/stdredo03.log' size 200M;
    Database altered.
    SQL> alter database add standby logfile '/u01/app/oracle/oradata/orcl/stdredo04.log' size 200M;
    Database altered.
再次查看日志信息

    SQL> select member from v$logfile;
    MEMBER
    --------------------------------------------------
    /u01/app/oracle/oradata/orcl/redo03.log
    /u01/app/oracle/oradata/orcl/redo02.log
    /u01/app/oracle/oradata/orcl/redo01.log
    /u01/app/oracle/oradata/orcl/stdredo01.log
    /u01/app/oracle/oradata/orcl/stdredo02.log
    /u01/app/oracle/oradata/orcl/stdredo03.log
    /u01/app/oracle/oradata/orcl/stdredo04.log
    7 rows selected.

### 5、 分别在主备库配置监听文件并启动
配置监听文件

    SID_LIST_LISTENER =
      (SID_LIST =
        (SID_DESC =
          (GLOBAL_DBNAME = orcl)
          (ORACLE_HOME = /u01/app/oracle/product/11.2.0/db_1)
          (SID_NAME = orcl)
        )
      )
    LISTENER =
      (DESCRIPTION_LIST =
        (DESCRIPTION =
          (ADDRESS = (PROTOCOL = IPC)(KEY = EXTPROC1521))
          (ADDRESS = (PROTOCOL = TCP)(HOST = primary)(PORT = 1521))
        )
      )
    ADR_BASE_LISTENER = /u01/app/oracle
重启监听

    [oracle@primary admin]$ lsnrctl reload
    LSNRCTL for Linux: Version 11.2.0.4.0 - Production on 11-JAN-2019 23:53:24
    Copyright (c) 1991, 2013, Oracle.  All rights reserved.
    Connecting to (DESCRIPTION=(ADDRESS=(PROTOCOL=IPC)(KEY=EXTPROC1521)))
    The command completed successfully

### 6、 分别在主备库上配置tnsname.ora
配置tnsname.ora

    primary =
      (DESCRIPTION =
        (ADDRESS = (PROTOCOL = TCP)(HOST = 192.168.1.20)(PORT = 1521))
        (CONNECT_DATA =
          (SERVER = DEDICATED)
          (SERVICE_NAME = orcl)
        )
     )
    standby =
      (DESCRIPTION =
        (ADDRESS = (PROTOCOL = TCP)(HOST = 192.168.1.21)(PORT = 1521))
        (CONNECT_DATA =
          (SERVER = DEDICATED)
          (SERVICE_NAME = orcl)
        )
      )
将primary库上的tnsname.ora文件复制到standby库上

    [oracle@primary admin]$ scp tnsnames.ora oracle@standby:/u01/app/oracle/product/11.2.0/db_1/network/admin/
    oracle@standby's password: 
    tnsnames.ora                                     100%  682   592.0KB/s   00:00 
配置完成后，通过tnsping命令进行校验，需要项目ping通对方才行。如果ping不通，检查是否关闭防火墙或者配置是不是存在问题

    [oracle@primary ~]$ tnsping standby
    TNS Ping Utility for Linux: Version 11.2.0.4.0 - Production on 12-JAN-2019 00:06:07
    Copyright (c) 1997, 2013, Oracle.  All rights reserved.
    Used parameter files:
    /u01/app/oracle/product/11.2.0/db_1/network/admin/sqlnet.ora
    Used TNSNAMES adapter to resolve the alias
    Attempting to contact (DESCRIPTION = (ADDRESS = (PROTOCOL = TCP)(HOST = 192.168.1.21)(PORT = 1521))(CONNECT_DATA = (SERVER = DEDICATED) (SERVICE_NAME = orcl)))OK (0 msec)
    [oracle@standby ~]$ tnsping primary
    TNS Ping Utility for Linux: Version 11.2.0.4.0 - Production on 06-JAN-2019 22:27:44
    Copyright (c) 1997, 2013, Oracle.  All rights reserved.
    Used parameter files:
    /u01/app/oracle/product/11.2.0/db_1/network/admin/sqlnet.ora
    Used TNSNAMES adapter to resolve the alias
    Attempting to contact (DESCRIPTION = (ADDRESS = (PROTOCOL = TCP)(HOST = 192.168.1.20)(PORT = 1521)) (CONNECT_DATA = (SERVER = DEDICATED) (SERVICE_NAME = orcl)))OK (10 msec)

### 7、 在备库上创建必要的目录

    [oracle@standby ~]$ mkdir -p /u01/archive
    [oracle@standby ~]$ mkdir -p /u01/app/oracle/admin/orcl/adump
    [oracle@standby ~]$ mkdir -p /u01/app/oracle/fast_recovery_area/orcl
    [oracle@standby ~]$ mkdir -p /u01/app/oracle/oradata/orcl

### 8、 在主库上创建pfile文件并修改pfile内容
通过spfile创建pfile文件

    SQL> create pfile from spfile;
    File created.
pfile文件添加内容

    [oracle@primary dbs]$ vi initorcl.ora 
    *.db_unique_name='primary'
    *.log_archive_config='dg_config=(primary,standby)'
    *.log_archive_dest_1='location=/u01/archive/ valid_for=(all_logfiles,all_roles) db_unique_name=primary'
    *.log_archive_dest_2='service=standby lgwr affirm sync valid_for=(online_logfiles,primary_role) db_unique_name=standby'
    *.log_archive_dest_state_1=enabl
    *.log_archive_dest_state_2=enable
    *.standby_file_management='auto'
    *.fal_server='standby'
    *.log_file_name_convert='/u01/app/oracle/oradata/orcl','/u01/app/oracle/oradata/orcl'
    *.db_file_name_convert='/u01/app/oracle/oradata/orcl','/u01/app/oracle/oradata/orcl'
使用新的参数重启数据库

    SQL> create spfile from pfile;
    File created.
    SQL> startup
    ORACLE instance started.
    Total System Global Area  972898304 bytes
    Fixed Size                  2259160 bytes
    Variable Size             616564520 bytes
    Database Buffers          348127232 bytes
    Redo Buffers                5947392 bytes
    Database mounted.
    Database opened.

### 9、 将主库的口令文件复制到备库中

    [oracle@primary dbs]$ scp orapworcl oracle@standby:$ORACLE_HOME/dbs
    oracle@standby's password: 
    orapworcl                                           100% 1536    31.3KB/s   00:00

### 10、将主库的参数文件复制到备库中并修改
复制initorcl.ora文件到备库中

    [oracle@primary dbs]$ scp initorcl.ora oracle@standby:$ORACLE_HOME/dbs 
    oracle@standby's password: 
    initorcl.ora                                       100% 1545   983.4KB/s   00:00 
修改备库中的参数文件

    [oracle@standby dbs]$ vi initorcl.ora 
    *.db_unique_name='standby'
    *.log_archive_config='dg_config=(primary,standby)'
    *.log_archive_dest_1='location=/u01/archive/ valid_for=(all_logfiles,all_roles) db_unique_name=standby'
    *.log_archive_dest_2='service=standby lgwr affirm sync valid_for=(online_logfiles,primary_role)
    db_unique_name=primary'
    *.log_archive_dest_state_1=enable
    *.log_archive_dest_state_2=enable
    *.standby_file_management='auto'
    *.fal_server='primary'
    *.log_file_name_convert='/u01/app/oracle/oradata/orcl','/u01/app/oracle/oradata/orcl'
    *.db_file_name_convert='/u01/app/oracle/oradata/orcl','/u01/app/oracle/oradata/orcl'
通过pfile文件生成spfile并将standby库启动到nomount状态

    SQL> create spfile from pfile;
    File created.
    SQL> startup nomount;
    ORACLE instance started.
    Total System Global Area  972898304 bytes
    Fixed Size                  2259160 bytes
    Variable Size             616564520 bytes
    Database Buffers          348127232 bytes
    Redo Buffers                5947392 bytes
重启监听

### 11、在主库上开始进行duplicate achive
需要创建好所有需要的目录
 

    [oracle@primary ~]$ rman target / auxiliary sys/oracle@standby
    Recovery Manager: Release 11.2.0.4.0 - Production on Sun Jan 13 00:01:44 2019
    Copyright (c) 1982, 2011, Oracle and/or its affiliates.  All rights reserved.
    connected to target database: ORCL (DBID=1524649093)
    connected to auxiliary database: ORCL (not mounted)
开始进行复制

    RMAN> duplicate target database for standby from active database nofilenamecheck dorecover;
    Starting Duplicate Db at 2019/01/13 00:03:25
    using target database control file instead of recovery catalog
    allocated channel: ORA_AUX_DISK_1
    channel ORA_AUX_DISK_1: SID=134 device type=DIS
    contents of Memory Script:
    {
      backup as copy reuse
      targetfile  '/u01/app/oracle/product/11.2.0/db_1/dbs/orapworcl' auxiliary format
    '/u01/app/oracle/product/11.2.0/db_1/dbs/orapworcl'   ;
    }
    executing Memory Script
    Starting backup at 2019/01/13 00:03:25
    allocated channel: ORA_DISK_1
    channel ORA_DISK_1: SID=145 device type=DISK
    Finished backup at 2019/01/13 00:03:27
    contents of Memory Script:
    {
      backup as copy current controlfile for standby auxiliary format  '/u01/app/oracle/oradata/orcl/control01.ctl';
      restore clone controlfile to  '/u01/app/oracle/fast_recovery_area/orcl/control02.ctl' from '/u01/app/oracle/oradata/orcl/control01.ctl';
    }
    executing Memory Script
    Starting backup at 2019/01/13 00:03:27
    using channel ORA_DISK_1
    channel ORA_DISK_1: starting datafile copy
    copying standby control file
    output file name=/u01/app/oracle/product/11.2.0/db_1/dbs/snapcf_orcl.f tag=TAG20190113T000327 RECID=1 STAMP=997401808
    channel ORA_DISK_1: datafile copy complete, elapsed time: 00:00:03
    Finished backup at 2019/01/13 00:03:30
    Starting restore at 2019/01/13 00:03:30
    using channel ORA_AUX_DISK_1
    channel ORA_AUX_DISK_1: copied control file copy
    Finished restore at 2019/01/13 00:03:31
    contents of Memory Script:
    {
      sql clone 'alter database mount standby database';
    }
    executing Memory Script
    sql statement: alter database mount standby database
    contents of Memory Script:
    {
    set newname for tempfile  1 to "/u01/app/oracle/oradata/orcl/temp01.dbf";
    switch clone tempfile all;
    set newname for datafile  1 to "/u01/app/oracle/oradata/orcl/system01.dbf";
    set newname for datafile  2 to "/u01/app/oracle/oradata/orcl/sysaux01.dbf";
    set newname for datafile  3 to "/u01/app/oracle/oradata/orcl/undotbs01.dbf";
    set newname for datafile  4 to "/u01/app/oracle/oradata/orcl/users01.dbf";
    backup as copy reuse datafile  1 auxiliary format 
    "/u01/app/oracle/oradata/orcl/system01.dbf"   datafile 
    2 auxiliary format 
    "/u01/app/oracle/oradata/orcl/sysaux01.dbf"   datafile 
    3 auxiliary format 
    "/u01/app/oracle/oradata/orcl/undotbs01.dbf"   datafile 
    4 auxiliary format "/u01/app/oracle/oradata/orcl/users01.dbf"   ;
    sql 'alter system archive log current';
    }
    executing Memory Script
    executing command: SET NEWNAME
    renamed tempfile 1 to /u01/app/oracle/oradata/orcl/temp01.dbf in control file
    executing command: SET NEWNAME
    executing command: SET NEWNAME
    executing command: SET NEWNAME
    executing command: SET NEWNAME
    Starting backup at 2019/01/13 00:03:39
    using channel ORA_DISK_1
    channel ORA_DISK_1: starting datafile copy
    input datafile file number=00001 name=/u01/app/oracle/oradata/orcl/system01.dbf
    output file name=/u01/app/oracle/oradata/orcl/system01.dbf tag=TAG20190113T000339
    channel ORA_DISK_1: datafile copy complete, elapsed time: 00:00:35
    channel ORA_DISK_1: starting datafile copy
    input datafile file number=00002 name=/u01/app/oracle/oradata/orcl/sysaux01.dbf
    output file name=/u01/app/oracle/oradata/orcl/sysaux01.dbf tag=TAG20190113T000339
    channel ORA_DISK_1: datafile copy complete, elapsed time: 00:00:25
    channel ORA_DISK_1: starting datafile copy
    input datafile file number=00003 name=/u01/app/oracle/oradata/orcl/undotbs01.dbf
    output file name=/u01/app/oracle/oradata/orcl/undotbs01.dbf tag=TAG20190113T000339
    channel ORA_DISK_1: datafile copy complete, elapsed time: 00:00:03
    channel ORA_DISK_1: starting datafile copy
    input datafile file number=00004 name=/u01/app/oracle/oradata/orcl/users01.dbf
    output file name=/u01/app/oracle/oradata/orcl/users01.dbf tag=TAG20190113T000339
    channel ORA_DISK_1: datafile copy complete, elapsed time: 00:00:01
    Finished backup at 2019/01/13 00:04:44
    sql statement: alter system archive log current
    contents of Memory Script:
    {
    backup as copy reuse
    archivelog like  "/u01/archive/1_10_996868360.dbf" auxiliary format 
    "/u01/archive/1_10_996868360.dbf"   archivelog like 
    "/u01/archive/1_11_996868360.dbf" auxiliary format 
    "/u01/archive/1_11_996868360.dbf"   ;
    catalog clone archivelog  "/u01/archive/1_10_996868360.dbf";
    catalog clone archivelog  "/u01/archive/1_11_996868360.dbf";
    switch clone datafile all;
    }
    executing Memory Script
    Starting backup at 2019/01/13 00:04:44
    using channel ORA_DISK_1
    channel ORA_DISK_1: starting archived log copy
    input archived log thread=1 sequence=10 RECID=4 STAMP=997401823
    output file name=/u01/archive/1_10_996868360.dbf RECID=0 STAMP=0
    channel ORA_DISK_1: archived log copy complete, elapsed time: 00:00:01
    channel ORA_DISK_1: starting archived log copy
    input archived log thread=1 sequence=11 RECID=5 STAMP=997401884
    output file name=/u01/archive/1_11_996868360.dbf RECID=0 STAMP=0
    channel ORA_DISK_1: archived log copy complete, elapsed time: 00:00:01
    Finished backup at 2019/01/13 00:04:47
    cataloged archived log
    archived log file name=/u01/archive/1_10_996868360.dbf RECID=1 STAMP=996971170
    cataloged archived log
    archived log file name=/u01/archive/1_11_996868360.dbf RECID=2 STAMP=996971171
    datafile 1 switched to datafile copy
    input datafile copy RECID=1 STAMP=996971171 file name=/u01/app/oracle/oradata/orcl/system01.dbf
    datafile 2 switched to datafile copy
    input datafile copy RECID=2 STAMP=996971171 file name=/u01/app/oracle/oradata/orcl/sysaux01.dbf
    datafile 3 switched to datafile copy
    input datafile copy RECID=3 STAMP=996971171 file name=/u01/app/oracle/oradata/orcl/undotbs01.dbf
    datafile 4 switched to datafile copy
    input datafile copy RECID=4 STAMP=996971171 file name=/u01/app/oracle/oradata/orcl/users01.dbf
    contents of Memory Script:
    {
    set until scn  1053384;
    recover
    standby
    clone database
    delete archivelog;
    }
    executing Memory Script
    executing command: SET until clause
    Starting recover at 2019/01/13 00:04:47
    using channel ORA_AUX_DISK_1
    starting media recovery
    archived log for thread 1 with sequence 10 is already on disk as file /u01/archive/1_10_996868360.dbf
    archived log for thread 1 with sequence 11 is already on disk as file /u01/archive/1_11_996868360.dbf
    archived log file name=/u01/archive/1_10_996868360.dbf thread=1 sequence=10
    archived log file name=/u01/archive/1_11_996868360.dbf thread=1 sequence=11
    media recovery complete, elapsed time: 00:00:01
    Finished recover at 2019/01/13 00:04:48Finished Duplicate Db at 2019/01/13 00:05:06

### 12、打开备份并启动apply
打开备库

    SQL> select open_mode from v$database;
    OPEN_MODE
    --------------------
    MOUNTED
    SQL> alter database open;
    Database altered.
查看主库

    SQL> select log_mode,open_mode ,database_role from v$database;
    LOG_MODE     OPEN_MODE            DATABASE_ROLE
    ------------ -------------------- ----------------
    ARCHIVELOG   READ WRITE           PRIMARY
查看备库

    SQL> select log_mode,open_mode ,database_role from v$database;
    LOG_MODE     OPEN_MODE            DATABASE_ROLE
    ------------ -------------------- ----------------
    ARCHIVELOG   READ ONLY            PHYSICAL STANDBY
启动real-time apply

    SQL>  alter database recover managed standby database using current logfile disconnect from session;
    Database altered.
    SQL> select open_mode from v$database;
    OPEN_MODE
    --------------------
    READ ONLY WITH APPLY

### 13、验证DG
在主库上创建表test

    SQL> create table test as select * from dba_users;
    Table created.
    SQL> alter system switch logfile;
    System altered.
查看备库中是否存在表test

    SQL> select count(*) from test;
      COUNT(*)
    ----------
        30