---
title: Pig安装及入门示例
date: 2017-05-24 17:53:08 
tags: Hadoop
category: Hadoop
---
pig是Hadoop下的一个子项目。用于简化Mapreduce的开发工作,可以用更人性化的脚本方式分析数据。

### 1 安装: 
+ 1 下载: 
		wget http://mirror.bit.edu.cn/apache/pig/pig-0.16.0/pig-0.16.0.tar.gz

+ 2 配置环境变量
		export PIG_HOME=/home/hadoop/pig-0.16.0
		export PIG_CLASSPATH=${HADOOP_HOME}/etc/hadoop/
+ 3 启动
		hadoop@master:~/pig-0.16.0$ bin/pig
		17/05/15 14:29:45 INFO pig.ExecTypeProvider: Trying ExecType : LOCAL
		17/05/15 14:29:45 INFO pig.ExecTypeProvider: Trying ExecType : MAPREDUCE
		17/05/15 14:29:45 INFO pig.ExecTypeProvider: Picked MAPREDUCE as the ExecType
		2017-05-15 14:29:45,309 [main] INFO  org.apache.pig.Main - Apache Pig version 0.16.0 (r1746530) compiled Jun 01 2016, 23:10:49
		2017-05-15 14:29:45,310 [main] INFO  org.apache.pig.Main - Logging error messages to: /home/hadoop/pig-0.16.0/pig_1494829785307.log
		2017-05-15 14:29:45,457 [main] INFO  org.apache.pig.impl.util.Utils - Default bootup file /home/hadoop/.pigbootup not found
		2017-05-15 14:29:47,641 [main] INFO  org.apache.hadoop.conf.Configuration.deprecation - mapred.job.tracker is deprecated. Instead, use mapreduce.jobtracker.address
		2017-05-15 14:29:47,642 [main] INFO  org.apache.hadoop.conf.Configuration.deprecation - fs.default.name is deprecated. Instead, use fs.defaultFS
		2017-05-15 14:29:47,642 [main] INFO  org.apache.pig.backend.hadoop.executionengine.HExecutionEngine - Connecting to hadoop file system at: hdfs://master:9000/
		2017-05-15 14:29:49,426 [main] INFO  org.apache.pig.PigServer - Pig Script ID for the session: PIG-default-8d939127-794e-4ee6-9750-a4ee1914bff4
		2017-05-15 14:29:49,429 [main] WARN  org.apache.pig.PigServer - ATS is disabled since yarn.timeline-service.enabled set to false
		grunt>


### 2 基本的HDFS操作
+ 1 ls 			
		grunt> ls /
		hdfs://master:9000/tmp  <dir>
		hdfs://master:9000/user <dir>

+ 2 cat
		grunt> cat /user/hdfs/log_kpi/pv/part-00000
		/about  5
		/black-ip-list/ 2
		/cassandra-clustor/     3
		/finance-rhive-repurchase/      13
		/hadoop-family-roadmap/ 13
		/hadoop-hive-intro/     14
		/hadoop-mahout-roadmap/ 20
		/hadoop-zookeeper-intro/        6

### 3 基本数据分析
+ 求count
		grunt> pwd
		hdfs://master:9000/user/hadoop
		grunt> mkdir access
		grunt> cd access
		grunt> copyFromLocal /home/hadoop/datafile/access_log.10 access.log
		grunt> ls
		hdfs://master:9000/user/hadoop/access/access.log<r 1>   3025757
		grunt> a = load '/user/hadoop/access/access.log' using PigStorage(' ') as (ip,a1,a3,a4,a5,a6,a7,a8);
		grunt> b = foreach a generate ip;
		grunt> c = group b by ip;
		grunt> d = foreach c generate group,COUNT($1);
		grunt> dump d;
		...
		(5.39.81.6,1)
		(60.10.8.5,56)
		(65.88.2.2,22)
		....

+ 求MAX: 
		grunt> c = FOREACH b GENERATE MAX(a.value);
+ 求平均值(AVG)
		grunt> c = FOREACH b GENERATE AVG(a.value);
+ 求和(SUM)
		grunt> c = FOREACH b GENERATE SUM(a.value);
	

文章参考: 
http://www.cnblogs.com/yjmyzz/p/hadoop-pig-tutorial.html
http://blog.csdn.net/lichangzai/article/details/8542718