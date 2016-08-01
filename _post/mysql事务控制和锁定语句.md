---
title: mysql事务控制和锁定语句 
date: 2016-07-12 13:34:08 
tags: MySql
category: MySQL
---
#### 事务控制和锁定语句  
> LOCK TABLES 和 UNLOCK TABLES:   LOCK TABLES 可以锁定用于当前线程的表。如果表被其他线程锁定，则当前线程会等待，直到可以获取所有锁定为止。UNLOCK TABLES 可以释放当前线程获得的任何锁定。当前线程执行另一个 LOCK TABLES 时,或当与服务器的连接被关闭时，所有由当前线程锁定的表被隐含地解锁，具体语法如下：

	LOCK TABLES

	tbl_name [AS alias] {READ [LOCAL] | [LOW_PRIORITY] WRITE}

	[, tbl_name [AS alias] {READ [LOCAL] | [LOW_PRIORITY] WRITE}] ...

	

	UNLOCK TABLES
    
+ 例子:

	session1

		mysql> 获得表film_text的READ锁\c

		mysql> lock table film_text read;

		Query OK, 0 rows affected (0.00 sec)

		mysql> select * from film_text;

		+---------+----------+----------------------------------------------------------------+

		| film_id | title    | description                                                    |

		+---------+----------+----------------------------------------------------------------+

		|       1 | TestLock | A Epic Drama of a Feminist And a Mad Scientist who must Battle |

		+---------+----------+----------------------------------------------------------------+

		1 row in set (0.00 sec)

	session2

		mysql> select * from film_text;

		+---------+----------+----------------------------------------------------------------+

		| film_id | title    | description                                                    |

		+---------+----------+----------------------------------------------------------------+

		|       1 | TestLock | A Epic Drama of a Feminist And a Mad Scientist who must Battle |

		+---------+----------+----------------------------------------------------------------+

		1 row in set (0.00 sec)
        
		mysql> 其他session更新锁定表会等待获得锁\c

		mysql> update film_text set title='Lock Test' where film_id=1;

		waiting...	

	session1

		mysql> 释放锁\c

		mysql> unlock tables;

		Query OK, 0 rows affected (0.00 sec)

	session2

		mysql> update film_text set title='Lock Test' where film_id=1;

		Query OK, 1 row affected (1 min 1.75 sec)

		Rows matched: 1  Changed: 1  Warnings: 0



#### 事务控制
MySQL 通过 SET AUTOCOMMIT、START TRANSACTION、COMMIT 和 ROLLBACK 等语句支持本地事务，具体语法如下。

	START TRANSACTION | BEGIN [WORK]

	COMMIT [WORK] [AND [NO] CHAIN] [[NO] RELEASE]

	ROLLBACK [WORK] [AND [NO] CHAIN] [[NO] RELEASE]

	SET AUTOCOMMIT = {0 | 1}
    
+ 例子:  
	session1 

		mysql> 从表actor中查询actor_id=3的记录,结果为空\c

		mysql> select * from actor where actor_id=3;

		Empty set (0.00 sec
	session2

		mysql> 从表actor中查询actor_id=3的记录,结果为空\c

		mysql> select * from actor where actor_id=3;

		Empty set (0.00 sec)
	session1--->

		mysql> 用start transaction命令启动一个事务,往表actor中插入一条记录,没有commit:\c

		mysql> start transaction;

		Query OK, 0 rows affected (0.00 sec)

		mysql> insert into actor(actor_id, first_name, last_name) values(3,'Chen','qizhou');

		Query OK, 1 row affected (0.00 sec)

	session2--->

		mysql> 查询actor,结果仍然为空\c

		mysql> select * from actor where actor_id=3;

		Empty set (0.00 sec)

	session1--->

		mysql> 执行提交\c

		mysql> commit;

		Query OK, 0 rows affected (0.01 sec)

	session2--->

		mysql> 再次查询表actor\c

		mysql> select *  from actor where actor_id=3;

		+----------+------------+-----------+

		| actor_id | first_name | last_name |

		+----------+------------+-----------+

		|        3 | Chen       | qizhou    |

		+----------+------------+-----------+

		1 row in set (0.00 sec)

	session1--->

		mysql> 这个事务是自动提交执行的\c

		mysql> insert into actor(actor_id,first_name,last_name) values(4, 'Lisa', 'Lan');

		Query OK, 1 row affected (0.00 sec)

	session2--->

		再次查询表actor

		mysql> select * from actor where actor_id=4;

		+----------+------------+-----------+

		| actor_id | first_name | last_name |

		+----------+------------+-----------+

		|        4 | Lisa       | Lan       |

		+----------+------------+-----------+

		1 row in set (0.00 sec)

	session1--->

		mysql> 重新用 start transaction启动一个事务\c

		mysql> start transaction;

		Query OK, 0 rows affected (0.00 sec)

		mysql> 往表actor中插入一条记录\c

		mysql> insert into actor(actor_id,first_name,last_name) values(5,'Lisa', 'TT');

		Query OK, 1 row affected (0.00 sec)

		mysql> 用commit and chain 命令提交\c

		mysql> commit and chain;

		Query OK, 0 rows affected (0.01 sec)

		mysql> 此时自动开始一个新的事务\c

		mysql> insert into actor(actor_id,first_name,last_name) values(6,'Lisa', 'Mou');

		Query OK, 1 row affected (0.00 sec)

	session2--->

		mysql> session1 刚插入的记录无法看到:\c

		mysql> select * from actor where first_name='Lisa';

		+----------+------------+-----------+

		| actor_id | first_name | last_name |

		+----------+------------+-----------+

		|        4 | Lisa       | Lan       |

		|        5 | Lisa       | TT        |

		+----------+------------+-----------+

		2 rows in set (0.00 sec)

	session1--->

		mysql> 用commit命令提交\c

		mysql> commit;

		Query OK, 0 rows affected (0.01 sec)

	session2--->

		mysql> select * from actor where first_name='Lisa';

		+----------+------------+-----------+

		| actor_id | first_name | last_name |

		+----------+------------+-----------+

		|        4 | Lisa       | Lan       |

		|        5 | Lisa       | TT        |

		|        6 | Lisa       | Mou       |

		+----------+------------+-----------+

		3 rows in set (0.00 sec)
	如果在锁表期间，用 start transaction 命令开始一个新事务，会造成一个隐含的 unlock tables 被执行,

	如:

	session1--->

		mysql> 从表中查询actor_id=7的记录,结果为空\c

		mysql> select * from actor where actor_id=7;

		Empty set (0.00 sec)

	session2--->

		mysql> 从表中查询actor_id=7的记录,结果为空\c

		mysql> select * from actor where actor_id=7;

		Empty set (0.00 sec)

	session1--->

		mysql> 对actor加写锁\c

		mysql> lock table actor write;

		Query OK, 0 rows affected (0.00 sec)

	session2--->

		mysql> 对表actor的读操作被阻塞\c

		mysql> select * from actor where actor_id=7;

		waiting....

	session1--->

		mysql> 插入一条记录\c

		mysql> insert into actor(actor_id,first_name,last_name) values(7,'Lisa', 'Tom');

		Query OK, 1 row affected (0.01 sec)

		mysql> 回滚到刚才的记录\c

		mysql> rollback;

		Query OK, 0 rows affected (0.00 sec)

		mysql> 用start transaction命令重新开始一个事务\c

		mysql> start transaction;

		Query OK, 0 rows affected (0.00 sec)

	session2--->

		mysql> select * from actor where actor_id=7;

		+----------+------------+-----------+

		| actor_id | first_name | last_name |

		+----------+------------+-----------+

		|        7 | Lisa       | Tom       |

		+----------+------------+-----------+

		1 row in set (3 min 37.27 sec)
	以下例子就是模拟回滚事务的一个部分，通过定义 SAVEPOINT 来指定需要回滚的事务的位置。

	session1--->

		mysql> 从表actor中查询first_name='Simon'的记录结果为空\c

		mysql> select * from actor where first_name='Simon';

		Empty set (0.00 sec)

	session2--->

		mysql> 从表actor中查询first_name='Simon'的记录结果为空\c

		mysql> select * from actor where first_name='Simon';

		Empty set (0.00 sec)

	session1--->

		mysql> 启动一个事务,往表actor中插入一条记录\c

		mysql> start transaction;

		Query OK, 0 rows affected (0.00 sec)

		mysql> insert into actor(actor_id,first_name,last_name) values(8,'Simon','Tom');

		Query OK, 1 row affected (0.00 sec)

		mysql> 可以查到刚插入的记录\c

		mysql> select * from actor where first_name='Simon';

		+----------+------------+-----------+

		| actor_id | first_name | last_name |

		+----------+------------+-----------+

		|        8 | Simon      | Tom       |

		+----------+------------+-----------+

		1 row in set (0.00 sec)

	session2--->

		mysql> 无法从表actor中查到session1刚插入的记录\c

		mysql> select * from actor where first_name='Simon';

		Empty set (0.00 sec)

	session1--->

		mysql> savepoint test;

		Query OK, 0 rows affected (0.00 sec)
        
		mysql> insert into actor(actor_id,first_name,last_name) values(9,'Simon','Cof');

		Query OK, 1 row affected (0.00 sec)

		mysql> 可以查询到两条记录\c

		mysql> select * from actor where first_name='Simon';

		+----------+------------+-----------+

		| actor_id | first_name | last_name |

		+----------+------------+-----------+

		|        8 | Simon      | Tom       |

		|        9 | Simon      | Cof       |

		+----------+------------+-----------+

		2 rows in set (0.00 sec)

	session2--->

		mysql> 无法从表actor中查到session1刚插入的记录\c

		mysql> select * from actor where first_name='Simon';

		Empty set (0.00 sec)	

	session1--->

		mysql> 回滚到刚才定义的savepoint\c

		mysql> rollback to savepoint test;

		Query OK, 0 rows affected (0.02 sec)

		mysql> select * from actor where first_name='Simon';

		+----------+------------+-----------+

		| actor_id | first_name | last_name |

		+----------+------------+-----------+

		|        8 | Simon      | Tom       |

		+----------+------------+-----------+

		1 row in set (0.00 sec)

	session2--->

		mysql> 无法从表actor中查到记录

		mysql> select * from actor where first_name='Simon';

		Empty set (0.00 sec)

	session1--->

		提交只能查到一个数据

		mysql> commit;

		Query OK, 0 rows affected (0.01 sec)
    
        mysql> select * from actor where first_name='Simon';

		+----------+------------+-----------+

		| actor_id | first_name | last_name |

		+----------+------------+-----------+

		|        8 | Simon      | Tom       |

		+----------+------------+-----------+

		1 row in set (0.00 sec)

	session2--->

		也可以查到一条数据

		mysql> select * from actor where first_name='Simon';

		+----------+------------+-----------+

		| actor_id | first_name | last_name |

		+----------+------------+-----------+

		|        8 | Simon      | Tom       |

		+----------+------------+-----------+

		1 row in set (0.00 sec)

