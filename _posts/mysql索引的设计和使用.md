---
title: mysql索引的设计和使用
date: 2016-06-19 15:20:08 
tags: MySql
category: MySQL
---
##### 索引的创建 修改 删除
    create [unique|filltext|spatial] index idex_name
			[using index_type]
			on tbl_name(index_col_name,...)
		alter table 
		..
		drop index index_name on table_name
        
        
##### BTREE和HASH索引:

		关于HASH索引:
			1> 只用于使用=或<=>操作符的等是比较
			2>优化器不能使用HASH索引来加速ORDER BY 操作符的等是比较
			3> MySQL 不能确定在两个值之间大约有多少行.如果将一个MyISAM表改成HASH索引的MEMEORY表,会影响一些查询的执行效率
			4> 只能使用整个关键字来搜索一行
		BTREE索引,当使用>,< >= <= between,!= <> like 'pattern%'都可以使用相关列上的索引