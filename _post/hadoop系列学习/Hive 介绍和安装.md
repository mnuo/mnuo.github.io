---
title: Hive 介绍和安装
date: 2017-05-23 16:53:08 
tags: Hadoop
category: Hadoop
---
### 1 Hive概述
        Hive来源facebook,它使得针对Hadoop进行SQL查询变成可能。Hive是一个基于Hadoop的一个仓库工具。可以将结构化的数据文件映射为一张数据库表,并提供完整的SQL查询功能,可以将SQL语句转换成MapReduce任务运行。Hive是基于Hadoop构建的,简单的来说就是一套Hadoop的访问工具。
		
### 2 Hive安装配置
#### 2.1 Hive安装
+ 1 下载: sudo wget http://mirrors.hust.edu.cn/apache/hive/hive-1.2.2/apache-hive-1.2.2-bin.tar.gz
+ 2 解压:
		tar -zxvf apache-hive-1.2.2-bin.tar.gz
+ 3 配置HIVE_HOME: sudo vim /etc/profile 添加如下:然后source /etc/profile生效
		# set hive classpath
		export HIVE_HOME=/home/hadoop/hive-1.2.2
		export PATH=$PATH:$HIVE_HOME/bin

+ 4 启动Hive:这里启动是默认启动了derby数据库
		hadoop@master:~/hive-1.2.2$ hive
		Logging initialized using configuration in jar:file:/home/hadoop/hive-1.2.2/lib/hive-common-1.2.2.jar!/hive-log4j.properties
		hive> show databases;
		OK
		default
		Time taken: 0.911 seconds, Fetched: 1 row(s)
+ 5 修改配置:hive-1.2.2/conf/hive-site.xml
		mv hive-default.xml.template hive-default.xml
		vim hive-site.xml
		<?xml version="1.0" encoding="UTF-8" standalone="no"?>
		<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
		<configuration>
		  <property>
			<name>javax.jdo.option.ConnectionURL</name>
			<value>jdbc:mysql://localhost:3306/hive?createDatabaseIfNotExist=true</value>
			<description>JDBC connect string for a JDBC metastore</description>
		  </property>
		  <property>
			<name>javax.jdo.option.ConnectionDriverName</name>
			<value>com.mysql.jdbc.Driver</value>
			<description>Driver class name for a JDBC metastore</description>
		  </property>
		  <property>
			<name>javax.jdo.option.ConnectionUserName</name>
			<value>hive</value>
			<description>username to use against metastore database</description>
		  </property>
		  <property>
			<name>javax.jdo.option.ConnectionPassword</name>
			<value>hive</value>
			<description>password to use against metastore database</description>
		  </property>
		  <property>
				<name>hive.metastore.warehouse.dir</name>
				<value>/user/hive/warehouse</value>
			  <description>location of default database for the warehouse</description>
			</property>
		</configuration>
+ 6 hadoop创建目录:
		hadoop@master:~/hadoop-2.7.3$ bin/hadoop fs -mkdir -p /user/hive/warehouse
		hadoop@master:~/hadoop-2.7.3$ bin/hadoop fs -chmod g+w /user/hive/warehouse
#### 2.2 Mysql安装
+ 1 安装: sudo apt-get install mysql-server
+ 2 mysql一些命令:
		查看运行:  hadoop@master:~$ sudo netstat -tap | grep mysql	
		启动:service mysql start
		关闭:service mysql stop
		进入shell:  mysql -u root -padmin123
			mysql: [Warning] Using a password on the command line interface can be insecure.
			Welcome to the MySQL monitor.  Commands end with ; or \g.
			...
			mysql>
+ mysql编码设置:
		查看: show variables like "char%";
		设置: sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf 
		重启服务: service mysql restart

+ 下载mysql-connector-java,将jar复制到$HIVE_HOME/lib/目录下
		wget https://cdn.mysql.com//Downloads/Connector-J/mysql-connector-java-5.1.42.tar.gz
		cp mysql-connector-java-5.1.42-bin.jar /home/hadoop/hive-1.2.2/lib/
	
+ 登录mysql shell 创建hive数据库:
		mysql -uroot -padmin123
		mysql> create database if not exists hive default charset utf8 collate utf8_general_ci;
		mysql> create user hive@host identified by hive;
		mysql> grant all on hive.* to hive@localhost identified by 'hive';
		mysql> flush privileges;
+ 启动hive: hive


本文主要参考: 
http://blog.fens.me/hadoop-hive-intro/
http://blog.csdn.net/flyfish111222/article/details/52809850

		
	


			



		
