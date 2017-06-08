---
title: mysql存储过程 
date: 2016-06-16 15:20:08 
tags: MySql
category: MySQL
---
#### 创建存储过程   

	DELIMITER $$

	CREATE PROCEDURE film_in_stock(IN p_film_id INT, IN p_store_id INT, OUT p_film_count INT)
		READS SQL DATA
		BEGIN
			SELECT inventory_id
			FROM inventory
			WHERE film_id = p_film_id
			AND store_id = p_store_id;
			SELECT FOUND_ROWS() INTO p_film_count;
		END $$
	DELIMITER ;
		
#### 执行:

    CALL film_in_stock(1,1,@a)
		select @a;
#### 删除:
		drop PROCEDURE film_in_stock;
#### 查看:
- 查看存储过程状态:
		
        mysql> show procedure status like 'film_in_stock'\G
		*************************** 1. row ***************************
						  Db: test
						Name: film_in_stock
						Type: PROCEDURE
					 Definer: root@%
					Modified: 2016-06-04 13:08:00
					 Created: 2016-06-04 13:08:00
			   Security_type: DEFINER
					 Comment:
		character_set_client: utf8
		collation_connection: utf8_general_ci
		  Database Collation: latin1_swedish_ci
		1 row in set (0.00 sec)
- 查看存储过程或者函数的定义
			
		mysql> show create procedure film_in_stock \G
		*************************** 1. row ***************************
				   Procedure: film_in_stock
					sql_mode: STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION
			Create Procedure: CREATE DEFINER=`root`@`%` PROCEDURE `film_in_stock`(IN p_film_id INT, IN p_store_id INT, OUT p_film_count INT)
							  READS SQL DATA
								BEGIN
									SELECT inventory_id
									FROM inventory
									WHERE film_id = p_film_id AND store_id = p_store_id;
									SELECT FOUND_ROWS() INTO p_film_count;
								END
		character_set_client: utf8
		collation_connection: utf8_general_ci
		  Database Collation: latin1_swedish_ci
		1 row in set (0.00 sec)
- 通过查看information_schema数据库中的routine表
- 
		mysql> select * from information_schema.routines where ROUTINE_NAME='film_in_stock'\G
		*************************** 1. row ***************************
				   SPECIFIC_NAME: film_in_stock
				 ROUTINE_CATALOG: def
				  ROUTINE_SCHEMA: test
					ROUTINE_NAME: film_in_stock
					ROUTINE_TYPE: PROCEDURE
					   DATA_TYPE:
		CHARACTER_MAXIMUM_LENGTH: NULL
		  CHARACTER_OCTET_LENGTH: NULL
			   NUMERIC_PRECISION: NULL
				   NUMERIC_SCALE: NULL
			  DATETIME_PRECISION: NULL
			  CHARACTER_SET_NAME: NULL
				  COLLATION_NAME: NULL
				  DTD_IDENTIFIER: NULL
					ROUTINE_BODY: SQL
			  ROUTINE_DEFINITION: BEGIN
										SELECT inventory_id
										FROM inventory
										WHERE film_id = p_film_id
										AND store_id = p_store_id;
										SELECT FOUND_ROWS() INTO p_film_count;
								END
				   EXTERNAL_NAME: NULL
			   EXTERNAL_LANGUAGE: NULL
				 PARAMETER_STYLE: SQL
				IS_DETERMINISTIC: NO
				 SQL_DATA_ACCESS: READS SQL DATA
						SQL_PATH: NULL
				   SECURITY_TYPE: DEFINER
						 CREATED: 2016-06-04 13:08:00
					LAST_ALTERED: 2016-06-04 13:08:00
						SQL_MODE: STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION
				 ROUTINE_COMMENT:
						 DEFINER: root@%
			CHARACTER_SET_CLIENT: utf8
			COLLATION_CONNECTION: utf8_general_ci
			  DATABASE_COLLATION: latin1_swedish_ci
		1 row in set (0.00 sec)

#### 变量的使用:
- 变量的定义:

		DECLARE last_month_start 
	作用只能在begin..end中
		
- 变量的赋值:
			
        SET last_month_start = DATE_SUB(CURRENT_DATE(), INTERVAL 1 MONTH);
		
			CREATE FUNCTION get_customer_balance(p_customer_id INT,p_effective_date DATETIME)
			RETURNS DECIMAL(5,2)
			DETERMINISTIC
			READS SQL DATA
			BEGIN
				...
				DECLARE v_payments DECIMAL(5,2)##sum of payments made previously
				...
				SELECT IFNULL(SUM(payment.amount),0) INTO v_payments
				FROM payment
				WHERE payment.payment_date <= p_effective_date
				AND payment.customer_id = p_customer_id;
			END $$

#### 条件的定义和处理:
- 条件的定义:
- 
		DECLARE condition_name CONDITION FOR coundition_value
		
		condition_value:
			SQLSTATE [VALUE] sqlstate_value
			| mysql_error_code
	
- 条件的处理:

		DECLARE handler_type HANDLER FOR condition_value[,...] sp_statement
			handler_type:
			CONTINUE
			| EXIT
			| UNDO
			condition_value:
				SQLSTATE [VALUE] sqlstate_value
				| condition_name
				| SQLWARNING
				| NOT FOUND
				| SQLEXCEPTION
				| mysql_error_code
	
    + 例子1:
			
			create table actor(
				actor_id SMALLINT NOT NULL PRIMARY KEY,
				first_name VARCHAR(50) ,
				last_name VARCHAR(50)
			)ENGINE=INNODB AUTO_INCREMENT=0 DEFAULT CHARSET=UTF8;
			
			
			mysql> insert into actor values(1,'WADE','SAXON');
			Query OK, 1 row affected (0.00 sec)
			
			mysql> delimiter $$
			mysql> CREATE PROCEDURE actor_insert()
				-> BEGIN
				->   SET @x=1;
				->   INSERT INTO actor(actor_id,first_name,last_name) values(2,'zhang','jiagang');
				->   SET @x=2;
				->   INSERT INTO actor(actor_id,first_name,last_name) VALUES(1,'Test','1');
				->   SET @x=3;
				-> END;
				-> $$
			Query OK, 0 rows affected (0.00 sec)

			mysql> DELIMITER ;
			mysql> call actor_insert();
			ERROR 1062 (23000): Duplicate entry '1' for key 'PRIMARY'
			mysql> select @x;
			+------+
			| @x   |
			+------+
			|    2 |
			+------+
			1 row in set (0.00 sec)
		从上例子看出运行抛出异常;

	+ 例子修改:
			
            mysql> delimiter $$
			mysql>
			mysql> CREATE PROCEDURE actor_insert ()
			-> BEGIN
			-> DECLARE CONTINUE HANDLER FOR SQLSTATE '23000' SET @x2 = 1;
			-> SET @x = 1;
			-> INSERT INTO actor(actor_id,first_name,last_name) VALUES (201,'Test','201');
			-> SET @x = 2;
			-> INSERT INTO actor(actor_id,first_name,last_name) VALUES (1,'Test','1');
			-> SET @x = 3;
			-> END;
			-> $$
			Query OK, 0 rows affected (0.00 sec)
			mysql> delimiter ;
        > 调用条件处理的过程，再遇到主键重的错误时，会按照定义的处理方式进行处理，由于例子中定义的是 CONTINUE，所以会继续执行下面的语句。    
		> handler_type 现在还只支持 CONTINUE 和 EXIT 两种，CONTINUE 表示继续执行下面的语句，EXIT 则表示执行终止，UNDO 现在还不支持。
#### 光标的使用:  
> 在存储过程和函数中可以使用光标对结果集进行循环的处理。
		
+ 声明光标:

		DELARE cursor_name CURSOR FOR select_statement
+ OPEN光标:

        OPEN cursor_name;
+ FETCH光标:
		
		FETCH cursor_name INTO var_name[,...]
+ CLOSEF光标:

		CLOSE cursor_name;
	> 以下例子是一个简单的使用光标的过程， 对 payment 表按照行进行循环的处理， 按照 staff_id 值的不同累加 amount 的值，判断循环结束的条件是捕获 NOT FOUND 的条件，当 FETCH 光 标找不到下一条记录的时候，就会关闭光标然后退出过程。   

    	mysql> delimiter $$
    	mysql>
    	mysql> CREATE PROCEDURE payment_stat ()
    		-> BEGIN
    		-> DECLARE i_staff_id int;
    		-> DECLARE d_amount decimal(5,2);
    		-> DECLARE cur_payment cursor for select staff_id,amount from payment;
    		-> DECLARE EXIT HANDLER FOR NOT FOUND CLOSE cur_payment;
    		->
    		-> set @x1 = 0;
    		-> set @x2 = 0;
    		->
    		-> OPEN cur_payment;
    		->
    		-> REPEAT
    		-> FETCH cur_payment INTO i_staff_id, d_amount;
    		-> if i_staff_id = 2 then
    		-> set @x1 = @x1 + d_amount;
    		-> else
    		-> set @x2 = @x2 + d_amount;
    		-> end if;
    		-> UNTIL 0 END REPEAT;
    		->
    		-> CLOSE cur_payment;
    		->
    		-> END;
    		-> $$
    	mysql> delimiter $$
#### 流程控制: 
可以使用IF CASE LOOP LEAVE ITERATE REPEAT 及 WHILE 语句进行流程的控制.
+ 1  IF:

		IF search_condition THEN statement_list
			[ELSEIF search_condition THEN statement_list] ...
			[ELSE statement_list]
		END IF
		
+ 2 CASE:

		CASE case_value
			WHEN when_value THEN statement_list
			[WHEN when_value THEN statement_list] ...
			[ELSE statement_list]
		END CASE
		Or:
		CASE
			WHEN search_condition THEN statement_list
			[WHEN search_condition THEN statement_list] ...
			[ELSE statement_list]
		END CASE
	> 在上文光标的使用例子中，IF 语句也可以使用 CASE 语句来完成：
	
		case
		when i_staff_id = 2 then
				set @x1 = @x1 + d_amount;
			else
				set @x2 = @x2 + d_amount;
		end case;
	或者：
	
		case i_staff_id
			when 2 then
				set @x1 = @x1 + d_amount;
			else
				set @x2 = @x2 + d_amount;
		end case;
		
+ 3 LOOP:  
		&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;LOOP 实现简单的循环，退出循环的条件需要使用其他的语句定义，通常可以使用 LEAVE 语句实现，具体语法如下：

		[begin_label:] LOOP
			statement_list
		END LOOP [end_label]
			
	如果不在 statement_list 中增加退出循环的语句，那么 LOOP 语句可以用来实现简单的死循环。
		
+ 4 LEVER:  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;用来从标注的流程构造中退出，通常和 BEGIN ... END 或者循环一起使用。下面是一个使用 LOOP 和 LEAVE 的简单例子， 循环 100 次向 actor 表中插入记录， 当插入 100条记录后，退出循环：

		mysql> CREATE PROCEDURE actor_insert ()
			-> BEGIN
			-> set @x = 0;
			-> ins: LOOP
			-> set @x = @x + 1;
			-> IF @x = 100 then
			-> leave ins;
			-> END IF;
			-> INSERT INTO actor(first_name,last_name) VALUES ('Test','201');
			-> END LOOP ins;
			-> END;
			-> $$
		
+ 5 ITERATE:  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ITERATE 语句必须用在循环中，作用是跳过当前循环的剩下的语句，直接进入下一轮循环。下面的例子使用了 ITERATE 语句，当@x 变量是偶数的时候，不再执行循环中剩下的语句，而直接进行下一轮的循环：

		mysql> CREATE PROCEDURE actor_insert ()
			-> BEGIN
			-> set @x = 0;
			-> ins: LOOP
			-> set @x = @x + 1;
			-> IF @x = 10 then
			-> leave ins;
			-> ELSEIF mod(@x,2) = 0 then
			-> ITERATE ins;
			-> END IF;
			-> INSERT INTO actor(actor_id,first_name,last_name) VALUES (@x+200,'Test',@x);
			-> END LOOP ins;
			-> END;
			-> $$
		
+ 6 REPEAT:  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;有条件的循环控制语句，当满足条件的时候退出循环，具体语法如下：

		[begin_label:] REPEAT
			statement_list
		UNTIL search_condition
		END REPEAT [end_label]
		-->
			-> REPEAT
			-> 	FETCH cur_payment INTO i_staff_id, d_amount;
			-> 		if i_staff_id = 2 then
			-> 			set @x1 = @x1 + d_amount;
			-> 		else
			-> 			set @x2 = @x2 + d_amount;
			-> 		end if;
			-> UNTIL 0 END REPEAT;
		
+ 7 WHILE:  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;WHILE 语句实现的也是有条件的循环控制语句，即当满足条件时执行循环的内容，具体语法 如下：

		[begin_label:] WHILE search_condition DO
			statement_list
		END WHILE [end_label]
	> WHILE 循环和 REPEAT 循环的区别在于：WHILE 是满足条件才执行循环，REPEAT 是满足条件退出循环；WHILE 在首次循环执行之前就判断条件，所以循环最少执行 0 次，而 REPEAT 是在首次执行循环之后才判断条件，所以循环最少执行 1 次。以下例子用来对比 REPEAT 和 WHILE 语句的功能：
		