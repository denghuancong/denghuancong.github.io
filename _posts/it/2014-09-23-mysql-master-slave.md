---
layout: post
title:  "Mysql设置Master和Slave同步"
categories:
    - it
date:   2014-09-23 23:46:33
categories: post mysql master slave replication
---

#### 工作2年都没有自己做过mysql的主从配置，平时使用的比较多，今天有个服务要用到，就写来记录下，下面的操作基于这个环境：
	Centos 7
	Mysql Ver 14.14 Distrib 5.1.73
	Master IP 假设为 10.10.10.1
	Slave IP 假设为 10.10.10.2

### 配置Master
#### 1.打开mysql的配置文件
	vim /etc/mysql.my.conf


#### 2.设置绑定地址，这个参数在[mysqld]下面
	bind-address=10.10.10.1


#### 3.修改db服务ID，待会也需要设置slave的，必须和这里的不一样
	sever-id = 1

#### 4.设置binlog文件的位置
	log_bin = /var/log/mysql/mysql-bin.log

#### 5.设置那个db要记录binlog
	binlog_do_db = db_to_replication

#### 6.重启mysqld
	service mysqld restart

#### 7.登陆mysql,设置slave访问的权限,slave的账户也使用root,密码为123（假设）
	GRANT REPLICATION SLAVE ON *.* TO 'root'@'10.10.10.2' IDENTIFIED BY '123';

#### 8.进去要同步的库
	use db_to_replication;

#### 9.锁定db_to_replication，以免有数据进入，如果当前没有服务接入可以跳过这步。
	FLUSH TABLES WITH READ LOCK;

#### 10.查看Master的状态（注意，mysql新版本都叫mariadb，mysql被卖了 - -!)，记录position和file，等下会用到，其实这里的file为binlog文件，postiton为slave要开始执行同步的位置。
	MariaDB [db_to_replication]> show master status;
	+------------------+----------+--------------+------------------+
	| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
	+------------------+----------+--------------+------------------+
	| mysql-bin.000002 | 1234     | db_to_replication|              |

#### 11.导出Master要同步的库，等下Slave需要用到。
	mysqldump -u root -p --opt db_to_replication > db_to_replication.sql

#### 12.恢复db_to_replication，如果刚才没有执行第8步，那这里就不需要恢复了
	UNLOCK TABLES;

#### 13.退出，Master配置完成。
	quit

### 设置Slave
#### 1.创建db_to_replication库
	create database db_to_replication;

#### 2.导入上面在Master导出的db_to_replication库
	mysql -u root -p db_to_replication < /path/to/db_to_replication.sql

#### 3.打开mysql的配置文件
	vim /etc/mysql.my.conf

#### 4.修改db服务ID,要和Master不一样
	sever-id = 2

#### 5.binlog的日志的位置
	relay-log = /var/log/mysql/mysql-relay-bin.log
	log_bin = /var/log/mysql/mysql-bin.log

#### 6.设置那个db要记录binlog
	binlog_do_db = db_to_replication

#### 7.relay相关文件位置
	relay_log = /var/lib/mysql/mysql-relay-bin
	relay-log-index = /var/lib/mysql/relay-bin.index
	relay-log-info-file = /var/lib/mysql/relay-bin.info

#### 8.重启mysql服务
	service mysqld restart

#### 9.进入mysql，设置Master的信息,假设发送binlog的用户为root，密码为456，log文件名和开始同步的位置为设置Master中第10步记录的。
	CHANGE MASTER TO MASTER_HOST='10.10.10.1',MASTER_USER='root',
	MASTER_PASSWORD='456',MASTER_LOG_FILE='mysql-bin.000002', MASTER_LOG_POS=1234;

#### 10.开启Slave
	START SLAVE;

### 现在往Master插入一条数据就会发生同步到Slave，其他修改也是。
