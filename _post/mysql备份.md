---
title: mysql备份
tags: MySql
category: MySQL
---
#### 1,备份/恢复策略
> ☆ 确定要备份的表的存储引擎是事务型还是非事务性， 两种不同的存储引擎备份方式在处理数据一致性方面是不太一样的。

> ☆确定使用全备份还是增量备份。全备份的优点是备份保持最新备份，恢复的时候可以花费更少的时间；缺点是如果数据量大，将会花费很多的时间，并对系统造成较长时间的压力。增量备份则恰恰相反，只需要备份每天的增量日志，备份时间少，对负载压力也小； 缺点就是恢复的时候需要全备份加上次备份到故障前的所有日志，恢复时间会长些。

> ☆ 可以考虑采取复制的方法来做异地备份，但是记住，复制不能代替备份，它对数据库的误操作也无能为力。

> ☆ 要定期做备份，备份的周期要充分考虑系统可以承受的恢复时间。备份要在系统负载较小的时候进行。

> ☆ 确保 MySQL 打开 log-bin 选项，有了 BINLOG，MySQL 才可以在必要的时候做完整恢复，或基于时间点的恢复，或基于位置的恢复。

> ☆ 要经常做备份恢复测试，确保备份是有效的，并且是可以恢复的。



#### 2, 逻辑备份和恢复

在MySQL中，逻辑备份的最大优点是对于各种存储引擎,都可以用同样的方法来备份;而物理备份则不同,不通过的存储引擎有着不同的备份方法。

+ 1) 备份   
使用mysqldump工具来完成逻辑备份

    - 备份指定的数据库，或者此数据库中某些表。

			shell> mysqldump [options] db_name [tables]

	- 备份指定的一个或多个数据库。

			shell> mysqldump [options] ---database DB1 [DB2 DB3...]

	- 备份所有数据库。

			shell> mysqldump [options] --all--database
		- （1）备份所有数据库：

				[zzx@localhost ~]$ mysqldump -uroot -p --all-database > all.sql

				Enter password:

		- （2）备份数据库 test：

			    [zzx@localhost ~]$ mysqldump -uroot -p test > test.sql

				Enter password:

		- （3）备份数据库 test 下的表 emp：

			    [zzx@localhost ~]$ mysqldump -uroot -p test emp > emp.sql

				Enter password:

		- （4）备份数据库 test 下的表 emp 和 dept：

				[zzx@localhost ~]$ mysqldump -uroot -p test emp dept > emp_dept.sql

				Enter password:

		- （5）备份数据库 test 下的所有表为逗号分割的文本，备份到/tmp：

				[root@localhost tmp]# mysqldump -uroot -T /tmp test emp --fields-terminated-by ','

				[root@localhost tmp]# more emp.txt

				1,z1

				2,z2

				3,z3

				4,z4

				1,z1