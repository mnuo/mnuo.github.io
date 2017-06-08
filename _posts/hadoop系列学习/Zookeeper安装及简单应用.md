---
title: Zookeeper安装及简单应用
date: 2017-05-25 14:53:08 
tags: Hadoop
category: Hadoop
---
Zookeeper是一个分布式应用所设计的分布的,开源的协调服务,它主要用来解决 分布式应用中经常遇到的一些数据管理问题,简化分布式应用协调及其管理的难度,提高性能的分布式服务。
它向外部应用暴露一组通用服务: 分布式同步(Distributed Synchronization),命令服务(Naming Server),集群维护(Group Maintenance)等。
Zookeeper本身可以单机运行,不过它的长处在于通过分布式Zookeeper集群(Leader,多个Follower),基于一定的策略来保证Zookeeper集群的稳定性和可用性,从而实现分布式应用的可靠性。
### 1 Zookeeper 集群安装
+ 下载: 
		hadoop@master:~$ wget http://mirror.bit.edu.cn/apache/zookeeper/zookeeper-3.4.9/zookeeper-3.4.9.tar.gz

+ 解压修改文件名称:
		hadoop@master:~$ tar -zxvf zookeeper-3.4.9.tar.gz
		hadoop@master:~$ mv zookeeper-3.4.9 zookeeper-3.4.9_1
+ 配置环境变量:		
		# set zookeeper environment
		export ZOOKEEPER_HOME=/home/hadoop/zookeeper-3.4.9_1
		export PATH=$PATH:$ZOOKEEPER_HOME/bin:$ZOOKEEPER_HOME/conf
+ 修改配置文件 zoo.cfg, 
		hadoop@master:~/zookeeper-3.4.9_1/conf$ mv zoo_sample.cfg zoo.cfg
		hadoop@master:~/zookeeper-3.4.9_1/conf$ vim zoo.cfg
			# the directory where the snapshot is stored.
			# do not use /tmp for storage, /tmp here is just
			# example sakes.
			dataDir=/home/hadoop/zookeeper-3.4.9_1/data
			# the directory where the log is stored.
			dataLogDir=/home/hadoop/zookeeper-3.4.9_1/logs
			# the port at which the clients will connect
			clientPort=2181
			# the servers setting
			server.1=192.168.159.132:2881:3881
			server.2=192.168.159.134:2881:3881
			server.3=192.168.158.135:2881:3881
		在dataDir所指定的目录下创建一个文件名为myid的文件，文件中的内容只有一行，为本主机对应的id值，也就是server.id中的id
		hadoop@master:~/zookeeper-3.4.9_1$ mkdir logs
		hadoop@master:~/zookeeper-3.4.9_1$ mkdir data
		hadoop@master:~/zookeeper-3.4.9_1$ cd data/
		hadoop@master:~/zookeeper-3.4.9_1/data$ vim myid
+ 同以上操作分别配置其它的节点

+ 启动,分别在节点上启动zookeeper
		hadoop@master:~/zookeeper-3.4.9_1$ bin/zkServer.sh start
		ZooKeeper JMX enabled by default
		Using config: /home/hadoop/zookeeper-3.4.9_1/bin/../conf/zoo.cfg
		Starting zookeeper ... STARTED
		hadoop@master:~/zookeeper-3.4.9_1$ jps
		61920 ResourceManager
		61570 NameNode
		14827 QuorumPeerMain
		14845 Jps
		61775 SecondaryNameNode
		其中QuorumPeerMain就是zookeeper的进程
		
		hadoop@slave1:~/zookeeper-3.4.9_2$ bin/zkServer.sh start
		ZooKeeper JMX enabled by default
		Using config: /home/hadoop/zookeeper-3.4.9_2/bin/../conf/zoo.cfg
		Starting zookeeper ... STARTED
		hadoop@slave1:~/zookeeper-3.4.9_2$ jps
		8272 NodeManager
		8180 DataNode
		63735 Jps
		63708 QuorumPeerMain
		
		hadoop@slave2:~/zookeeper-3.4.9_3$ bin/zkServer.sh start
		ZooKeeper JMX enabled by default
		Using config: /home/hadoop/zookeeper-3.4.9_3/bin/../conf/zoo.cfg
		Starting zookeeper ... STARTED
		hadoop@slave2:~/zookeeper-3.4.9_3$ jps
		61636 QuorumPeerMain
		53931 DataNode
		54028 NodeManager
		61661 Jps

+ 查看zookeeper状态:
		hadoop@master:~/zookeeper-3.4.9_1$ bin/zkServer.sh status
		ZooKeeper JMX enabled by default
		Using config: /home/hadoop/zookeeper-3.4.9_1/bin/../conf/zoo.cfg
		Mode: follower
		
		hadoop@slave1:~/zookeeper-3.4.9_2$ bin/zkServer.sh status
		ZooKeeper JMX enabled by default
		Using config: /home/hadoop/zookeeper-3.4.9_2/bin/../conf/zoo.cfg
		Mode: leader

		hadoop@slave2:~/zookeeper-3.4.9_3$ bin/zkServer.sh status
		ZooKeeper JMX enabled by default
		Using config: /home/hadoop/zookeeper-3.4.9_3/bin/../conf/zoo.cfg
		Mode: follower
		
		通过上面状态查询结果可见，slave1是集群的Leader，其余的两个结点是Follower。

+ 关闭节点:
		hadoop@master:~/zookeeper-3.4.9_1$ bin/zkServer.sh stop

### 2 Zookeeper命令行操作
+ 连接命令: 
		hadoop@master:~/zookeeper-3.4.9_1$ bin/zkCli.sh -server 192.168.159.134:2181
+ 查看目录/:
		[zk: 192.168.159.134:2181(CONNECTED) 2] ls /
		[zookeeper]
+ 创建一个node节点:
		[zk: 192.168.159.134:2181(CONNECTED) 3] create /node conan
		Created /node
		[zk: 192.168.159.134:2181(CONNECTED) 4] ls /
		[node, zookeeper]
+ 查看/node信息:
		[zk: 192.168.159.134:2181(CONNECTED) 5] get /node
		conan
		cZxid = 0x100000004
		ctime = Mon May 15 20:46:56 CST 2017
		mZxid = 0x100000004
		mtime = Mon May 15 20:46:56 CST 2017
		pZxid = 0x100000004
		cversion = 0
		dataVersion = 0
		aclVersion = 0
		ephemeralOwner = 0x0
		dataLength = 5
		numChildren = 0
+ 修改/node信息:
		[zk: 192.168.159.134:2181(CONNECTED) 6] set /node fens.me
		cZxid = 0x100000004
		ctime = Mon May 15 20:46:56 CST 2017
		mZxid = 0x100000005
		mtime = Mon May 15 20:47:42 CST 2017
		pZxid = 0x100000004
		cversion = 0
		dataVersion = 1
		aclVersion = 0
		ephemeralOwner = 0x0
		dataLength = 7
		numChildren = 0
		[zk: 192.168.159.134:2181(CONNECTED) 7] get /node
		fens.me
		cZxid = 0x100000004
		ctime = Mon May 15 20:46:56 CST 2017
		mZxid = 0x100000005
		mtime = Mon May 15 20:47:42 CST 2017
		pZxid = 0x100000004
		cversion = 0
		dataVersion = 1
		aclVersion = 0
		ephemeralOwner = 0x0
		dataLength = 7
		numChildren = 0
+ 删除/node:
		[zk: 192.168.159.134:2181(CONNECTED) 8] delete /node
		[zk: 192.168.159.134:2181(CONNECTED) 9] ls /
		[zookeeper]

+ 退出连接:
		[zk: 192.168.159.134:2181(CONNECTED) 10] quit
		Quitting...
		2017-05-15 20:59:15,375 [myid:] - INFO  [main:ZooKeeper@684] - Session: 0x25c0c16ef3a0000 closed
		2017-05-15 20:59:15,385 [myid:] - INFO  [main-EventThread:ClientCnxn$EventThread@519] - EventThread shut down for session: 0x25c0c16ef3a0000

### 3 JAVA编程实现:
+ pom.xml
		<!-- https://mvnrepository.com/artifact/org.apache.zookeeper/zookeeper -->
		<dependency>
		    <groupId>org.apache.zookeeper</groupId>
		    <artifactId>zookeeper</artifactId>
		    <version>3.4.9</version>
		</dependency>
		
+ 代码示例:
		public static void main(String[] args) throws Exception {
			//创建一个与服务器的连接
			ZooKeeper zk = new ZooKeeper("192.168.159.132:2181", 60000, new Watcher(){
				//监控所有触发事件
				public void process(WatchedEvent event) {
					System.out.println("EVENT: " + event.getType());
				}
			});
			
			//查看根节点:
			System.out.println("ls / => " + zk.getChildren("/", true));
			
			//创建一个目录节点
			if(zk.exists("/node", true) == null){
				zk.create("/node", "conan".getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
				System.out.println("create /node conan");
				//查看node节点数据
				System.out.println("ls /node => " + new String(zk.getData("/node", false, null)));
				//查看根节点
				System.out.println("ls / => " + zk.getChildren("/", true));
			}
			//创建一个子节点目录
			if(zk.exists("/node/sub1", true) == null){
				zk.create("/node/sub1", "sub1".getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
				System.out.println("create /node/sub1 sub1");
				//查看/node节点
				System.out.println("ls /node => " + zk.getChildren("/node", true));
			}
			//修改节点
			if(zk.exists("/node", true) != null){
				zk.setData("/node", "change".getBytes(), -1);
				//查看node节点数据
				System.out.println("ls /node => " + new String(zk.getData("/node", false, null)));
			}
			//删除节点
			if(zk.exists("/node/sub1", true) != null){
				zk.delete("/node/sub1", -1);
				zk.delete("/node", -1);
				//查看根节点
				System.out.println("ls / => " + zk.getChildren("/", true));
			}
			//关闭连接
			zk.close();
		}

文章参考:
http://blog.csdn.net/haihongazar/article/details/52606801

		

		 

	











