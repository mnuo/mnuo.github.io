---
title: mysql日志
date: 2016-07-12 17:12:08 
tags: MySql
category: MySQL
---
#### 1  错误日志
错误日志是MySQL中最重要的日志之一,他记录了当mysqld启动和停止时,以及服务器在运行过程中发生任何严重错误时的相关信息,当数据库出现任何故障导致无法正常使用时们可以首先查看此日志。

可以使用--log-error[=file_name]选项来指定mysqld保存错误日志文件的位置

如果没有给定的file_name值,mysqld使用错误日志名host_name.err(host_name为主机名),并默认在参数DATADIR(数据目录)指定的目录中写入日志文件

	[root@iZ94ibhx8rnZ ~]# ps -ef|grep mysql

	root     24981     1  0 Jun03 ?        00:00:00 /bin/sh /usr/bin/mysqld_safe --datadir=/var/lib/mysql --pid-file=/var/lib/mysql/iZ94ibhx8rnZ.pid

	mysql    25096 24981  0 Jun03 ?        00:06:07 /usr/sbin/mysqld --basedir=/usr --datadir=/var/lib/mysql --plugin-dir=/usr/lib64/mysql/plugin --user=mysql --log-error=/var/lib/mysql/iZ94ibhx8rnZ.err --pid-file=/var/lib/mysql/iZ94ibhx8rnZ.pid

	root     32651 32636  0 09:20 pts/0    00:00:00 mysql -u root -px xxxxxxxxxxxxxxxxxx

	root     32677 32659  0 13:26 pts/1    00:00:00 grep mysql

	[root@iZ94ibhx8rnZ ~]# cd /var/lib/mysql/

	[root@iZ94ibhx8rnZ mysql]# ls

	auto.cnf  ib_logfile0  iZ94ibhx8rnZ.err  lifeworld  mysql.sock          RPM_UPGRADE_HISTORY      test

	ibdata1   ib_logfile1  iZ94ibhx8rnZ.pid  mysql      performance_schema  RPM_UPGRADE_MARKER-LAST

	[root@iZ94ibhx8rnZ mysql]# more iZ94ibhx8rnZ.err

	160603 16:45:05 mysqld_safe Starting mysqld daemon with databases from /var/lib/mysql

	2016-06-03 16:45:05 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see docum

	entation for more details).

	2016-06-03 16:45:05 25096 [Note] Plugin 'FEDERATED' is disabled.

	2016-06-03 16:45:05 25096 [Note] InnoDB: Using atomics to ref count buffer pool pages

	2016-06-03 16:45:05 25096 [Note] InnoDB: The InnoDB memory heap is disabled

####  2 二进制日志

二进制日志(BINLOG)记录了所有的DDL(数据定义语言)语句和DML(数据操纵语言)语句,但是不包括数据查询语句,语句以"事件"的形式保存,它面熟了数据的更改过程,此日志对于灾难时的数据恢复起着极其重要的作用。

+ 1) 日志的位置和格式:   
    当用--log-bin[=file_name]选项启动时， mysqld 将包含所有更新数据的 SQL 命令写入日志文件。如果没有给出 file_name 值，默认名为主机名后面跟“-bin” 。如果给出了文件名，但没有包含路径，则文件默认被写入参数 DATADIR（数据目录）指定的目录。

		[root@iZ94ibhx8rnZ mysql]# vi /usr/my.cnf

		log_bin=iZ94ibhx8rnZ-bin.log

+ 2) 查看 使用mysqlbinlog工具查看

	mysqlbinlog log-file

		[root@iZ94ibhx8rnZ mysql]# mysqlbinlog iZ94ibhx8rnZ-bin.000001 -d test

		/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=1*/;

		/*!40019 SET @@session.max_insert_delayed_threads=0*/;

		/*!50003 SET @OLD_COMPLETION_TYPE=@@COMPLETION_TYPE,COMPLETION_TYPE=0*/;

		DELIMITER /*!*/;

		# at 4

		#160613 13:44:33 server id 1  end_log_pos 120 CRC32 0x8e1c0755  Start: binlog v 4, server v 5.6.21-log created 160613 13:44:33 at startup

		# Warning: this binlog is either in use or was not closed properly.

		ROLLBACK/*!*/;

		BINLOG '

		QUheVw8BAAAAdAAAAHgAAAABAAQANS42LjIxLWxvZwAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA

		AAAAAAAAAAAAAAAAAABBSF5XEzgNAAgAEgAEBAQEEgAAXAAEGggAAAAICAgCAAAACgoKGRkAAVUH

		HI4=

		'/*!*/;

		# at 120

		#160613 13:50:37 server id 1  end_log_pos 199 CRC32 0x99d9ba92  Query   thread_id=2     exec_time=0     error_code=0

		SET TIMESTAMP=1465797037/*!*/;

		SET @@session.pseudo_thread_id=2/*!*/;

		SET @@session.foreign_key_checks=1, @@session.sql_auto_is_null=0, @@session.unique_checks=1, @@session.autocommit=1/*!*/;

		SET @@session.sql_mode=1075838976/*!*/;

		SET @@session.auto_increment_increment=1, @@session.auto_increment_offset=1/*!*/;

		/*!\C utf8 *//*!*/;

		SET @@session.character_set_client=33,@@session.collation_connection=33,@@session.collation_server=8/*!*/;

		SET @@session.lc_time_names=0/*!*/;

		SET @@session.collation_database=DEFAULT/*!*/;

		BEGIN

		/*!*/;

		# at 199

		#160613 13:50:37 server id 1  end_log_pos 312 CRC32 0xb56e27db  Query   thread_id=2     exec_time=0     error_code=0

		use `test`/*!*/;

		SET TIMESTAMP=1465797037/*!*/;

		insert into actor values(9,'Wang','Wu')

		/*!*/;

		# at 312

		#160613 13:50:37 server id 1  end_log_pos 343 CRC32 0x7d6367a9  Xid = 25

		COMMIT/*!*/;

		# at 343

		#160613 13:50:52 server id 1  end_log_pos 422 CRC32 0x40e047f9  Query   thread_id=2     exec_time=0     error_code=0

		SET TIMESTAMP=1465797052/*!*/;

		BEGIN

		/*!*/;

		# at 422

		#160613 13:50:52 server id 1  end_log_pos 538 CRC32 0x32226933  Query   thread_id=2     exec_time=0     error_code=0

		SET TIMESTAMP=1465797052/*!*/;

		insert into actor values(10,'Zhang','san')

		/*!*/;

		# at 538

		#160613 13:50:52 server id 1  end_log_pos 569 CRC32 0x39986639  Xid = 26

		COMMIT/*!*/;

		DELIMITER ;

		# End of log file

		ROLLBACK /* added by mysqlbinlog */;

		/*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*/;

		/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=0*/;



+ (3) 日志的删除

    - 方法1: 使用reset master

			mysql> system ls -ltr /var/lib/mysql/iZ94ibhx8rnZ-bin*

			-rw-rw---- 1 mysql mysql  26 Jun 13 13:44 /var/lib/mysql/iZ94ibhx8rnZ-bin.index

			-rw-rw---- 1 mysql mysql 569 Jun 13 13:50 /var/lib/mysql/iZ94ibhx8rnZ-bin.000001

			mysql> reset master;

			Query OK, 0 rows affected (0.03 sec)

			mysql> system ls -ltr /var/lib/mysql/iZ94ibhx8rnZ-bin*

			-rw-rw---- 1 mysql mysql 120 Jun 13 14:04 /var/lib/mysql/iZ94ibhx8rnZ-bin.000001

			-rw-rw---- 1 mysql mysql  26 Jun 13 14:04 /var/lib/mysql/iZ94ibhx8rnZ-bin.index

	- 方法2: 使用 purge master logs to ...

	    - (1)查看删除前日志。

				mysql> system ls -ltr localhost-bin*

				-rw-rw---- 1 mysql mysql 145 Aug 10 04:00 localhost-bin.000004

				-rw-rw---- 1 mysql mysql 98 Aug 10 04:00 localhost-bin.000006

				-rw-rw---- 1 mysql mysql 145 Aug 10 04:00 localhost-bin.000005

				-rw-rw---- 1 mysql mysql 108 Aug 10 04:00 localhost-bin.index

		- (2)用 PURGE 命令进行删除。

				mysql> purge master logs to 'localhost-bin.000006';

				Query OK, 0 rows affected (0.04 sec)

		- (3)查看删除后日志。

				mysql> system ls -ltr localhost-bin*

				-rw-rw---- 1 mysql mysql 98 Aug 10 04:00 localhost-bin.000006

				-rw-rw---- 1 mysql mysql 36 Aug 10 04:03 localhost-bin.index

			从结果中发现，编号“000006”之前的所有日志都已经被删除。

	- 方法3:	执行“PURGE MASTER LOGS BEFORE 'yyyy-mm-dd hh24:mi:ss'”命令，该命令将删除日期为"yyyy-mm-dd hh24:mi:ss"之前产生的所有日志。 下例中删除了日期在 “2007-08-10 04:07:00”之前的所有日志。

		- （1）查看删除前日志。

				mysql> system ls -ltr localhost-bin*

				-rw-rw---- 1 mysql mysql 145 Aug 10 04:06 localhost-bin.000001

				-rw-rw---- 1 mysql mysql 145 Aug 10 04:06 localhost-bin.000002

				-rw-rw---- 1 mysql mysql 145 Aug 10 04:06 localhost-bin.000003

				-rw-rw---- 1 mysql mysql 145 Aug 10 04:06 localhost-bin.000004

				-rw-rw---- 1 mysql mysql 145 Aug 10 04:06 localhost-bin.000005

				-rw-rw---- 1 mysql mysql 252 Aug 10 04:07 localhost-bin.index

				-rw-rw---- 1 mysql mysql 98 Aug 10 04:07 localhost-bin.000007

				-rw-rw---- 1 mysql mysql 145 Aug 10 04:07 localhost-bin.000006

		- （2）用 PURGE 命令删除“2007-08-10 04:07:00”之前的所有日志。

				mysql> purge master logs before '2007-08-10 04:07:00';

				Query OK, 0 rows affected (0.04 sec)

		- （3）查看删除后日志，果然，系统只保留了两个指定删除日期后的日志。

				mysql> system ls -ltr localhost-bin*

				-rw-rw---- 1 mysql mysql 98 Aug 10 04:07 localhost-bin.000007

				-rw-rw---- 1 mysql mysql 145 Aug 10 04:07 localhost-bin.000006

				-rw-rw---- 1 mysql mysql 72 Aug 10 04:08 localhost-bin.index	

	- 方法4:设置参数--expire_logs_days=#，此参数的含义是设置日志的过期天数，过了指定的天数后日志将会被自动删除，这样将将有利于减少 DBA 管理日志的工作量。下例中通过手工更改系统日期来测试此参数的使用。

		- （1）查看删除前日志。

				mysql> system ls -ltr localhost-bin.*

				-rw-rw---- 1 mysql mysql 145 Dec 25 00:00 localhost-bin.000041

				-rw-rw---- 1 mysql mysql 145 Dec 25 00:06 localhost-bin.000042

				-rw-rw---- 1 mysql mysql 144 Dec 25 00:06 localhost-bin.index

				-rw-rw---- 1 mysql mysql 98 Dec 25 00:06 localhost-bin.000044

				-rw-rw---- 1 mysql mysql 145 Dec 25 00:06 localhost-bin.000043

		- （2）在 my.cnf 的[mysqld]中加入“expire_logs_day=3” ，然后重新启动 MySQL 服务。

				[root@localhost zzx]# more /etc/my.cnf

				[mysqld]

				ndbcluster

				log-bin

				expire_logs_day=3

				319

				[root@localhost log]# service mysql restart

				Shutting down MySQL.[ OK ]

				Starting MySQL[ OK ]

		- （3）将系统时间改为 1 天以后。

				[root@localhost mysql]# date

				Tue Dec 25 01:14:19 CST 2007

				[root@localhost mysql]# date -s '20071226 14:00:00'

				Wed Dec 26 14:00:00 CST 2007

		- （4）用“flush logs”触发日志文件更新，这是由于没有到 3 天，所有日志将不会被删除。

				[root@localhost mysql]# mysqladmin flush-log

				[root@localhost mysql]# ls -ltr |grep 'localhost-bin*'

				-rw-rw---- 1 mysql mysql 145 Dec 25 00:00 localhost-bin.000041

				-rw-rw---- 1 mysql mysql 145 Dec 25 00:06 localhost-bin.000042

				-rw-rw---- 1 mysql mysql 145 Dec 25 00:06 localhost-bin.000043

				-rw-rw---- 1 mysql mysql 145 Dec 26 14:00 localhost-bin.000044

				-rw-rw---- 1 mysql mysql 216 Dec 26 14:00 localhost-bin.index

				-rw-rw---- 1 mysql mysql 98 Dec 26 14:00 localhost-bin.000046

				-rw-rw---- 1 mysql mysql 145 Dec 26 14:00 localhost-bin.000045

		- （5）将日期改为 3 天以后，再次执行“flush logs”触发日志文件更新。

				[root@localhost mysql]# date -s '20071228 14:00:00'

				Fri Dec 28 14:00:00 CST 2007

				[root@localhost mysql]# mysqladmin flush-log

				[root@localhost mysql]# ls -ltr |grep 'localhost-bin*'

				-rw-rw---- 1 mysql mysql 145 Dec 26 14:00 localhost-bin.000044

				-rw-rw---- 1 mysql mysql 145 Dec 26 14:00 localhost-bin.000045

				-rw-rw---- 1 mysql mysql 144 Dec 28 14:00 localhost-bin.index

				-rw-rw---- 1 mysql mysql 98 Dec 28 14:00 localhost-bin.000047

				-rw-rw---- 1 mysql mysql 145 Dec 28 14:00 localhost-bin.000046

			从结果中可以看出，3 天前日志 localhost-bin.000041-- localhost-bin.000043 都已经被删除。

		- (4) 其他选项

	        二进制日志由于记录了数据的变化过程，对于数据的完整性和安全性起着非常重要的作用。因此，MySQL 还提供了一些其他参数选项来进行更小粒度的管理，具体介绍如下。

            	--binlog-do-db=db_name

			该选项告诉主服务器，如果当前的数据库(即 USE 选定的数据库)是 db_name，应将更新记录到二进制日志中。其他所有没有显式指定的数据库更新将被忽略，不记录在日志中。

                --binlog-ignore-db=db_name

			该选项告诉主服务器，如果当前的数据库(即 USE 选定的数据库)是 db_name，不应将更新保存到二进制日志中，其他没有显式忽略的数据库都将进行记录。如果想记录或忽略多个数据库，可以对上面两个选项分别使用多次，即对每个数据库指定相应的选项。例如，如果只想记录数据库 db1 和 db2 的日志，可以在参数文件中设置两行：

				--binlog-do-db=db1

				--binlog-do-db=db2
                
                --innodb-safe-binlog

			此选项经常和--sync-binlog＝N（每写 N 次日志同步磁盘）一起配合使用，使得事务在日志中的记录更加安全。

                SET SQL_LOG_BIN=0

			具有 SUPER 权限的客户端可以通过此语句禁止将自己的语句记入二进制记录。这个选项在某些环境下是有用的，但是使用时一定要小心，因为它很可能造成日志记录的不完整或者在复制环境中造成主从数据的不一致。		



#### 3 查询日志

查询日志记录了客户端的所有语句,而二进制日志不包含只查询的语句

+ (1) 日志的位置:

    	[root@iZ94ibhx8rnZ mysql]# cat /usr/my.cnf
    
		# changes to the query log between backups.

		general_log=ON

		general_log_file=iZ94ibhx8rnZ.log

+ (2) 读取

		[root@iZ94ibhx8rnZ mysql]# cat iZ94ibhx8rnZ.log

		/usr/sbin/mysqld, Version: 5.6.21-log (MySQL Community Server (GPL)). started with:

		Tcp port: 0  Unix socket: (null)

		Time                 Id Command    Argument

		160613 14:33:31     1 Connect   root@localhost on test

							1 Query     show databases

							1 Query     show tables

							1 Field List        actor

							1 Field List        books

							1 Field List        books2

							1 Field List        company2

							1 Field List        duck_cust

							1 Field List        film

							1 Field List        film_text

							1 Field List        innodb_monitor

							1 Field List        inventory

							1 Field List        order_rab

							1 Field List        sales

							1 Field List        sales3

							1 Field List        sales_views3

							1 Field List        t

							1 Field List        t1

							1 Field List        t2

							1 Field List        users

							1 Field List        users2

							1 Query     reset master

		160613 14:34:33     1 Query     insert into actor values(12,'Lii','Sii')

		160613 14:34:40     1 Query     select * from actor

	一般情况下建议关闭

			

#### 4  慢查询日志

+ (1) 位置和开启

		mysql> show variables like '%quer%';

			+----------------------------------------+--------------------------------------+

			| Variable_name                          | Value                                |

			+----------------------------------------+--------------------------------------+

					...

			| slow_query_log                         | OFF                                  |

			| slow_query_log_file                    | /var/lib/mysql/iZ94ibhx8rnZ-slow.log |

			+----------------------------------------+--------------------------------------+

			[root@iZ94ibhx8rnZ mysql]# cat /usr/my.cnf

			# open slow query log

			slow_query_log=ON

			# change to slow query log

			#slow_query_log_file=/var/lib/mysql/iZ94ibhx8rnZ-slow.log

+ (2) 查看读取

			[root@iZ94ibhx8rnZ mysql]# cat iZ94ibhx8rnZ-slow.log

			/usr/sbin/mysqld, Version: 5.6.21-log (MySQL Community Server (GPL)). started with:

			Tcp port: 0  Unix socket: (null)

			Time                 Id Command    Argument

			[root@iZ94ibhx8rnZ mysql]# mysqldumpslow iZ94ibhx8rnZ-slow.log



			Reading mysql slow query log from iZ94ibhx8rnZ-slow.log

			Count: 1  Time=0.00s (0s)  Lock=0.00s (0s)  Rows=0.0 (0), 0users@0hosts



+ (3) 其他选项

			在 MySQL 5.1 中，通过--log-slow-admin-statements 服务器选项，可以请求将慢管理语句，例如 OPTIMIZE TABLE、ANALYZE TABLE 和 ALTER TABLE 写入慢查询日志。

