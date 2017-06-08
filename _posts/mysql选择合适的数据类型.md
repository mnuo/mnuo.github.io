---
title: mysql选择合适的数据类型
date: 2016-06-18 15:20:08 
tags: MySql
category: MySQL
---
#### 1) CHAR 与VARCHAR 
CHAR 和 VARCHAR 都是用来存储字符串,但他们保存和检索的方式不同.char 属于固定长度的字符类型,而varchar数据可变长度的字符类型
		
		1> MyISAM存储引擎:  建议使用固定长度的数据列代替可变长度的数据列
		2> MEMORY存储引擎:  目前都是使用固定长度的数据行存储,因此无论使用CHAR或VARCHAR列都没有关系,两者都是作为char类型处理
		3> InnoDB存储引擎:  建议使用VARCHAR类型.
#### 2) TEXT与BLOB
两者的主要差别是BLOB能用来保存二进制数据,比如照片;而TEXT只能保存字符数据,比如一篇文章或者日记
		
        1>BLOB和TEXT值会引起一些性能问题,特别是执行了大量的删除操作时.为了提高性能,建议定期使用OPTIMIZE TABLE功能对这类表进行脆片整理避免因为空洞导致性能问题
		2>可以使用合成的(Synthetic)索引来提高大文本字段(BLOB或TEXT)的查询性能
			create table t(id varchar(100), context blob,hash_value varchar(40));
			insert into t values(1,repeat('beijing',2),md5(context))...
			select * from t where hash_value=md5(repeat('beijing',2));
			以上方法只能精确匹配,如果需要对BLOB后者CLOB字段进行模糊查询,MYSQL提供了前缀索引,也就是只为字段的前n列创建索引
			create index idx_blob on t(context(100));
		3> 在不必要的时候避免检索大型的BLOB或者TEXT
		4>把BLOB或TEXT列分离到单独的表中
	
#### 3) 浮点数和定点数
		浮点数一般用于表示含有小数部分的数值,当一个字段被定义为浮点类型后,如果插入数据的精度超过该列定义的实际精度,则插入值会被四舍五入到实际定义的精度值,四舍五入的过程不会报错.在MYSQL中float double用来表示浮点数
		定点数实际上以字符串形式存放的,所以定点数可以更加精确的保存数据,如果实际插入的数值精度大于实际定义的精度,则MYSQL会进行警告,但数据按照实际精度四舍五入后插入,如果是传统模式,系统直接报错,无法插入.

		注:
			1.浮点数存在误差问题
			2.对货币等对精度敏感的数据,应该用定点数表示或存储
			3.在编程中,如果用到浮点数,要特别注意误差问题,并尽量避免做浮点数比较
			4.要注意浮点数中一些特殊值的处理
            
#### 4)日期类型选择
		1.根据实际需要选择能够满足应用的最小存储日期类型
		2.如果记录年月日时分秒,并且记录的年份比较久远,那么最后使用datetime而不要使用timestamp
		3.如果记录的日期需要让不同时区的用户使用g,zuihou使用timestamp,因为日期类中只有他能够和实际市区相对应