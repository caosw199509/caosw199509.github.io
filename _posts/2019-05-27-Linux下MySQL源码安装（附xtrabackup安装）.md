---
layout:     post
title:      Linux下MySQL源码安装（附xtrabackup安装）
subtitle:   Linux服务器上可以通过rpm包直接安装MySQL，但是通过源码包安装可以手动配置一些参数，更利于MySQL的理解。
date:       2019-05-27
author:     caosw
header-img: img/post-bg-os-metro.jpg
catalog: true
tags:
    - MySQL
---

# Linux下MySQL源码安装(附xtrabackup安装)
***
环境说明：
Centos7.4 最小化安装

### 1、创建mysql用户和组
运行MySQL的mysqld需要一个用户和组

    [root@mysql ~]# groupadd mysql
    [root@mysql ~]# useradd -r -g mysql -s /bin/false mysql

### 2、创建数据目录
MySQL安装需要初始化数据目录用于存放MySQL数据库的数据
该目录可以个人自行设置
同时需要对设置该目录的用户和组

    [root@mysql ~]# mkdir -p /data/mysql_data
    [root@mysql ~]# chown -R mysql:mysql /data/mysql_data/

### 3、解压源码包创建软链接
将源码包上传至服务器的/opt目录下并解压
在/usr/local/目录下创建相应的软链接

    [root@mysql opt]# tar zxvf mysql-5.7.24-linux-glibc2.12-x86_64.tar.gz
    [root@mysql opt]# ln -s /opt/mysql-5.7.24-linux-glibc2.12-x86_64 /usr/local/mysql

### 4、配置环境变量
修改/etc/profile文件
添加mysql相关的环境变量

    [root@mysql opt]# vi /etc/profile
    export PATH=/usr/local/mysql/bin:$PATH
    export LD_LIBRARY_PATH=/usr/local/mysql/lib:$LD_LIBRARY_PATH
    [root@mysql opt]# source /etc/profile

### 5、配置数据库参数文件my.cnf
参数文件存放在/etc目录下
参数文件中的datadir具体看个人创建的数据目录
其他参数也可看需求进行更改

    [root@mysql etc]# vi /etc/my.cnf
    [client]
    default-character-set = utf8mb4
    [mysql]
    default-character-set = utf8mb4
    [mysqld]
    ########basic settings########
    server-id = 11port = 3306
    user = mysql
    bind_address = 0.0.0.0
    #skip_name_resolve
    autocommit = 1  
    character_set_server=utf8mb4
    datadir = /data/mysql_data
    transaction_isolation = READ-COMMITTED
    explicit_defaults_for_timestamp = 1
    tmpdir = /tmp
    max_allowed_packet = 1073741824
    event_scheduler = 1
    sql_mode = "STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER"
    # connection #
    interactive_timeout = 1800
    wait_timeout = 900
    lock_wait_timeout = 1800
    skip_name_resolve = 1
    max_connections = 2500
    max_connect_errors = 1000000
    # table cache performance settings #
    table_open_cache = 4096
    table_definition_cache = 4096
    table_open_cache_instances = 128
    # session memory settings #
    read_buffer_size = 16M
    read_rnd_buffer_size = 32M
    sort_buffer_size = 32M
    tmp_table_size = 64M
    join_buffer_size = 128M
    thread_cache_size = 64
    lower_case_table_names = 1
    ########log settings########
    log_error = error.log
    slow_query_log = 0
    slow_query_log_file = slow.log
    log_queries_not_using_indexes = 1
    log_slow_admin_statements = 1log_slow_slave_statements = 1
    log_throttle_queries_not_using_indexes = 0
    expire_logs_days = 7
    long_query_time = 2
    min_examined_row_limit = 100
    binlog-rows-query-log-events = 1
    log_bin_trust_function_creators = 1
    log_slave_updates = 1
    ########innodb settings########
    innodb_page_size = 16K
    innodb_buffer_pool_size = 4G
    innodb_buffer_pool_instances = 4
    innodb_buffer_pool_load_at_startup = 1
    innodb_buffer_pool_dump_at_shutdown = 1innodb_lru_scan_depth = 2000
    innodb_lock_wait_timeout = 5
    innodb_io_capacity = 1000
    innodb_io_capacity_max = 1500
    innodb_flush_method = O_DIRECT
    innodb_file_format = Barracuda
    innodb_file_format_max = Barracuda#innodb_log_group_home_dir = /redolog/
    #innodb_undo_directory = /undolog/
    innodb_undo_logs = 128
    innodb_undo_tablespaces = 3
    innodb_flush_neighbors = 1innodb_log_file_size = 4G
    innodb_log_files_in_group = 2
    innodb_log_buffer_size = 16777216
    innodb_purge_threads = 4
    innodb_large_prefix = 1
    innodb_thread_concurrency = 64
    innodb_print_all_deadlocks = 1
    innodb_strict_mode = 1
    innodb_sort_buffer_size = 67108864
    innodb_write_io_threads = 16
    innodb_read_io_threads = 16
    innodb_file_per_table = 1
    innodb_stats_persistent_sample_pages = 64
    innodb_autoinc_lock_mode = 2
    innodb_online_alter_log_max_size=1G
    innodb_open_files=4096
    #######replication settings########
    master_info_repository = TABLE
    relay_log_info_repository = TABLE
    log_bin = bin.log
    sync_binlog = 1
    gtid_mode = on
    enforce_gtid_consistency = 1
    log_slave_updates
    binlog_format = ROW
    binlog_rows_query_log_events = 1
    binlog_gtid_simple_recovery = 1
    relay_log = relay.log
    relay_log_recovery = 1
    slave_pending_jobs_size_max=2147483648
    slave_skip_errors = ddl_exist_errors
    slave-rows-search-algorithms = 'INDEX_SCAN,HASH_SCAN'
    plugin_dir=/usr/local/mysql/lib/plugin
    ########semi sync replication settings########
    #plugin_load = "validate_password.so;rpl_semi_sync_master=semisync_master.so;rpl_semi_sync_slave=semisync_slave.so"
    plugin_load = "rpl_semi_sync_master=semisync_master.so;rpl_semi_sync_slave=semisync_slave.so"
    loose_rpl_semi_sync_master_enabled = 1
    loose_rpl_semi_sync_slave_enabled = 1
    loose_rpl_semi_sync_master_timeout = 5000
    # password plugin #
    #validate_password_policy=STRONG
    #validate-password=FORCE_PLUS_PERMANENT
    # need change it
    report-host=127.0.0.1
    [mysqld-5.6]
    # metalock performance settings
    metadata_locks_hash_instances=64
    [mysqld-5.7]
    # new innodb settings #
    loose_innodb_numa_interleave=1
    innodb_buffer_pool_dump_pct = 40
    innodb_page_cleaners = 8
    innodb_undo_log_truncate = 1
    innodb_max_undo_log_size = 2G
    innodb_purge_rseg_truncate_frequency = 128
    # new replication settings #
    slave-parallel-type = LOGICAL_CLOCK
    slave-parallel-workers = 4
    slave_preserve_commit_order=1
    slave_transaction_retries=128
    # other change settings #
    binlog_gtid_simple_recovery=1
    log_timestamps=system
    show_compatibility_56=on
    # performance_schema #
    performance_schema_digests_size = 35000
    max_digest_length = 4096
    performance_schema_max_digest_length = 4096
    performance_schema_max_table_instances=35000

### 6、初始化数据库

    [root@mysql etc]# mysqld --initialize-insecure

### 7、配置MySQL启动项并设置为开机自启
在源码包的support-files目录下有一个mysql.server可以通过该文件启动MySQL服务
通常将MySQL配置为系统服务将其复制到/etc/init.d/下并重命名为mysqld
通过chkconfig将mysqld添加为开机自启服务

    [root@mysql mysql_data]# cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysqld
    [root@mysql mysql_data]# chkconfig --add mysqld
    [root@mysql mysql_data]# chkconfig --list mysqld
    mysqld          0:off   1:off   2:on    3:on    4:on    5:on    6:off

### 8、启动MySQL服务并修改密码

    [root@mysql mysql_data]# service mysqld start
    Starting MySQL..... SUCCESS!
    [root@mysql mysql_data]# ps -ef | grep mysql
    root       1358      1  0 04:12 pts/1    00:00:00 /bin/sh /usr/local/mysql/bin/mysqld_safe --datadir=/data/mysql_data --pid-file=/data/mysql_data/mysql.pid
    mysql      2578   1358 35 04:12 pts/1    00:00:04 /usr/local/mysql/bin/mysqld --basedir=/usr/local/mysql --datadir=/data/mysql_data --plugin-dir=/usr/local/mysql/lib/plugin --user=mysql --log-error=error.log --pid-file=/data/mysql_data/mysql.pid --port=3306
    root       2645   1187  0 04:12 pts/1    00:00:00 grep --color=auto mysql
    [root@mysql mysql_data]# mysqladmin -u root password Gepoint

<br/>至此MySQL部分安装完成，可以通过mysql -uroot -p验证是否能够登陆。如果想通过外部进行访问该数据库还需要关闭防火墙，以及修改/etc/selinux/config，最后需要创建一个root用户提供外部连接。

    [root@mysql ~]# vi /etc/selinux/config 
    SELINUX=disabled
    [root@mysql ~]# systemctl stop firewalld
    [root@mysql ~]# systemctl disable firewalld  
    mysql> create user 'root'%'%' identified by 'Gepoint';
    mysql> grant privileges on *.* to 'root'@'%';
    mysql> flush privileges;

## 附：
xtrabackup是MySQL物理热备利器。
安装步骤如下：
### 1、安装依赖

    [root@mysql opt]# yum install -y libev perl-DBD-MySQL perl-Digest-MD5 rsync

### 2、通过rpm包安装xtrabackup
xtrabackup的rpm包下载地址为https://www.percona.com/downloads/Percona-XtraBackup-2.4/LATEST/

    [root@mysql opt]# rpm -ivh percona-xtrabackup-24-2.4.15-1.el7.x86_64.rpm