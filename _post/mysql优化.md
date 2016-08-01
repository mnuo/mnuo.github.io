---
title: sql优化
date: 2016-07-12 14:12:08 
tags: MySql
category: MySQL
---
#### 1 优化sql语句的一般步骤:

+ 1> 通过show status 命令了解各种sql的执行频率

		mysql> show status like 'Innodb_rows%';

		+----------------------+-------+

		| Variable_name        | Value |

		+----------------------+-------+

		| Innodb_rows_deleted  | 0     |

		| Innodb_rows_inserted | 185   |

		| Innodb_rows_read     | 312   |

		| Innodb_rows_updated  | 4     |

		+----------------------+-------+

		4 rows in set (0.00 sec)

		mysql> show global status like 'Com_selec%';

		+---------------+-------+

		| Variable_name | Value |

		+---------------+-------+

		| Com_select    | 404   |

		+---------------+-------+

		1 row in set (0.00 sec)

		mysql> show global status like 'Com_commit';

		+---------------+-------+

		| Variable_name | Value |

		+---------------+-------+

		| Com_commit    | 4     |

		+---------------+-------+

		1 row in set (0.00 sec)

		mysql> show global status like 'Com_rollback';

		+---------------+-------+

		| Variable_name | Value |

		+---------------+-------+

		| Com_rollback  | 1     |

		+---------------+-------+

		1 row in set (0.00 sec)

		mysql> show global status like 'connections'

			-> ;

		+---------------+-------+

		| Variable_name | Value |

		+---------------+-------+

		| Connections   | 450   |

		+---------------+-------+

		1 row in set (0.00 sec)

		mysql> show status like 'connections';

		+---------------+-------+

		| Variable_name | Value |

		+---------------+-------+

		| Connections   | 450   |

		+---------------+-------+

		1 row in set (0.00 sec)

		mysql> show status like 'uptime';

		+---------------+--------+

		| Variable_name | Value  |

		+---------------+--------+

		| Uptime        | 342771 |

		+---------------+--------+

		1 row in set (0.00 sec)

		mysql> show status like 'slow queries';#慢查询

		Empty set (0.00 sec)

		mysql> show status like 'slow_queries';

		+---------------+-------+

		| Variable_name | Value |

		+---------------+-------+

		| Slow_queries  | 0     |

		+---------------+-------+

		1 row in set (0.00 sec)
+ 2>定位执行效率较低的SQL语句:  
	- 通过慢查询日志定位那些执行效率较低的SQL 语句,用--log-slow-queries[=file_name]选项启动时,mysqld 写一个包含所有执行时间超过long_query_time秒的SQL语句的日志文件。具体查看日志管理章节

	- 慢查询日志在查询结束之后才记录,所以在应用反应执行效率出现问题的时候查询慢查询日志并不能定位问题,可以使用show processlist命令查看当前MySQL在进行的线程,包括线程的状态,是否锁表等,可以实时地查看SQL的执行情况,同时对一些梭镖操作进行优化。

+ 3> 通过EXPLAIN 分析低效SQL的执行计划

    	mysql> explain select year,country,product,sum(profit) from sales group by year,country,product with rollup\G;
    
    	*************************** 1. row ***************************
    
    			   id: 1
    
    	  select_type: SIMPLE
    
    			table: sales
    
    			 type: ALL
    
    	possible_keys: NULL
    
    			  key: NULL
    
    		  key_len: NULL
    
    			  ref: NULL
    
    			 rows: 12
    
    			Extra: Using filesort
    
    	1 row in set (0.00 sec)

		ERROR:

		No query specified
	每个列的简单解释:
    - select_type : 表示SELECT 的类型,常见的取值有SIMPLE(简单表,即不适用表连接或者子查询),PRIMARY(主查询,即外层的查询),UNION(union中的第二个或者后面的查询语句),SUBQUERY(子查询中的第一个SELECT)等。  

    - table: 输出结果集的表   
   
    - type: 表示表的连接类型,性能由好到差的连接类型为:
        - system: 表中仅有一行,即常量表

		- const: 但表中最多有一个匹配行,例如primary key或者unique index

		- eq_ref: 对于前面的每一行,在此表中只查询一条记录,简单来说就是多表连接中primary key 或者 unique index

		- ref: req_ref类似,区别在于不是使用primary key 或者unique index,而是使用普通的索引

		- ref_or_null: 与ref类似,区别在于条件中包含对NULL的查询

		- index_merge: 索引合并优化

		- unique_subquery: in的后面是一个查询字段的子查询

		- index_subquery: 与unique_subquery类似,区别在于in的后面是查询非唯一索引字段的子查询

		- range: 单表中的范围查询

		- index: 对于前面的每一行,都通过查询索引来得到数据

		- all: 对于前面的每一行,都通过权标扫面来得到数据	

    - 	possible_keys: 表示查询时,可能使用的索引

    - key: 表示实际使用的索引

	- key_len: 索引字段的长度

	- rows: 扫描行的数量

	- extra: 执行情况的说明和描述

+ 4> 确定问题并采取相应的优化措施   
经过以上步骤,基本就可以去人问题出现的原因。

		mysql> create index ind_sales2_year on sales(year);

		Query OK, 0 rows affected (0.15 sec)

		Records: 0  Duplicates: 0  Warnings: 0

		mysql> explain select year,country,product,sum(profit) from sales group by year,country,product with rollup\G;

		*************************** 1. row ***************************

				   id: 1

		  select_type: SIMPLE

				table: sales

				 type: ALL

		possible_keys: NULL

				  key: NULL

			  key_len: NULL

				  ref: NULL

				 rows: 12

				Extra: Using filesort

		1 row in set (0.00 sec)

		ERROR:

		No query specified

		mysql> explain select year,country,product,sum(profit) from sales where year in (2004,2005,2006) group by year,country,product with rollu

		*************************** 1. row ***************************

				   id: 1

		  select_type: SIMPLE

				table: sales

				 type: ALL

		possible_keys: ind_sales2_year

				  key: NULL

			  key_len: NULL

				  ref: NULL

				 rows: 12

				Extra: Using where; Using filesort

		1 row in set (0.02 sec)

		ERROR:

		No query specified



#### 2 索引问题

> 索引是数据库优化中最常用也是最重要的手段之一,通过索引通常可以帮助用户解决大多数的SQL性能问题

			

+ 	1) 索引的存储分类
    - MyISAM存储引擎的表的数据和索引是自动分开存储的,各自是独立的一个文件;InnoDB存储引擎的表的数据和多音是存储在同一个表空间里面,但可以 有多个文件组成。

    - MySQL中索引的存储类型目前只有两种:BTREE和HASH,具体和表的存储引擎相关:
        - MyISAM 和 INNODB存储引擎都只支持BTREE索引;Memory/HEAP存储引擎可以支持HASH和BTREE索引
        - MySQL目前不支持函数索引,但是能对列的前面某一部分建索引,例如name字段,可以只取name的前4个字符进行索引,这个特性可以大大缩小索引文件的大小,用户在设计表结构的时候,也可以对文本列根据此特性进行灵活设计
        例:
        
                create index ind_company2_name on company2(name(4));

+ 2) MySQL如何使用索引
    - 1> 使用索引
        - (1) 对于创建的多列索引,只要查询的条件中用到了最左边的列,索引一般就会被使用

    			mysql> create index ind_sales2_company_id_moneys on sales2(company_id,moneys);
    
    			Query OK, 0 rows affected (0.07 sec)
    
    			Records: 0  Duplicates: 0  Warnings: 0

				mysql> explain select * from sales2 where company_id=1\G;

				*************************** 1. row ***************************

						   id: 1

				  select_type: SIMPLE

						table: sales2

						 type: ALL

				possible_keys: ind_sales2_company_id_moneys

						  key: NULL

					  key_len: NULL

						  ref: NULL

						 rows: 12

						Extra: Using where

				1 row in set (0.00 sec)

				ERROR:

				No query specified

			根据以上sql 即便where 条件中不是用的company_id 与moneys的组合条件,索引仍然能用到,这就是索引的前缀特性。但是如果只按moneys条件查询表,那么索引就不会被用到,具体如下:

				mysql> explain select * from sales2 where moneys=1\G;

				*************************** 1. row ***************************

						   id: 1

				  select_type: SIMPLE

						table: sales2

						 type: ALL

				possible_keys: NULL

						  key: NULL

					  key_len: NULL

						  ref: NULL

						 rows: 12

						Extra: Using where

				1 row in set (0.00 sec)

				ERROR:

				No query specified

		- (2) 对于使用like的查询,后面如果是常量并且只有%号不在第一个字符,索引才可能会被使用:

				mysql> create index ind_company2_name on company2(name);

				Query OK, 0 rows affected (0.07 sec)

				Records: 0  Duplicates: 0  Warnings: 0

				mysql> explain select * from company2 where name like '%ji'\G;

				*************************** 1. row ***************************

						   id: 1

				  select_type: SIMPLE

						table: company2

						 type: ALL

				possible_keys: NULL

						  key: NULL

					  key_len: NULL

						  ref: NULL

						 rows: 2

						Extra: Using where

				1 row in set (0.00 sec)

				ERROR:

				No query specified

				mysql> explain select * from company2 where name like 'xiao%'\G;

				*************************** 1. row ***************************

						   id: 1

				  select_type: SIMPLE

						table: company2

						 type: range

				possible_keys: ind_company2_name

						  key: ind_company2_name

					  key_len: 53

						  ref: NULL

						 rows: 2

						Extra: Using index condition

				1 row in set (0.00 sec)

				ERROR:

				No query specified

		- (3) 如果对打的文本进行搜索,使用全文索引而不用使用like '%...%'

		- (4) 如果列名是索引,使用column_name is null 将使用索引。如:

    			mysql> explain select * from company2 where name is null\G;
    
    			*************************** 1. row ***************************
    
    					   id: 1
    
    			  select_type: SIMPLE
    
    					table: company2
    
    					 type: ref
    
    			possible_keys: ind_company2_name
    
    					  key: ind_company2_name
    
    				  key_len: 53
    
    					  ref: const
    
    					 rows: 1
    
    					Extra: Using index condition
    
    			1 row in set (0.00 sec)

				ERROR:

				No query specified

	- 2>存在索引但不使用索引

        - (1) 如果MySQL估计使用索引比全表扫描更慢,则不使用索引。例如如果列key_part1均匀分布在1和100之间,下列查询中使用索引就不是很好:

				SELECT * FROM table_name where key_part1 > 1 and key_part1<90;

		- (2) 如果使用MEMEORY/HEAP表并且where条件中不使用'=' 进行索引列,那么不会用到索引。head表只有在'='的条件下才会使用索引

		- (3) 用or分割开的条件,如果or前的条件中的列有索引,而后面的列中没有索引,那么涉及到的索引都不会被用到

				mysql> show index from sales\G;

				*************************** 1. row ***************************

						Table: sales

				   Non_unique: 1

					 Key_name: ind_sales2_year

				 Seq_in_index: 1

				  Column_name: year

					Collation: A

				  Cardinality: 6

					 Sub_part: NULL

					   Packed: NULL

						 Null:

				   Index_type: BTREE

					  Comment:

				Index_comment:

				1 row in set (0.00 sec)

				ERROR:

				No query specified

				mysql> explain select * from sales where year=2006 or country='China'\G;

				*************************** 1. row ***************************

						   id: 1

				  select_type: SIMPLE

						table: sales

						 type: ALL

				possible_keys: ind_sales2_year

						  key: NULL

					  key_len: NULL

						  ref: NULL

						 rows: 12

						Extra: Using where

				1 row in set (0.00 sec)

				ERROR:

				No query specified

		- (4) 如果不是索引列的第一部分

				mysql> explain select * from sales2 where moneys = 1\G;

				*************************** 1. row ***************************

				id: 1

				select_type: SIMPLE

				table: sales2

				type: ALL

				possible_keys: NULL

				key: NULL

				key_len: NULL

				ref: NULL

				rows: 1000

				Extra: Using where

		- (5) 如果like 以%开始

				mysql> explain select * from company2 where name like '%3'\G;

				*************************** 1. row ***************************

				id: 1

				select_type: SIMPLE

				table: company2

				type: ALL

				possible_keys: NULL

				key: NULL

				key_len: NULL

				ref: NULL

				rows: 1000

				Extra: Using where

				1 row in set (0.00 sec)

		- (6)如果列类型是字符串，那么一定记得在 where 条件中把字符常量值用引号引起来，否则的话即便这个列上有索引，MySQL 也不会用到的，因为，MySQL 默认把输入的常量值进行转换以后才进行检索。 如下面的例子中 company2 表中的 name字段是字符型的，但是 SQL 语句中的条件值 294 是一个数值型值，因此即便在 name 上有索引， MySQL 也不能正确地用上索引，而是继续进行全表扫描。

				mysql> explain select * from company2 where name=299\G;

				*************************** 1. row ***************************

						   id: 1

				  select_type: SIMPLE

						table: company2

						 type: ALL

				possible_keys: ind_company2_name

						  key: NULL

					  key_len: NULL

						  ref: NULL

						 rows: 2

						Extra: Using where

				1 row in set (0.01 sec)

				ERROR:

				No query specified

				mysql> explain select * from company2 where name='299'\G;

				*************************** 1. row ***************************

						   id: 1

				  select_type: SIMPLE

						table: company2

						 type: ref

				possible_keys: ind_company2_name

						  key: ind_company2_name

					  key_len: 53

						  ref: const

						 rows: 1

						Extra: Using index condition

				1 row in set (0.00 sec)

				ERROR:

				No query specified

    - 4> 查看索引的使用情况  
        如果索引正在工作，Handler_read_key 的值将很高，这个值代表了一个行被索引值读的次数，很低的值表明增加索引得到的性能改善不高，因为索引并不经常使用。Handler_read_rnd_next 的值高则意味着查询运行低效，并且应该建立索引补救。这个值的含义是在数据文件中读下一行的请求数。如果正进行大量的表扫描，

			mysql> show status like 'Handler_read%';

			+-----------------------+-------+

			| Variable_name         | Value |

			+-----------------------+-------+

			| Handler_read_first    | 0     |

			| Handler_read_key      | 0     |

			| Handler_read_last     | 0     |

			| Handler_read_next     | 0     |

			| Handler_read_prev     | 0     |

			| Handler_read_rnd      | 0     |

			| Handler_read_rnd_next | 48    |

			+-----------------------+-------+

			7 rows in set (0.00 sec)

- 3 两个简单的优化实例

    - 1>定期分析表和检查表

		- 分析表:   
			 分析表的语法如下：

			    ANALYZE [LOCAL | NO_WRITE_TO_BINLOG] TABLE tbl_name [, tbl_name] ...

            本语句用于分析和存储表的关键字分布，分析的结果将可以使得系统得到准确的统计信息，使得 SQL 能够生成正确的执行计划。

				mysql> analyze table sales;

				+------------+---------+----------+----------+

				| Table      | Op      | Msg_type | Msg_text |

				+------------+---------+----------+----------+

				| test.sales | analyze | status   | OK       |

				+------------+---------+----------+----------+

				1 row in set (0.05 sec)

		- 检查表:

				CHECK TABLE tbl_name [, tbl_name] ... [option] ... option = {QUICK | FAST | MEDIUM | EXTENDED| CHANGED}

			检查表的作用是检查一个或多个表是否有错误

				mysql> check table sales;

				+------------+-------+----------+----------+

				| Table      | Op    | Msg_type | Msg_text |

				+------------+-------+----------+----------+

				| test.sales | check | status   | OK       |

				+------------+-------+----------+----------+

				1 row in set (0.03 sec)
			- (1)首先创建一个视图\c

					mysql> create view sales_views3 as select * from sales2;

					Query OK, 0 rows affected (0.10 sec)

			- (2)然后CHECK 一下该试图，发现没有问题\c

					mysql> check table sales_views3;

					+-------------------+-------+----------+----------+

					| Table             | Op    | Msg_type | Msg_text |

					+-------------------+-------+----------+----------+

					| test.sales_views3 | check | status   | OK       | 

					+-------------------+-------+----------+----------+

					1 row in set (0.05 sec)

			- (3)现在删除试图依赖的表\c

					mysql> drop table sales2; 

					Query OK, 0 rows affected (0.07 sec)

			- (4)在来check一下刚才的试图发现报错了\c

					mysql> check table sales_views3\G;

					*************************** 1. row ***************************

					   Table: test.sales_views3

						  Op: check

					Msg_type: Error

					Msg_text: Table 'test.sales2' doesn't exist

					*************************** 2. row ***************************

					   Table: test.sales_views3

						  Op: check

					Msg_type: Error

					Msg_text: View 'test.sales_views3' references invalid table(s) or column(s) or function(s) or definer/invoker of view lack rights to use them

					*************************** 3. row ***************************

					   Table: test.sales_views3

						  Op: check

					Msg_type: error

					Msg_text: Corrupt

					3 rows in set (0.04 sec)

					ERROR: 

					No query specified

	- 2>优化表：   
 
     优化表的语法如下：

		    OPTIMIZE [LOCAL | NO_WRITE_TO_BINLOG] TABLE tbl_name [, tbl_name] ...

		如果已经删除了表的一大部分，或者如果已经对含有可变长度行的表（含有 VARCHAR、BLOB 或 TEXT 列的表）进行了很多更改，则应使用 OPTIMIZE TABLE 命令来进行表优化。这个命令可以将表中的空间碎片进行合并，并且可以消除由于删除或者更新造成的空间浪费，但OPTIMIZE TABLE 命令只对 MyISAM、BDB 和 InnoDB 表起作用。

			mysql> optimize table sales3;

			+-------------+----------+----------+-------------------------------------------------------------------+

			| Table       | Op       | Msg_type | Msg_text                                                          |

			+-------------+----------+----------+-------------------------------------------------------------------+

			| test.sales3 | optimize | note     | Table does not support optimize, doing recreate + analyze instead | 

			| test.sales3 | optimize | status   | OK                                                                | 

			+-------------+----------+----------+-------------------------------------------------------------------+

			2 rows in set (0.14 sec)

+ 4 常用sql优化

    - 1） 大批量插入数据：
		- 当用load命令导入数据的时候，适当的设置可以提高导入的速度 对于MYISAM存储引擎的表，可以通过一下方式，快速的导入大量的数据

				ALTER TABLE tbl_name DISABLE KEYS;

				loading the data

				ALTER TABLE tbl_name ENDABLE KEYS;

		- 对于InnoDB：  
		> （1） 因为INNODB类型的表是按照主键的顺序保存的，所以将导入的数据按照主键的顺序排序，可以有效的提高导入的效率

		> （2）在导入数据前执行 SET UNIQUE_CHECK =0,关闭唯一性校验，在导入结束后执行 SET UNIQUE_CHECKS＝1，恢复唯一性校验，可以提高导入的效率   

		> （3）如果应用使用自动提交的方式，建议在导入前执行 SET AUTOCOMMIT＝0 关闭自动提交，导入结束后在执行SET AUTOCOMMIT＝1

	- 2） 优化INSERT语句

		> （1） 如果同时从同一客户插入很多行，尽量使用多个指表的INSERT语句，这种方式将大大所见客户端与数据库之间的链接，关闭等消耗，似的效率比分开执行的单个insert语句快：

			insert into test values(1,2),(1,3),(1,4)...

        > （2）如果从不同客户插入很多行，能通过使用INSERT DELAYE语句得到更高的速度

		> （3）将索引文件和数据文件分在不同的磁盘上存放（利用建表中的选项）；

		> （4） 如果进行批量插入，可以增加爱bulk_insert_buffer_size变量值的方法来提高簌簌，但是这只能对MYISAM表使用

		> （5） 当从一个文本文件装载一个标时，使用LOAD DATA INFILE 这通常比使用很多INSERT语句快20倍

	- 3） 优化GROUP BY 语句

	> 如果查询包括GROUP BY 但用户想要避免排序结果的消耗，则可以指定ORDER BY NULL禁止排序，

	- 4） 优化ORDER BY 语句

	> 在某些情况中，mysql可以使用一个索引来满足ORDER BY 自句，而不需要额外的排序

	- 5） 优化嵌套查询	

	- 6）or语句
    - 7）使用sql提示

		- （1） use index   
        	在查询语句中表名的后面，添加use index 来提供希望mysql去参考的索引列表，就可以让mysql不再考虑其他可用的索引

				mysql> explain select * from sales3 use index (ind_companyid) where company_id=2\G;

				*************************** 1. row ***************************

						   id: 1

				  select_type: SIMPLE

						table: sales3

						 type: ref

				possible_keys: ind_companyid

						  key: ind_companyid

					  key_len: 5

						  ref: const

						 rows: 1

						Extra: NULL

				1 row in set (0.05 sec)

				ERROR: 

				No query specified

		- （2）IGNORE INDEX	
            如果用户只是单纯地想让mysql忽略一个或多个索引，则可以使用IGNORE INDEX 作为HINT 同样是上面的例子，这次来看一下查询过程忽略索引：

				mysql> explain select * from sales3 ignore index (ind_companyid) where company_id=2\G;

				*************************** 1. row ***************************

						   id: 1

				  select_type: SIMPLE

						table: sales3

						 type: ALL

				possible_keys: NULL

						  key: NULL

					  key_len: NULL

						  ref: NULL

						 rows: 12

						Extra: Using where

				1 row in set (0.04 sec)

				ERROR: 

				No query specified

		- （3） FORCE INDEX    
            为牵制MYSQL使用一个特定的索引，可在查询中使用FORCE INDEX 作为HINT

				mysql> explain select * from sales3 where company_id > 0\G;

				*************************** 1. row ***************************

						   id: 1

				  select_type: SIMPLE

						table: sales3

						 type: ALL

				possible_keys: ind_companyid

						  key: NULL

					  key_len: NULL

						  ref: NULL

						 rows: 12

						Extra: Using where

				1 row in set (0.04 sec)

				ERROR: 

				No query specified
                
				mysql> explain select * from sales3 force index (ind_companyid) where company_id > 0\G;

				*************************** 1. row ***************************

						   id: 1

				  select_type: SIMPLE

						table: sales3

						 type: range

				possible_keys: ind_companyid

						  key: ind_companyid

					  key_len: 5

						  ref: NULL

						 rows: 12

						Extra: Using index condition

				1 row in set (0.03 sec)

				ERROR: 

				No query specified
