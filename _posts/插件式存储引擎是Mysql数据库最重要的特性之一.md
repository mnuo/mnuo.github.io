---
title: 插件式存储引擎是Mysql数据库最重要的特性之一
date: 2016-06-07 23:53:08
tags: MySql
category: MySQL
---
##### 1) 默认情况下,创建新表不指定表的存储引擎,则新表是默认存储引擎,如果需要修改默认的存储引擎,则可以再参数文件中设置 default-table-type(5.5.3之后instead default_storage_engine).   
- 查看当前的默认存储引擎:
            
			mysql> show variables like 'default_storage_engine';
			+------------------------+--------+
			| Variable_name          | Value  |
			+------------------------+--------+
			| default_storage_engine | InnoDB |
			+------------------------+--------+
			1 row in set (0.00 sec)
			
- 查看支持的存储引擎:

			方法1:
					mysql> show engines \G
					*************************** 1. row ***************************
						  Engine: PERFORMANCE_SCHEMA
						 Support: YES
						 Comment: Performance Schema
					Transactions: NO
							  XA: NO
					  Savepoints: NO
					*************************** 2. row ***************************
						  Engine: MRG_MYISAM
						 Support: YES
						 Comment: Collection of identical MyISAM tables
					Transactions: NO
							  XA: NO
					  Savepoints: NO
					*************************** 3. row ***************************
						  Engine: CSV
						 Support: YES
						 Comment: CSV storage engine
					Transactions: NO
							  XA: NO
					  Savepoints: NO
					*************************** 4. row ***************************
						  Engine: BLACKHOLE
						 Support: YES
						 Comment: /dev/null storage engine (anything you write to it disappears)
					Transactions: NO
							  XA: NO
					  Savepoints: NO
					*************************** 5. row ***************************
						  Engine: MyISAM
						 Support: YES
						 Comment: MyISAM storage engine
					Transactions: NO
							  XA: NO
					  Savepoints: NO
					*************************** 6. row ***************************
						  Engine: MEMORY
						 Support: YES
						 Comment: Hash based, stored in memory, useful for temporary tables
					Transactions: NO
							  XA: NO
					  Savepoints: NO
					*************************** 7. row ***************************
						  Engine: ARCHIVE
						 Support: YES
						 Comment: Archive storage engine
					Transactions: NO
							  XA: NO
					  Savepoints: NO
					*************************** 8. row ***************************
						  Engine: InnoDB
						 Support: DEFAULT
						 Comment: Supports transactions, row-level locking, and foreign keys
					Transactions: YES
							  XA: YES
					  Savepoints: YES
					*************************** 9. row ***************************
						  Engine: FEDERATED
						 Support: NO
						 Comment: Federated MySQL storage engine
					Transactions: NULL
							  XA: NULL
					  Savepoints: NULL
					9 rows in set (0.00 sec)
			方法2:
				mysql> mysql> show engines;
				+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
				| Engine             | Support | Comment                                                        | Transactions | XA   | Savepoints |
				+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
				| PERFORMANCE_SCHEMA | YES     | Performance Schema                                             | NO           | NO   | NO         |
				| MRG_MYISAM         | YES     | Collection of identical MyISAM tables                          | NO           | NO   | NO         |
				| CSV                | YES     | CSV storage engine                                             | NO           | NO   | NO         |
				| BLACKHOLE          | YES     | /dev/null storage engine (anything you write to it disappears) | NO           | NO   | NO         |
				| MyISAM             | YES     | MyISAM storage engine                                          | NO           | NO   | NO         |
				| MEMORY             | YES     | Hash based, stored in memory, useful for temporary tables      | NO           | NO   | NO         |
				| ARCHIVE            | YES     | Archive storage engine                                         | NO           | NO   | NO         |
				| InnoDB             | DEFAULT | Supports transactions, row-level locking, and foreign keys     | YES          | YES  | YES        |
				| FEDERATED          | NO      | Federated MySQL storage engine                                 | NULL         | NULL | NULL       |
				+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
				9 rows in set (0.00 sec)
- 修改存储引擎:
			mysql> create table ai (i bigint(20) not null auto_increment,primary key(i))engine=MyISAM DEFAULT CHARSET=gbk;
			Query OK, 0 rows affected (0.00 sec)

			mysql> show tables;
			+----------------+
			| Tables_in_test |
			+----------------+
			| ai             |
			| emp            |
			+----------------+
			2 rows in set (0.00 sec)

			mysql> desc ai;
			+-------+------------+------+-----+---------+----------------+
			| Field | Type       | Null | Key | Default | Extra          |
			+-------+------------+------+-----+---------+----------------+
			| i     | bigint(20) | NO   | PRI | NULL    | auto_increment |
			+-------+------------+------+-----+---------+----------------+
			1 row in set (0.01 sec)

			mysql> show create table ai;
			+-------+-----------------------------------------------------------------------------------------------------------------------+
			| Table | Create Table                                                                                                          |
			+-------+-----------------------------------------------------------------------------------------------------------------------+
			| ai    | CREATE TABLE `ai` (
			  `i` bigint(20) NOT NULL AUTO_INCREMENT,
			  PRIMARY KEY (`i`)
			) ENGINE=MyISAM DEFAULT CHARSET=gbk |
			+-------+-----------------------------------------------------------------------------------------------------------------------+
			1 row in set (0.00 sec)

			mysql> CREATE TABLE country(country_id SMALLINT UNSIGNED NOT NULL AUTO_INCREMENT,
				-> country VARCHAR(50) NOT NULL,last_update TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,PRIMARY KEY (country_id))ENGINE=InnoDB DEFAULT CHARSET=utf8;
			Query OK, 0 rows affected (0.10 sec)

			mysql> desc contry;
			ERROR 1146 (42S02): Table 'test.contry' doesn't exist
			mysql> desc country;
			+-------------+----------------------+------+-----+-------------------+-----------------------------+
			| Field       | Type                 | Null | Key | Default           | Extra                       |
			+-------------+----------------------+------+-----+-------------------+-----------------------------+
			| country_id  | smallint(5) unsigned | NO   | PRI | NULL              | auto_increment              |
			| country     | varchar(50)          | NO   |     | NULL              |                             |
			| last_update | timestamp            | NO   |     | CURRENT_TIMESTAMP | on update CURRENT_TIMESTAMP |
			+-------------+----------------------+------+-----+-------------------+-----------------------------+
			3 rows in set (0.00 sec)

			mysql> alter table ai engine=innodb;
			Query OK, 0 rows affected (0.12 sec)
			Records: 0  Duplicates: 0  Warnings: 0

			mysql> show create table ai\G;
			*************************** 1. row ***************************
				   Table: ai
			Create Table: CREATE TABLE `ai` (
			  `i` bigint(20) NOT NULL AUTO_INCREMENT,
			  PRIMARY KEY (`i`)
			) ENGINE=InnoDB DEFAULT CHARSET=gbk
			1 row in set (0.00 sec)

			ERROR:
			No query specified

##### 2) 各个存储引擎特性
- 1) MyISAM   
    是Mysql的默认存储引擎.MyISAM不支持事务,也不支持外键.优势是访问的速度快,对事物完整性没有要求,或者以SELECT,INSERT为主的应用,基本上都可以使用这个引擎来创建表.每个MyISAM在磁盘上存储成3个文件,其文件名都和表名相同,但扩展名分别是:

				.frm 存储表定义;
				.MYD (MYData,存储数据);
				.MYI (MYIndex,存储索引)
			注意:该存储引擎,字段后的空格将被过滤
- 2) InnoDB   
innodb存储引擎提供了具有提交回滚和恢复能力的事务安全.但是对比MyISAM的存储引擎,InnoDB写的处理效率差一些并且会占用更多的磁盘空间以保留数据和索引	
    
    > 1> 自动增长列
	    	auto_increment 默认值1,auto_increment = n 默认值为n,但是这个默认值是存储在内存中,数据库关闭后需要重新启动
	    	使用: select last_insert_id();查看最后一次插入使用的值;
		    对于InnoDB表,自动增长列必须是索引,如果是组合索引,也必须是组合索引的第一列,对于MyISAM表,自动增长列可以是组合索引的其他列,这样插入记录后,自动增长列按照组合索引的前面几列进行排序后递增的			
                mysql> create table autoincre_demo
					-> (i smallint not null auto_increment,
					-> name varchar(10),primary key (i)
					-> )engine=innodb;
				Query OK, 0 rows affected (0.05 sec)
				mysql> insert into autoincre_demo values(1,'1'),(0,'2'),(null,'3');
				Query OK, 3 rows affected (0.03 sec)
				Records: 3  Duplicates: 0  Warnings: 0
				mysql> select * from autoincre_demo;
				+---+------+
				| i | name |
				+---+------+
				| 1 | 1    |
				| 2 | 2    |
				| 3 | 3    |
				+---+------+
				3 rows in set (0.00 sec)
				mysql> select LAST_INSERT_ID();
				+------------------+
				| LAST_INSERT_ID() |
				+------------------+
				|                2 |
				+------------------+
				1 row in set (0.02 sec)   
				mysql> INSERT INTO autoincre_demo values(4,'4');
				Query OK, 1 row affected (0.00 sec)   
                mysql> select Last_insert_in();
				ERROR 1305 (42000): FUNCTION test.Last_insert_in does not exist
				mysql> select Last_insert_id();
				+------------------+
				| Last_insert_id() |
				+------------------+
				|                2 |
				+------------------+
				1 row in set (0.00 sec)   
				mysql> drop table autoincre_demo;
				Query OK, 0 rows affected (0.03 sec   
				mysql> show tables;
				+----------------+
				| Tables_in_test |
				+----------------+
				| ai             |
				| country        |
				| emp            |
				+----------------+
				3 rows in set (0.00 sec)   
				mysql> create table autoincre_demo
					-> (d1 smallint not null auto_increment,
					-> d2 smallint not null,
					-> name varchar(10),
					-> index(d2,d1)
					-> )engine=myisam;
				Query OK, 0 rows affected (0.01 sec)
				mysql> insert into autoincre_demo(d2,name) values(2,'2'),(3,'3'),(4,'4'),(2,'2'),(3,'3'),(4,'4');
				Query OK, 6 rows affected (0.02 sec)
				Records: 6  Duplicates: 0  Warnings: 0
				mysql> select * from autoincr_demo;
				ERROR 1146 (42S02): Table 'test.autoincr_demo' doesn't exist
				mysql> select * from autoincre_demo;
				+----+----+------+
				| d1 | d2 | name |
				+----+----+------+
				|  1 |  2 | 2    |
				|  1 |  3 | 3    |
				|  1 |  4 | 4    |
				|  2 |  2 | 2    |
				|  2 |  3 | 3    |
				|  2 |  4 | 4    |
				+----+----+------+
				6 rows in set (0.00 sec)
				mysql>  

    > 2>外键约束
			Mysql支持外键的存储引擎只有InnoDB,在创建外键的时候,要求父表必须有对应的索引,子表在创建外键的时候也会自动创建对应的索引。
				CREATE TABLE city (
					city_id SMALLINT UNSIGNED NOT NULL AUTO_INCREMENT,
					city VARCHAR(50) NOT NULL,
					country_id SMALLINT UNSIGNED NOT NULL,
					last_update TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
					PRIMARY KEY (city_id),
					KEY idx_fk_country_id (country_id),
					CONSTRAINT fk_city_country FOREIGN KEY (country_id) REFERENCES country (country_id) ON DELETE RESTRICT  ON UPDATE CASCADE
				)ENGINE=INNODB DEFAULT CHARSET=UTF8;

    > 3>存储方式
				使用共享表空间存储,这种方式创建的表的表结构保存在 .frm文件中,数据和索引保存在innodb_data_home_dir和innodb_data_file_path定义的表空间中,可以是多个文件.
				使用多表空间存储,这种方式创建的表的表结构仍然保存在.frm文件中,但是每个表的数据和索引单独保存在.ibd中,如果是个分区表,则每个分区对应单独的.ibd文件,文件名是"表名+分区名",可以再创建分区的时候指定每个分区的数据文件的位置,以此来将表的IO均匀分布在多个磁盘上
			
- 3) MEMORY 
			存储引擎使用存在内存中的内容来创建表,每个memory表只实际对应一个磁盘文件,格式是.frm 
			MEMORY类型的表访问非常的快,因为他的数据是放在内存中的,并且默认使用HASH索引,但是一旦服务关闭,表中的数据就会丢失掉
- 4) MERGE 
			MERGE 存储引擎是一组MyISAM表的组合,这些MyISAM表必须结构完全相同,MERGE表本身没有数据,对MERGE类型的表可以进行查询,更新,删除的操作,这些操作实际上是对内部的实际的MYISAM表进行的,对于merge 类型表的插入操作是通过INSERT_METHOD字句定义插入的表,可以有3个不同的值,使用first或last值使得插入操作被相应的作用在第一或最后一个表上,不定义这个子句或者定义为NO,表示不能对这个MERGE表执行插入操作.
			
			创建3个测试表payment_2006,payment_2007和payment_all,其中payment_all是前两个表的MERGE表
				mysql> create table payment_2006(
					-> country_id smallint,
					-> payment_date datetime,
					-> amount DECIMAL(15,2),
					-> key idx_fk_country_id (country_id)
					-> )engine=myisam default charset=utf8;
				Query OK, 0 rows affected (0.01 sec)

				mysql> create table payment_2007( country_id smallint, payment_date datetime, amount DECIMAL(15,2), key idx_fk_country_id (country_id) )engine=myisam default charset=utf8;
				Query OK, 0 rows affected (0.02 sec)

				mysql> create table payment_all( country_id smallint, payment_date datetime, amount DECIMAL(15,2), INDEX (country_id) )engine=merge union=(payment_2006,payment_2007) insert_method=last;
				Query OK, 0 rows affected (0.01 sec)

			分别向payment+2006和payment_2007插入数据:\c
				mysql> insert into payment_2006 values(1,'2006-05-01',100000),(2,'2006-08-15',150000);
				Query OK, 2 rows affected (0.00 sec)
				Records: 2  Duplicates: 0  Warnings: 0

				mysql> insert into payment_2007 values(1,'2007-02-20',35000),(2,'2007-07-15',220000);
				Query OK, 2 rows affected (0.00 sec)
				Records: 2  Duplicates: 0  Warnings: 0

				mysql> select * from payment_all;
				+------------+---------------------+-----------+
				| country_id | payment_date        | amount    |
				+------------+---------------------+-----------+
				|          1 | 2006-05-01 00:00:00 | 100000.00 |
				|          2 | 2006-08-15 00:00:00 | 150000.00 |
				|          1 | 2007-02-20 00:00:00 |  35000.00 |
				|          2 | 2007-07-15 00:00:00 | 220000.00 |
				+------------+---------------------+-----------+
				4 rows in set (0.00 sec)
				
			插入数据:
				mysql> insert into payment_all values(3,'2006-03-20',112200);
				Query OK, 1 row affected (0.00 sec)

				mysql> select * from payment_all;
				+------------+---------------------+-----------+
				| country_id | payment_date        | amount    |
				+------------+---------------------+-----------+
				|          1 | 2006-05-01 00:00:00 | 100000.00 |
				|          2 | 2006-08-15 00:00:00 | 150000.00 |
				|          1 | 2007-02-20 00:00:00 |  35000.00 |
				|          2 | 2007-07-15 00:00:00 | 220000.00 |
				|          3 | 2006-03-20 00:00:00 | 112200.00 |
				+------------+---------------------+-----------+
				5 rows in set (0.00 sec)

