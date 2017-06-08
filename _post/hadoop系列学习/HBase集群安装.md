---
title: HBase集群安装
date: 2017-05-26 23:53:08 
tags: Hadoop
category: Hadoop
---
HBase是Hadoop家族中的一个分布式数据库产品,HBase支持高并发读写,列式数据存储,高效的索引,自动分片,自动region迁移等优点,被业界认可并实施。

### 1 HBase安装
+ 下载
		wget https://mirrors.tuna.tsinghua.edu.cn/apache/hbase/1.3.1/hbase-1.3.1-bin.tar.gz
+ 解压
		tar -zxvf hbase-1.3.1-bin.tar.gz
+ 配置hbase-env.sh
		1 配置java_home
		export JAVA_HOME=/usr/local/jdk1.8.0_111
		2 关闭Hbase自带的zookeeper,使用zookeeper集群
		export  HBASE_MANAGES_ZK=false  
+ 编辑hbase-site.xml
		<?xml version="1.0"?>
		<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
		<configuration>
		　　<property> 
		　　　　<name>hbase.rootdir</name> 
		　　　　<value>hdfs://master:8020/hbase</value> 
		　　</property> 
		　　<property> 
		　　　　<name>hbase.cluster.distributed</name> 
		　　　　<value>true</value> 
		　　</property> 
		　　<property> 
		　　　　<name>hbase.zookeeper.quorum</name> 
		　　　　<value>master,slave1,slave2</value> 
		　　</property> 
		　　<property> 
		　　　　<name>hbase.zookeeper.property.dataDir</name> 
		　　　　<value>/home/hadoop/hbase-1.3.1/tmp/zk/data</value> 
		　　</property>
		</configuration>

+ 编辑 regionservers 
		vim regionservers
		加入:
		slave1
		slave2
+ 把Hbase复制到其他机器
		scp -r hbase-1.3.1 hadoop@slave1:~/
		scp -r hbase-1.3.2 hadoop@slave2:~/

+ 在master开启Hbase:
		bin/start-hbase.sh   
+ 进入shell
		hadoop@master:~/hbase-1.3.1$ bin/hbase shell
		hbase(main):001:0> version
		1.3.1, r930b9a55528fe45d8edce7af42fef2d35e77677a, Thu Apr  6 19:36:54 PDT 2017

### 2 Java API
+ 
			public class HBaseAPI {
				//公共连接代码
				private static Configuration conf = null;
				static {
					Configuration HBASE_CONFIG = new Configuration();
					// 与hbase/conf/hbase-site.xml 中 hbase.zookeeper.quorum 配置的值相同
					HBASE_CONFIG.set("hbase.zookeeper.quorum", "master,slave1,slave2");
					// 与hbase/conf/hbase-site.xml 中 hbase.zookeeper.property.clientPort 配置的值相同
					HBASE_CONFIG.set("hbase.zookeeper.property.clientPort", "9000");
					conf = HBaseConfiguration.create(HBASE_CONFIG);
				}
				public static void createTable(String tableName, String[] familys, boolean force) throws Exception{
					//建立连接
					Connection connection = ConnectionFactory.createConnection(conf);
					//表管理类
					Admin admin = connection.getAdmin();
					if(admin.tableExists(TableName.valueOf(tableName))){
						if(force){
							//禁用表
							admin.disableTable(TableName.valueOf(tableName));
							//删除表
							admin.deleteTable(TableName.valueOf(tableName));
							System.out.println("begin create table");
						}else{
							System.out.println("table is exists!");
							admin.close();
							connection.close();
							return;
						}
					}
					//定义表名
					HTableDescriptor tblDesc = HTableDescriptor.metaTableDescriptor(conf);
					for(int i = 0; i < familys.length; i ++){
						//定义列族
						HColumnDescriptor columnDesc = new HColumnDescriptor(familys[i]);
						//将列添加到table中
						tblDesc.addFamily(columnDesc);
					}
					//建表
					admin.createTable(tblDesc);
					System.out.println("create table: " + tableName + " success!");
					//关闭管理
					admin.close();
					connection.close();
				}
				/**
				 * @param args
				 * @throws Exception 
				 */
				public static void main(String[] args) throws Exception {
					createTable("userHbase", new String[]{"name", "age"}, true);
				}
			}

参考文章:
http://www.cnblogs.com/netbloomy/p/6677883.html
http://www.cnblogs.com/netbloomy/p/6683509.html
