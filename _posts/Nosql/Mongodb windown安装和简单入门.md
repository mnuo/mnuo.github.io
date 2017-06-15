---
title: Mongodb windown安装和简单入门
date: 2017-06-15 13:53:08 
tags: Nosql
category: Nosql
---
MongoDB 是一个基于分布式文件存储的数据库。由 C++ 语言编写。旨在为 WEB 应用提供可扩展的高性能数据存储解决方案。
MongoDB 是一个介于关系数据库和非关系数据库之间的产品，是非关系数据库当中功能最丰富，最像关系数据库的。
### 1 MongoDB安装
+ 1 下载
		https://www.mongodb.com/download-center?jmp=nav#community
		
+ 2 msi 安装
+ 3 配置文件
		md $MongoDB_home/data
		md $MongoDB_home/logs
		md $MongoDB_home/etc
		新建文件$MongoDB_home/etc/mongodb.config
		dbpath=D:\appservers\MongoDB\Server\3.4\data #数据库路径
		logpath=D:\appservers\MongoDB\Server\3.4\logs\mongodb.log #日志输出文件路径
		logappend=true #错误日志采用追加模式，配置这个选项后mongodb的日志会追加到现有的日志文件，而不是从新创建一个新文件
		journal=true #启用日志文件，默认启用
		quiet=true #这个选项可以过滤掉一些无用的日志信息，若需要调试使用请设置为false
		port=27017 #端口号 默认为27017
+ 4 安装到window服务里面
		mongod.exe --config D:\appservers\MongoDB\Server\3.4\etc\mongodb.conf --install

+ 5 启动
		net start MongoDB
+ 6 测试是否安装成功
		http://localhost:27017/
		浏览器显示:
		It looks like you are trying to access MongoDB over HTTP on the native driver port.

### 2 MongoDB基本操作
+ 使用shell连接:
		jh@jh-PC MINGW32 /d/appservers/MongoDB/Server/3.4/bin
		$ mongo
		MongoDB shell version v3.4.5
		connecting to: mongodb://127.0.0.1:27017
		MongoDB server version: 3.4.5
		> show dbs
		admin  0.000GB
		local  0.000GB
		>db
		test
		>use local
		switched to db local
		>db
		local
+ 增加
		2017-06-15T09:42:08.234+0800 I CONTROL  [initandlisten]
		> db.Person.insert({"name":"张三", "age":10})
		WriteResult({ "nInserted" : 1 })
		> db.Person.insert({"name":"李四", "age":20})
		WriteResult({ "nInserted" : 1 })
		>
+ 查询
		条件查询:
			> db.Person.find({"name":"张三"})
			{ "_id" : ObjectId("5941fd2562450e6e499f419f"), "name" : "张三", "age" : 10 }
			>
		查找全部:
			> db.Person.find()
			{ "_id" : ObjectId("5941fd2562450e6e499f419f"), "name" : "张三", "age" : 10 }
			{ "_id" : ObjectId("5941fd3962450e6e499f41a0"), "name" : "李四", "age" : 20 }

+ 删除
		> db.Person.remove({"name":"张三"})
		WriteResult({ "nRemoved" : 1 })
		> db.Person.find()
		{ "_id" : ObjectId("5941fd3962450e6e499f41a0"), "name" : "李四", "age" : 20 }

### 3 MongoDB 添加用户和权限
+ 操作
		> use admin
		switched to db admin
		> db
		admin
		> db.createUser(
		... {
		...   user: "sa",
		...   pwd: "123",
		...   roles: [{role:"__system", db: "admin"}]
		... }
		... )
		Successfully added user: {
				"user" : "sa",
				"roles" : [
						{
								"role" : "__system",
								"db" : "admin"
						}
				]
		}
		>
+ 开启权限验证
		在mongodb.conf文件中添加:
			# 开启验证
			auth=true
		登录: 
		> use admin
		switched to db admin
		> db.auth('sa','123')
		1
		如果没有登录:
			> db.Person.find()
			Error: error: {
					"ok" : 0,
					"errmsg" : "not authorized on admin to execute command { find: \"Person\", fi...
					"code" : 13,
					"codeName" : "Unauthorized"
			}
+  给“test”数据库创建一个账号：testUser，密码：123，拥有“readWrite”角色的权限
		> use test
		switched to db test
		> db.createUser(
		...    {
		...      user: "testUser",
		...      pwd: "123",
		...      roles: [ { role: "readWrite", db: "test" } ]
		...    }
		...  )
		Successfully added user: {
				"user" : "testUser",
				"roles" : [
						{
								"role" : "readWrite",
								"db" : "test"
						}
				]
		}

### 4 权限说明
+ read 数据库的只读权限，包括：
		aggregate,checkShardingIndex,cloneCollectionAsCapped,collStats,count,dataSize,dbHash,dbStats,distinct,filemd5，mapReduce (inline output only.),text (beta feature.)geoNear,geoSearch,geoWalk,group
+ readWrite 数据库的读写权限，包括:
		cloneCollection (as the target database.),convertToCapped，create (and to create collections implicitly.)，renameCollection (within the same database.)findAndModify,mapReduce (output to a collection.) drop(),dropIndexes,emptycapped,ensureIndex() 
		和read的所有权限

+ dbAdmin
		clean,collMod,collStats,compact,convertToCappe create,db.createCollection(),dbStats,drop(),dropIndexes ensureIndex()，indexStats,profile,reIndex renameCollection (within a single database.),validate 
+ userAdmin角色 数据库的用户管理权限
+ clusterAdmin角色 集群管理权限(副本集、分片、主从等相关管理)，包括：
		addShard,closeAllDatabases,connPoolStats,connPoolSync,_cpuProfilerStart_cpuProfilerStop,cursorInfo,diagLogging,dropDatabase 
		shardingState,shutdown,splitChunk,splitVector,split,top,touchresync 
		serverStatus,setParameter,setShardVersion,shardCollection 
		replSetMaintenance,replSetReconfig,replSetStepDown,replSetSyncFrom 
		repairDatabase,replSetFreeze,replSetGetStatus,replSetInitiate 
		logRotate,moveChunk,movePrimary,netstat,removeShard,unsetSharding 
		hostInfo,db.currentOp(),db.killOp(),listDatabases,listShardsgetCmdLineOpts,getLog,getParameter,getShardMap,getShardVersion 
		enableSharding,flushRouterConfig,fsync,db.fsyncUnlock()
+ readAnyDatabase 任何数据库的只读权限(和read相似)
+ readWriteAnyDatabase 任何数据库的读写权限(和readWrite相似)
+ userAdminAnyDatabase 任何数据库用户的管理权限(和userAdmin相似)
+ dbAdminAnyDatabase 任何数据库的管理权限(dbAdmin相似)
+ __system 什么权限都有

参考文章:
http://www.cnblogs.com/lzrabbit/p/3682510.html
http://www.cnblogs.com/sheepswallow/p/4863578.html
http://www.cnblogs.com/sheepswallow/p/4868519.html
