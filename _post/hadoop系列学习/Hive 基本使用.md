---
title: Hive 基本使用
date: 2017-05-23 19:53:08 
tags: Hadoop
category: Hadoop
---
### 1 使用操作: 
+ 准备文件: 
		hadoop@master:~/datafile$ vim t_hive.txt
		16	2	3
		61	12	13
		41	2	31
		17	21	3
		71	2	31
		1	12	34
		11	2	34
+ 创建表: 
		hive> CREATE TABLE t_hive(a int, b int) ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t';
		OK
		Time taken: 1.79 seconds
+ 导入数据: 		
		hive> LOAD DATA LOCAL INPATH '/home/hadoop/datafile/t_hive.txt' OVERWRITE INTO TABLE t_hive;
		Loading data to table default.t_hive
		Table default.t_hive stats: [numFiles=1, numRows=0, totalSize=56, rawDataSize=0]
		OK
		Time taken: 14.721 seconds
+ 查看表: 
		hive> show tables;
		OK
		t_hive
		Time taken: 0.318 seconds, Fetched: 1 row(s)
		hive> show tables '*t*';
		OK
		t_hive
		Time taken: 0.056 seconds, Fetched: 1 row(s)
+ 查看表数据: 
		hive> select * from t_hive;
		OK
		16      2
		61      12
		41      2
		17      21
		71      2
		1       12
		11      2
		Time taken: 3.169 seconds, Fetched: 7 row(s)
+ 查看表结构: 
		hive> desc t_hive;
		OK
		a                       int
		b                       int
		Time taken: 0.294 seconds, Fetched: 2 row(s)
+ 添加字段: 
		hive> ALTER TABLE t_hive ADD COLUMNS (new_col String);
		OK
		Time taken: 0.424 seconds
		hive> desc t_hive;
		OK
		a                       int
		b                       int
		new_col                 string
		Time taken: 0.222 seconds, Fetched: 3 row(s)
+ 重命名表: 
		hive> ALTER TABLE t_hive RENAME TO t_hadoop;
		OK
		Time taken: 0.328 seconds
		hive> show tables;
		OK
		t_hadoop
		Time taken: 0.072 seconds, Fetched: 1 row(s)
+ 删除表: 
		hive> drop table t_hadoop;
		OK
		Time taken: 1.308 seconds
		hive> show tables;
		OK
		Time taken: 0.068 seconds
### 2 Hive交互模式:
+ 
		quit,exit:  退出交互式shell
		reset: 重置配置为默认值
		set <key>=<value> : 修改特定变量的值(如果变量名拼写错误，不会报错)
		set :  输出用户覆盖的hive配置变量
		set -v : 输出所有Hadoop和Hive的配置变量
		add FILE[S] *, add JAR[S] *, add ARCHIVE[S] * : 添加 一个或多个 file, jar, archives到分布式缓存
		list FILE[S], list JAR[S], list ARCHIVE[S] : 输出已经添加到分布式缓存的资源。
		list FILE[S] *, list JAR[S] *,list ARCHIVE[S] * : 检查给定的资源是否添加到分布式缓存
		delete FILE[S] *,delete JAR[S] *,delete ARCHIVE[S] * : 从分布式缓存删除指定的资源
		! <command> :  从Hive shell执行一个shell命令
		dfs <dfs command> :  从Hive shell执行一个dfs命令
		<query string> : 执行一个Hive 查询，然后输出结果到标准输出
		source FILE <filepath>:  在CLI里执行一个hive脚本文件

### 3 导入数据
+ 1 导入数据: 
		hive> LOAD DATA LOCAL INPATH '/home/hadoop/datafile/t_hive.txt' OVERWRITE INTO TABLE t_hive;
		Loading data to table default.t_hive
		Table default.t_hive stats: [numFiles=1, numRows=0, totalSize=56, rawDataSize=0]
		OK
		Time taken: 2.975 seconds
		HDFS中查看: 		
		hadoop@master:~/hadoop-2.7.3$ bin/hadoop fs -cat /user/hive/warehouse/t_hive/t_hive.txt
		16      2       3
		61      12      13
		41      2       31
		17      21      3
		71      2       31
		1       12      34
		11      2       34

+ 2 从HDFS导入数据: 
		hive> CREATE TABLE t_hive2 (a int, b int, c int) ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t';
		OK
		Time taken: 0.174 seconds
		hive> LOAD DATA INPATH '/user/hive/warehouse/t_hive/t_hive.txt' OVERWRITE INTO TABLE t_hive2;
		Loading data to table default.t_hive2
		Table default.t_hive2 stats: [numFiles=1, numRows=0, totalSize=56, rawDataSize=0]
		OK
		Time taken: 0.954 seconds
		hive> select * from t_hive2;
		OK
		16      2       3
		61      12      13
		41      2       31
		17      21      3
		71      2       31
		1       12      34
		11      2       34
		Time taken: 1.309 seconds, Fetched: 7 row(s)
+ 3 从其他表导入数据:
		hive> insert overwrite table t_hive2 select * from t_hive;

+ 4 创建表并从其他表导入数据:
		hive> CREATE TABLE t_hive3 AS SELECT * FROM t_hive2 ;

+ 5 仅复制表结构不导数据
		hive> CREATE TABLE t_hive4 LIKE t_hive;
		hive> select * from t_hive4;
		OK
		Time taken: 0.077 seconds

### 4 数据导出
+ 1 从HDFS到HDFS 
	bin/hadoop fs -cp /user/hive/warehouse/t_hive /

+ 2 通过Hive导出到本地文件系统
	hive> INSERT OVERWRITE LOCAL DIRECTORY '/tmp/t_hive' SELECT * FROM t_hive;
	
### 5 Hive查询HiveQL
+ 普通查询：排序，列别名，嵌套子查询
		hive> FROM (
			>   SELECT b,c as c2 FROM t_hive
			> ) t
			> SELECT t.b, t.c2
			> WHERE b>2
			> LIMIT 2;
+ 连接查询：JOIN
		hive> SELECT t1.a,t1.b,t2.a,t2.b
			> FROM t_hive t1 JOIN t_hive2 t2 on t1.a=t2.a
			> WHERE t1.c>10;
+ 聚合查询1：count, avg
		hive> SELECT count(*), avg(a) FROM t_hive;
+ 聚合查询2：count, distinct
		hive> SELECT count(DISTINCT b) FROM t_hive;
+ 聚合查询3：GROUP BY, HAVING
		hive> SELECT avg(a),b,sum(c) FROM t_hive GROUP BY b,c
		hive> SELECT avg(a),b,sum(c) FROM t_hive GROUP BY b,c HAVING sum(c)>30
### 6 Hive视图
+ 
		创建: hive> CREATE VIEW v_hive AS SELECT a,b FROM t_hive where c>30;
		删除: hive> DROP VIEW IF EXISTS v_hive
### 7 Hive分区表
+ 常见数据:
		vi /home/cos/demo/t_hft_20130627.csv
		000001,092023,9.76
		000002,091947,8.99
		000004,092002,9.79
		000005,091514,2.2
		000001,092008,9.70
		000001,092059,9.45
		~ vi /home/cos/demo/t_hft_20130628.csv
		000001,092023,9.76
		000002,091947,8.99
		000004,092002,9.79
		000005,091514,2.2
		000001,092008,9.70
		000001,092059,9.45
		
+ 创建数据表: 
		DROP TABLE IF EXISTS t_hft;
		CREATE TABLE t_hft(
		SecurityID STRING,
		tradeTime STRING,
		PreClosePx DOUBLE
		) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',';

+ 导入数据: 
		hive> LOAD DATA LOCAL INPATH '/home/cos/demo/t_hft_20130627.csv' OVERWRITE INTO TABLE t_hft PARTITION (tradeDate=20130627);
		Copying data from file:/home/cos/demo/t_hft_20130627.csv
		Copying file: file:/home/cos/demo/t_hft_20130627.csv
		Loading data to table default.t_hft partition (tradedate=20130627)
		hive> LOAD DATA LOCAL INPATH '/home/cos/demo/t_hft_20130628.csv' OVERWRITE INTO TABLE t_hft PARTITION (tradeDate=20130628);
		Copying data from file:/home/cos/demo/t_hft_20130628.csv
		Copying file: file:/home/cos/demo/t_hft_20130628.csv
		Loading data to table default.t_hft partition (tradedate=20130628)

+ 查看分区表
		hive> SHOW PARTITIONS t_hft;
		tradedate=20130627
		tradedate=20130628
		Time taken: 0.082 seconds
+ 查询数据
		hive> select * from t_hft where securityid='000001';
		000001  092023  9.76    20130627
		000001  092008  9.7     20130627
		000001  092059  9.45    20130627
		000001  092023  9.76    20130628
		000001  092008  9.7     20130628
		000001  092059  9.45    20130628
		hive> select * from t_hft where tradedate=20130627 and PreClosePx<9;
		000002  091947  8.99    20130627
		000005  091514  2.2     20130627
		
文章参考: 
http://blog.fens.me/hadoop-hive-intro/