---
title: 数据类型
date: 2016-06-11 15:20:08 
tags: MySql
category: MySQL
---

#### 数据类型
###### 1)数值类型:
		TINYINT SMALLINT MEDIUMINT INT/INTEGER BIGINT
		使用zerofill 补0够位数,AUTO_INCREMENT自动增加
		FLOAT DOUBLE 
		DEC/DECIMAL(M,D)
		BIT(M)
		
###### 	2)时间类型:
		DATETIME 0000-00-00 00:00:00
		DATE     0000-00-00
		TIMESTAMP 00000000000000
		TIME	 00:00:00
		YEAR	0000
		1>show variables like 'time_zone' 查看时区
		2>now()函数表示当前日期
		3>set time_zone='+9:00';修改时区为东9区
###### 	3) 字符类型
		char 和varchar
			存储方式不同,char列的长度固定为创建表时生命的长度0-255 varchar是可以变的;在检索的时候char列删除了尾部的空格,而varchar则保留这些空格
		enum 类型 和 set类型
			enum一次只能选取一个成员,set一次可以选取多个成员
