---
title: mysql触发器
date: 2016-07-12 13:34:08 
tags: MySql
category: MySQL
---
#### 创建触发器:
		CREATE TRIGGER trigger_name trigger_event ON tbl_name FOR EACH ROW trigger_tmt;
> 注意:
			触发器只能创建在永久表(Paymenent Table)上,不能对临时表(Temporary Table)创建触发器。
		
		
		DELIMITER $$
		CREATE TRIGGER ins_film
		AFTER INSERT ON film FOR EACH ROW BEGIN
		INSERT INTO film_text (film_id, title, description)
		VALUES (new.film_id, new.title, new.description);
		END;
		$$
		delimiter ;
		
	对于有重复记录，需要进行 UPDATE 操作的 INSERT，触发器触发的顺序是 BEFORE INSERT、BEFORE UPDATE、AFTER UPDATE；对于没有重复记录的 INSERT，就是简单的执行 INSERT 操作，触发器触发的顺序是 BEFORE INSERT、AFTER INSERT。对于那些实际执行 UPDATE 操作的记录，仍然会执行 BEFORE INSERT 触发器的内容，在设计触发器的时候一定要考虑这种情况，避免错误地触发了触发器。

#### 删除触发器:

		DROP TRIGGER INS_FILM;
		
#### 查看触发器:
		mysql> show triggers \G;
		*************************** 1. row ***************************
					 Trigger: ins_film
					   Event: INSERT
					   Table: film
				   Statement: BEGIN
		INSERT INTO film_text (film_id, title, description)
		VALUES (new.film_id, new.title, new.description);
		END
					  Timing: AFTER
					 Created: NULL
					sql_mode: STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION
					 Definer: root@%
		character_set_client: utf8
		collation_connection: utf8_general_ci
		  Database Collation: latin1_swedish_ci
		1 row in set (0.00 sec)
		
		mysql> select * from information_schema.triggers where trigger_name like 'ins_film'\G;
		*************************** 1. row ***************************
				   TRIGGER_CATALOG: def
					TRIGGER_SCHEMA: test
					  TRIGGER_NAME: ins_film
				EVENT_MANIPULATION: INSERT
			  EVENT_OBJECT_CATALOG: def
			   EVENT_OBJECT_SCHEMA: test
				EVENT_OBJECT_TABLE: film
					  ACTION_ORDER: 0
				  ACTION_CONDITION: NULL
				  ACTION_STATEMENT: BEGIN
		INSERT INTO film_text (film_id, title, description)
		VALUES (new.film_id, new.title, new.description);
		END
				ACTION_ORIENTATION: ROW
					 ACTION_TIMING: AFTER
		ACTION_REFERENCE_OLD_TABLE: NULL
		ACTION_REFERENCE_NEW_TABLE: NULL
		  ACTION_REFERENCE_OLD_ROW: OLD
		  ACTION_REFERENCE_NEW_ROW: NEW
						   CREATED: NULL
						  SQL_MODE: STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION
						   DEFINER: root@%
			  CHARACTER_SET_CLIENT: utf8
			  COLLATION_CONNECTION: utf8_general_ci
				DATABASE_COLLATION: latin1_swedish_ci
		1 row in set (0.23 sec)

		ERROR:
		No query specified
		
#### 触发器的使用
 ##### 触发器执行的语句有以下两个限制。  
 +  1 触发程序不能调用将数据返回客户端的存储程序， 也不能使用采用 CALL 语句的动态 SQL语句，但是允许存储程序通过参数将数据返回触发程序。也就是存储过程或者函数通过 OUT或者 INOUT 类型的参数将数据返回触发器是可以的，但是不能调用直接返回数据的过程。  
+ 2 不能在触发器中使用以显式或隐式方式开始或结束事务的语句， 如 START TRANSACTION、COMMIT 或 ROLLBACK。MySQL 的触发器是按照 BEFORE 触发器、行操作、AFTER 触发器的顺序执行的，其中任何一步操作发生错误都不会继续执行剩下的操作。如果是对事务表进行的操作，那么会整个作为一个事务被回滚（Rollback），但是如果是对非事务表进行的操作，那么已经更新的记录将无法回滚，这也是设计触发器的时候需要注意的问题。
