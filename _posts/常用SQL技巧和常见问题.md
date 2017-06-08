---
title: 常用SQL技巧和常见问题
date: 2016-07-12 14:12:08 
tags: MySql
category: MySQL
---
#### 1 正则表达式的使用

	^		    在字符串开始处进行匹配

	$			在字符串的末尾进行匹配

	.			在匹配任意单个字符，包括换行符

	[...]		匹配出括号内的任意字符

	[^...]		匹配不出括号内的任意字符

	a*			匹配0个或多个a（包括空串）

	a+			匹配1个或多个a（不包换空串）		

	a？         匹配 1 个或零个 a

	a1|a2       匹配 a1 或 a2

	a(m)        匹配 m 个 a

	a(m,)       匹配 m 个或更多个 a

	a(m,n)      匹配 m 到 n 个 a

	a(,n)       匹配 0 到 n 个 a

	（…）       将模式元素组成单一元素
	
> 
        mysql> select 'abcdefg' REGEXP 'g$';
    	+-----------------------+
    	| 'abcdefg' REGEXP 'g$' |
    	+-----------------------+
    	|                     1 | 
    	+-----------------------+
    	1 row in set (0.10 sec)
    	mysql> select 'tabcgefg' REGEXP 'a';
    	+-----------------------+
    	| 'tabcgefg' REGEXP 'a' |
    	+-----------------------+
    	|                     1 | 
    	+-----------------------+
    	1 row in set (0.04 sec)

+ 使用：

    	mysql> select name, email from t where email REGEXP '@163[.,]com$';
    
    	+------------+--------------------+
    
    	| name       | email              |
    
    	+------------+--------------------+
    
    	| beijing    | beijing@163.com    | 
    
    	| beijing126 | beijin125g@163.com | 
    
    	| beijing188 | beijin1g@163.com   | 
    
    	+------------+--------------------+
    
    	3 rows in set (0.04 sec)

+ 巧用rand() 提取随机行

    	 mysql> select * from t order by rand() limit 1;
    
    	+---------+-----------------+
    
    	| name    | email           |
    
    	+---------+-----------------+
    
    	| beijing | beijing@163.com | 
    
    	+---------+-----------------+
    
    	1 row in set (0.04 sec)



+ 利用Group by 的with rollup自句做统计  
使用GROUP BY的WITH ROLLUP字句可以检索出更多的分组聚合信息，它不仅仅能像一般的GROUP BY语句那样检索出各组的聚合信息，还能检索出本组类的整体聚合信息，具体如下例所示。

		mysql> select year, country, product, sum(profit) from sales group by year, country, product;

		+------+---------+---------+-------------+

		| year | country | product | sum(profit) |

		+------+---------+---------+-------------+

		| 2004 | china   | tnt1    |        2001 | 

		| 2004 | china   | tnt2    |        2002 | 

		| 2004 | china   | tnt3    |        2003 | 

		| 2005 | china   | tnt1    |        4011 | 

		| 2005 | china   | tnt2    |        4013 | 

		| 2005 | china   | tnt3    |        4015 | 

		| 2006 | china   | tnt1    |        2010 | 

		| 2006 | china   | tnt2    |        2011 | 

		| 2006 | china   | tnt3    |        2012 | 

		+------+---------+---------+-------------+

		9 rows in set (0.05 sec)

		mysql> select year, country, product, sum(profit) from sales group by year, country, product with rollup;

		+------+---------+---------+-------------+

		| year | country | product | sum(profit) |

		+------+---------+---------+-------------+

		| 2004 | china   | tnt1    |        2001 | 

		| 2004 | china   | tnt2    |        2002 | 

		| 2004 | china   | tnt3    |        2003 | 

		| 2004 | china   | NULL    |        6006 | 

		| 2004 | NULL    | NULL    |        6006 | 

		| 2005 | china   | tnt1    |        4011 | 

		| 2005 | china   | tnt2    |        4013 | 

		| 2005 | china   | tnt3    |        4015 | 

		| 2005 | china   | NULL    |       12039 | 

		| 2005 | NULL    | NULL    |       12039 | 

		| 2006 | china   | tnt1    |        2010 | 

		| 2006 | china   | tnt2    |        2011 | 

		| 2006 | china   | tnt3    |        2012 | 

		| 2006 | china   | NULL    |        6033 | 

		| 2006 | NULL    | NULL    |        6033 | 

		| NULL | NULL    | NULL    |       24078 | 

		+------+---------+---------+-------------+

		16 rows in set (0.06 sec)

+ 用 BIT GROUP FUNCTIONS 做统计

		mysql> create table order_rab(id int, customer_id int, kind int);

		Query OK, 0 rows affected (0.13 sec)

		mysql> insert into order_rab values(1,1,5),(2,1,5),(3,2,3),(4,2,4);

		Query OK, 4 rows affected (0.04 sec)

		Records: 4  Duplicates: 0  Warnings: 0

		mysql> select * from order_rab;

		+------+-------------+------+

		| id   | customer_id | kind |

		+------+-------------+------+

		|    1 |           1 |    5 | 

		|    2 |           1 |    5 | 

		|    3 |           2 |    3 | 

		|    4 |           2 |    4 | 

		+------+-------------+------+

		4 rows in set (0.03 sec)

		mysql> select customer_id,bit_or(kind) from order_rab group by customer_id;

		+-------------+--------------+

		| customer_id | bit_or(kind) |

		+-------------+--------------+

		|           1 |            5 | 

		|           2 |            7 | 

		+-------------+--------------+

		2 rows in set (0.05 sec)

		mysql> select customer_id,bit_and(kind) from order_rab group by customer_id;

		+-------------+---------------+

		| customer_id | bit_and(kind) |

		+-------------+---------------+

		|           1 |             5 | 

		|           2 |             0 | 

		+-------------+---------------+

		2 rows in set (0.04 sec)

#### 数据库名,表名的大小写问题:

操作系统是CentOS。部分服务器的操作系统又是Windows。 出现了一个小毛病。那就是MySQL大小写的问题。 在CentOS安装的MySQL的配置文件中(/etc/my.cnf)，是没有lower_case_table_names=1这行的。 在Windows安装的MySQL的配置文件中(my.ini)，是有lower_case_table_names=1这行的。 lower_case_table_names=1的用途是让MySQL实现不区分大小写。 所以当时出了些毛病，后来才发现是这个的问题。连忙在CentOS中的my.cnf(/etc/my.cnf)的[mysqld]区段下增加: lower_case_table_names=1


#### 使用外键需要注意的问题

在MySQL中，INNODB 存储引擎支持对外部关键字约束条件的检查，而对于其他类型存储的表，当使用references tabl_name(col_name)自句定义列时可以使用外部关键字，但是该自句没有实际的效果，之作为备忘录或注释来提醒用户目前正定义的列指向另一个表中的一个列。
        
+ myisam表外键不起作用:
    
    	mysql> create table users(id int, name varchar(10), primary key(id))engin=myisam;
    
    	ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'engin=myisam' at line 1
    
    	mysql> create table users(id int, name varchar(10), primary key(id))engine=myisam;
    
    	mysql> create table books(id int, bookname varchar(10), userid int,primary key(id),constraint fk_userid_id foreign key(userid) references users(id))engine=myisam;
    
    	Query OK, 0 rows affected (0.05 sec)

		mysql> insert into books values(1,'book1',1);

		Query OK, 1 row affected (0.04 sec)

+ innodb 表外键起作用

		mysql> create table users2(id int,name varchar(10), primary key(id)) engine=innodb;

		Query OK, 0 rows affected (0.10 sec)

		mysql> create table books2(id int, bookname varchar(10),userid int,primary key(id),constraint fk_userid_id foreign key(userid) references users2(id))engine=innodb;

		Query OK, 0 rows affected (0.13 sec)

		mysql> insert into books2 values(1,'book1',1);

		ERROR 1452 (23000): Cannot add or update a child row: a foreign key constraint fails (`test`.`books2`, CONSTRAINT `fk_userid_id` FOREIGN KEY (`userid`) REFERENCES `users2` (`id`))

		mysql> 

	- 查看：

    		mysql> show table status from test where name='books'\G;
    
    		*************************** 1. row ***************************
    
    				   Name: books
    
    				 Engine: InnoDB
    
    				Version: 10
    
    			 Row_format: Compact
    
    				   Rows: 0
    
    		 Avg_row_length: 0
    
    			Data_length: 16384
    
    		Max_data_length: 0
    
    		   Index_length: 16384
    
    			  Data_free: 0
    
    		 Auto_increment: NULL
    
    			Create_time: 2016-06-07 15:15:39
    
    			Update_time: NULL
    
    			 Check_time: NULL
    
    			  Collation: latin1_swedish_ci
    
    			   Checksum: NULL
    
    		 Create_options: 
    
    				Comment: 
    
    		1 row in set (0.03 sec)

			mysql> show create table users\G;

			*************************** 1. row ***************************

				   Table: users

			Create Table: CREATE TABLE `users` (

			  `id` int(11) NOT NULL DEFAULT '0',

			  `name` varchar(10) DEFAULT NULL,

			  PRIMARY KEY (`id`)

			) ENGINE=InnoDB DEFAULT CHARSET=latin1

			1 row in set (0.04 sec)

			ERROR: 

			No query specified
