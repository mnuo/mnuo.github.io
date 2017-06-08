---
title: sql分类
date: 2016-06-10 15:20:08
tags: MySql
category: MySQL
---

### sql分类
##### 1 DDL(Data Definition Languages)语句:     
    数据定义语言,这些语句定义了不同的数据段,数据库,表,列,索引等数据库对象的定义,常用的语句关键字主要包括了create,drop,alter等
		1)创建数据库:create database test1
			mysql> show databases;
			+--------------------+
			| Database           |
			+--------------------+
			| information_schema |
			| fkcloud            |
			| lifeworld          |
			| mysql              |
			| performance_schema |
			| test               |
			+--------------------+
			6 rows in set (0.04 sec)
			
			information_schema:主要存储了系统中的一些数据库对象信息。比如用户表信息,列信息,权限信息,字符集信息,分区信息等。
			cluster: 存储了系统的集群信息
			mysql: 存储了系统的用户权限
			test: 系统自动创建的测试数据库,任何用户都可以使用
		2)删除数据库
			drop database dbname;
		3) 创建表
			 create table emp(ename varchar(10),hiredate date,sal decimal(10,2),deptno int(2));
			
			desc 查看表信息
			
			show create table emp\G;查看创建表的信息
			mysql> desc emp;
			+----------+---------------+------+-----+---------+-------+
			| Field    | Type          | Null | Key | Default | Extra |
			+----------+---------------+------+-----+---------+-------+
			| ename    | varchar(10)   | YES  |     | NULL    |       |
			| hiredate | date          | YES  |     | NULL    |       |
			| sal      | decimal(10,2) | YES  |     | NULL    |       |
			| deptno   | int(2)        | YES  |     | NULL    |       |
			+----------+---------------+------+-----+---------+-------+
			4 rows in set (0.03 sec)

			mysql> show create table emp\G;
			*************************** 1. row ***************************
				   Table: emp
			Create Table: CREATE TABLE `emp` (
			  `ename` varchar(10) DEFAULT NULL,
			  `hiredate` date DEFAULT NULL,
			  `sal` decimal(10,2) DEFAULT NULL,
			  `deptno` int(2) DEFAULT NULL
			) ENGINE=InnoDB DEFAULT CHARSET=latin1
			1 row in set (0.00 sec)

			ERROR:
			No query specified


		4) 删除表
			drop table tablename;
		5) 修改表
			(1) 修改表类型
				ALTER TABLE tablename MODIFY[COLUMN] column_definition [FIRST|AFTER col_name]
				例:修改emp表的ename字段定义,将varchar(10) 改成varchar(20)
				
					mysql> alter table emp modify ename varchar(20);
					Query OK, 0 rows affected (0.15 sec)
					Records: 0  Duplicates: 0  Warnings: 0

					mysql> desc emp;
					+----------+---------------+------+-----+---------+-------+
					| Field    | Type          | Null | Key | Default | Extra |
					+----------+---------------+------+-----+---------+-------+
					| ename    | varchar(20)   | YES  |     | NULL    |       |
					| hiredate | date          | YES  |     | NULL    |       |
					| sal      | decimal(10,2) | YES  |     | NULL    |       |
					| deptno   | int(2)        | YES  |     | NULL    |       |
					+----------+---------------+------+-----+---------+-------+
					4 rows in set (0.00 sec)
			(2) 增加字段
				ALTER TABLE tablename ADD [COLUMN] column_definition [first|after col_name]
				例子:添加字段age在ename之后
				mysql> alter table emp add age int(3) after ename;
				Query OK, 0 rows affected (0.17 sec)
				Records: 0  Duplicates: 0  Warnings: 0

				mysql> desc emp;
				+----------+---------------+------+-----+---------+-------+
				| Field    | Type          | Null | Key | Default | Extra |
				+----------+---------------+------+-----+---------+-------+
				| ename    | varchar(20)   | YES  |     | NULL    |       |
				| age      | int(3)        | YES  |     | NULL    |       |
				| hiredate | date          | YES  |     | NULL    |       |
				| sal      | decimal(10,2) | YES  |     | NULL    |       |
				| deptno   | int(2)        | YES  |     | NULL    |       |
				+----------+---------------+------+-----+---------+-------+
				5 rows in set (0.00 sec)
			(3) 删除字段
				ALTER TABLE tablename DROP [COLUMN] col_name;
			(4) 字段改名
				ALTER TABLE tablename CHANGE [column] old_col_name column_definition;
			(5) 修改字段排序
			(6) 修改表名
				ALTER TABLE tablename RENAME [TO] new_tablename
				
		
##### 2 DML(Data Manipulation Language)语句:
		数据操纵语句,用于添加,删除,更新和查询数据库记录,并检查数据完整性,常用的关键字有,insert update delete和select
			(1) 插入记录(多条记录使用逗号隔开即可)
				INSERT INTO tablename(field1,field2,...) VALUES(value1,value2,...),(value11,value12,...)...;
				
			(2) 更新记录
				UPDATE t1,t2 set t1.field1=expr1,tn,fieldn=exprn [WHERE CONDITION]
				
			(3) 删除表中的记录
				DELETE t1,t2,... from t1,t2,...[WHERE CONDITION]
				
			(4) 查询记录
				1> 查询不重复记录distinct
				2>group by.. with pollup..having
				3>union|union ALL 记录联合
					
##### 3 DCL (Data Control Language)语句:
		1)创建用户
			grant select,insert on test.* to 'mn'@'localhost' identified by '123';
		2)收回权限
			remove insert on test.* from 'mn'@'localhost' identified by '123'
		数据控制语句,用于不同数据段直接的许可和访问级别的语句,这些语句定义了数据库,表,字段,用户的访问权限和安全级别,主要的语句关键字包括grant revoke等
