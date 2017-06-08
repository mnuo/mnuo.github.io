---
title: mysql优化数据类型
date: 2016-07-12 16:12:08 
tags: MySql
category: MySQL
---
####  1 优化数据类型
	select * from tbl_name procedure analyse();
	select * from tbl_name procedure analyse(16,254);
######  建表：

	mysql> CREATE TABLE duck_cust(

		-> cust_num MEDIUMINT AUTO_INCREMENT,

		-> cust_title TINYINT,

		-> cust_last CHAR(20) NOT NULL,

		-> cust_first CHAR(15) NOT NULL,

		-> cust_suffix ENUM('jr.','II','III','IV','V','M.D.','PhD'),

		-> cust_add1 CHAR(30) NOT NULL,

		-> cust_add2 CHAR(10),

		-> cust_city CHAR(18) NOT NULL,

		-> cust_state CHAR(2) NOT NULL,

		-> cust_zip1 CHAR(5) NOT NULL,

		-> cust_zip2 CHAR(4), 

		-> cust_duckname CHAR(25) NOT NULL,

		-> cust_duckday DATE,

		-> PRIMARY KEY (cust_num)

		-> )engine=MyISAM;


#### 2表的拆分