---
title: SQL Mode
date: 2016-07-12 14:12:08 
tags: MySql
category: MySQL
---
#### 简介:
+ 1>通过设置SQL Mode,可以王城不同严格程度的数据校验,有效的保障数据准确性。
+ 2>通过设置SQL Mode为ANSI模式,来保证大多数SQL符合标准的SQL语法,这样应用在不同数据库之间进行迁移时,则不需要对业务SQL进行较大的修改
+ 3>在不同数据库之间进行数据迁移之前,通过设置SLQ Mode可以使用MySQL 上的数据更方便的迁移到目标数据库中   

   例子:
   
    >在Mysql5.0 上,查询默认的SLQ Mode(sql mode 参数)为: REAL_AS_FLOAT,PIPES_AS_CONCAT,ANSI_QUOTES,GNORE_SPACE和ANSI。在这种模式下允许插入超过字段长度的值,只是在插入后,MySQL会返回一个warning。通过修改sql_mode为STRICT_TRANS_TABLES(严格模式)是吸纳了数据的严格校验,使错误数据不能插入表中,从而保证数据的准确性

    1) 查看默认的SQL Mode:	

		mysql> select @@sql_mode;

		+--------------------------------------------+

		| @@sql_mode                                 |

		+--------------------------------------------+

		| STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION |

		+--------------------------------------------+

		1 row in set (0.00 sec)

    2) 由于上面的sql mode 是strict_trans_tables 所以insert失败:ERROR 1406 (22001): Data too long for column 'name' at row 1

		mysql> create table t(

			-> name varchar(20),

			-> email varchar(40)

			-> ) engine=innodb charset=utf8;

		Query OK, 0 rows affected (0.13 sec)

		mysql> 查看测试表t的表结构:\c

		mysql> desc t;

		+-------+-------------+------+-----+---------+-------+

		| Field | Type        | Null | Key | Default | Extra |

		+-------+-------------+------+-----+---------+-------+

		| name  | varchar(20) | YES  |     | NULL    |       |

		| email | varchar(40) | YES  |     | NULL    |       |

		+-------+-------------+------+-----+---------+-------+

		2 rows in set (0.00 sec)

		mysql> 在表t中插入一条记录,其中name故意超出了实际的定义值varchar(20):\c

		mysql> insert into t values('12340000000000000000999999','beijing@126.com');

		ERROR 1406 (22001): Data too long for column 'name' at row 1

    3) 修改sql_mode之后可以执行,但是字符被截断了

		mysql> set session sql_mode='REAL_AS_FLOAT,PIPES_AS_CONCAT,ANSI_QUOTES,IGNORE_SPACE,ANSI';

		Query OK, 0 rows affected (0.01 sec)

		mysql> select @@sql_mode;

		+-------------------------------------------------------------+

		| @@sql_mode                                                  |

		+-------------------------------------------------------------+

		| REAL_AS_FLOAT,PIPES_AS_CONCAT,ANSI_QUOTES,IGNORE_SPACE,ANSI |

		+-------------------------------------------------------------+

		1 row in set (0.00 sec)

		mysql> insert into t values('12340000000000000000999999','beijing@126.com');

		Query OK, 1 row affected, 1 warning (0.00 sec)

		mysql> show warnings;

		+---------+------+-------------------------------------------+

		| Level   | Code | Message                                   |

		+---------+------+-------------------------------------------+

		| Warning | 1265 | Data truncated for column 'name' at row 1 |

		+---------+------+-------------------------------------------+

		1 row in set (0.01 sec)

#### 校验日期数据合法性:

+  在 INSERT 或 UPDATE 过程中，如果 SQL MODE 处于 TRADITIONAL 模式， 运行 MOD(X，0)会产生错误，因为 TRADITIONAL 也属于严格模式，在非严格模式下 MOD(X，0)返回的结果是 NULL，所以在含有 MOD 的运算中要根据实际情况设定好 sql_mode。

    	mysql> set sql_mode='ANSI';
    
    	mysql> CREATE TABLE t1(i int);
    
    	Query OK, 0 rows affected (0.11 sec)
    
    	mysql> insert into t1 values(90%0);
    
    	Query OK, 1 row affected (0.04 sec)
    
    	mysql> select * from t1;
    
    	+------+
    
    	| i    |
    
    	+------+
    
    	| NULL | 
    
    	+------+
    
    	1 row in set (0.03 sec)

		mysql> set session sql_mode='TRADITIONAL';

		Query OK, 0 rows affected (0.04 sec)

		mysql> INSERT INTO t1 values(90%0);

		ERROR 1365 (22012): Division by 0
+ 启用 NO_BACKSLASH_ESCAPES 模式，使反斜线成为普通字符。在导入数据时，如果数据中含有反斜线字符，     启用 NO_BACKSLASH_ESCAPES 模式保证数据的正确性，是个不错的选择。

		mysql> set sql_mode='ANSI';   

		Query OK, 0 rows affected (0.04 sec)

		mysql> select @@sql_mode;

		+-------------------------------------------------------------+

		| @@sql_mode                                                  |

		+-------------------------------------------------------------+

		| REAL_AS_FLOAT,PIPES_AS_CONCAT,ANSI_QUOTES,IGNORE_SPACE,ANSI | 

		+-------------------------------------------------------------+

		1 row in set (0.04 sec)

		mysql> drop table t;

		Query OK, 0 rows affected (0.06 sec)

		mysql> create table t (context varchar(20));

		Query OK, 0 rows affected (0.09 sec)

		mysql> insert into t values('\beijing');

		Query OK, 1 row affected (0.04 sec)

		mysql> select * from t;

		+---------+

		| context |

		+---------+

		|eijing | 

		+---------+

		1 row in set (0.04 sec)

		mysql> insert into t values('\\beijing');

		Query OK, 1 row affected (0.04 sec)

		mysql> select * from t;

		+----------+

		| context  |

		+----------+

		|eijing  | 

		| \beijing | 

		+----------+

		2 rows in set (0.04 sec)

		mysql> set sql_mode='NO_BACKSLASH_ESCAPES';Query OK, 0 rows affected (0.04 sec)

		mysql> SELECT @@SQL_MODE;

		+----------------------+

		| @@SQL_MODE           |

		+----------------------+

		| NO_BACKSLASH_ESCAPES | 

		+----------------------+

		1 row in set (0.03 sec)

		mysql> set sql_mode='REAL_AS_FLOAT,PIPES_AS_CONCAT,ANSI_QUOTES,IGNORE_SPACE,ANSI,NO_BACKSLASH_ESCAPES';

		Query OK, 0 rows affected (0.03 sec)

		mysql> select @@sql_mode;

		+----------------------------------------------------------------------------------+

		| @@sql_mode                                                                       |

		+----------------------------------------------------------------------------------+

		| REAL_AS_FLOAT,PIPES_AS_CONCAT,ANSI_QUOTES,IGNORE_SPACE,ANSI,NO_BACKSLASH_ESCAPES | 

		+----------------------------------------------------------------------------------+

		1 row in set (0.04 sec)

		mysql> insert into t values('\\beijing');

		Query OK, 1 row affected (0.04 sec)

		mysql> select * from t;         

		+-----------+

		| context   |

		+-----------+

		|eijing   | 

		| \beijing  | 

		| \\beijing | 

+ 启用 PIPES_AS_CONCAT 模式。将“||”视为字符串连接操作符，在 Oracle 等数据库中， ||”被视为字符串的连接操作符，所以，在其他数据库中含有“||”操作符的 SQL在 MySQL 中将无法执行，为了解决这个问题，MySQL 提供了 PIPES_AS_CONCAT 模式。

		mysql> set sql_mode='ansi';

		Query OK, 0 rows affected (0.04 sec)

		mysql> select @@sql_mode;

		+-------------------------------------------------------------+

		| @@sql_mode                                                  |

		+-------------------------------------------------------------+

		| REAL_AS_FLOAT,PIPES_AS_CONCAT,ANSI_QUOTES,IGNORE_SPACE,ANSI | 

		+-------------------------------------------------------------+

		1 row in set (0.04 sec)

		mysql> select 'bejing'||'2008';

		+------------------+

		| 'bejing'||'2008' |

		+------------------+

		| bejing2008       | 

		+------------------+

		1 row in set (0.03 sec)

### 常用的sql_mode

> sql_mode 值&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;描述

> ANSI&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;等同于 REAL_AS_FLOAT、PIPES_AS_CONCAT、ANSI_QUOTES、IGNORE_SPACE 和 ANSI组合模式，这种模式使语法和行为更符合标准的SQL   

> STRICT_TRANS_TABLES &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;适用于事务表和非事务表，它是严格模式，不允许非法日期，STRICT_TRANS_TA也不允许超过字段长度的值插入字段中，对于插入不正确的值给出错误而不是警告

> BLES&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;TRADITIONAL 模式等同于 STRICT_TRANS_TABLES、STRICT_ALL_TABLES、TRADITIONAL

>  NO_ZERO_IN_DATE、NO_ZERO_DATE、ERROR_FOR_DIVISION_BY_ZERO、TRADITIONAL 和 NO_AUTO_CREATE_USER 组合模式，所以它也是严格模式，对于插入不正确的值是给出错误而不是警告。可以应用在事务表和非事务表，用在事务表时，只要出现错误  就会立即回滚
		
    mysql> show create table actor\G;

		*************************** 1. row ***************************

			   Table: actor

		Create Table: CREATE TABLE `actor` (

		  `actor_id` smallint(6) NOT NULL,

		  `first_name` varchar(50) DEFAULT NULL,

		  `last_name` varchar(50) DEFAULT NULL,

		  PRIMARY KEY (`actor_id`)

		) ENGINE=InnoDB DEFAULT CHARSET=utf8

		1 row in set (0.04 sec)

	ERROR: 

	No query specified

	mysql> set session sql_mode='ansi';

	Query OK, 0 rows affected (0.07 sec)

	mysql> show create table actor\G;

	*************************** 1. row ***************************

		   Table: actor

	Create Table: CREATE TABLE "actor" (

	  "actor_id" smallint(6) NOT NULL,

	  "first_name" varchar(50) DEFAULT NULL,

	  "last_name" varchar(50) DEFAULT NULL,

	  PRIMARY KEY ("actor_id")

	)

	1 row in set (0.04 sec)

	ERROR: 

	No query specified
