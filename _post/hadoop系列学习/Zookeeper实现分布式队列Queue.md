---
title: Zookeeper实现分布式队列Queue
date: 2017-05-25 23:53:08 
tags: Hadoop
category: Hadoop
---
队列有很多种,大都是消息系统所实现的。想ActiveMQ, JBossMQ, RabbitMQ, IBM-MQ等。分布式队列并不多,像Beanstalk.
本文实现的分布式队列,是基于Zookeeper实现的一种同步的分布式队列。当一个队列成员都聚集时,这个队列才可用,否则一直等待所有成员到达。

### 1 设计思路
+ 
		创建一个父目录/queue,每个成员都监控(watch)标志位目录(/queue/start)是否存在,然后每个成员加入这个队列,加入队列的方式就是创建/queue/x(i)的临时目录节点,然后
		每个成员获取/queue目录的所有目录节点,也就是x(i)。判断i的值是否已经是成员的个数,如果小于成员个数等待/queue/start的出现,如果已经相等就创建/queue/start
+ 产品流程图
	![产品流程](http://7xsqwa.com1.z0.glb.clouddn.com/zookeeper-queue.png)

+ 应用实例图
	![产品流程](http://7xsqwa.com1.z0.glb.clouddn.com/zookeeper-queue-dataprocess.png)
+ 图标解释
	- 1 app1,app2,app3,app4是4个独立的业务系统
	- 2 zk1,zk2, zk3是zookeeper集群的3个连接点
	- 3 /queue,是znode的队列,假设队列长度为3
	- 4 /queue/x1,是znode队列中,1号排队者,由app1提交,同步请求,app1挂起等待
	- 5 /queue/x2,是znod队列中,2号排队者,由app2提交,同步请求,app2挂起等待
	- 6 /queue/x3,是znod队列中,3号排队者,由app2提交,同步请求,app3挂起等待
	- 7 /queue/start,当znode队列满了,触发创建开始节点
	- 8 当/queue/start被创建后,app4启动,所有zk的连接通知同步程序,队列已完成,所有程序结束
+ 注意点: 
	- 1. 创建/queue/x1,/queue/x2,/queue/x3没有前后顺序，提交后程序就同步挂起。
	- 2. app1可以通过zk2提交，app2也可通过zk3提交
	- 3. app1可以提交3次请求，生成x1,x2,x3使用队列充满
	- 4. /queue/start被创建后，zk1会监听到这个事件，再告诉app1，队列已完成！

### 2 JAVA 实例
+ 单节点模拟
		public static void doOne() throws Exception{
			String host1 = "192.168.159.132:2181";
			ZooKeeper zk = connection(host1);
			initQueue(zk);
			joinQueue(zk, 1);
			joinQueue(zk, 2);
			joinQueue(zk, 3);
			zk.close();
		}
		
		public static ZooKeeper connection(String host) throws Exception{
			ZooKeeper zk = new ZooKeeper(host, 60000, new Watcher() {
				public void process(WatchedEvent event) {
					if(event.getPath() != null && event.getPath().equals("/queue/start") && event.getType() == Event.EventType.NodeCreated){
						System.out.println("Queue has Complete, Finish testing!!!");
					}
				}
			});
			return zk;
		}
		
		public static void initQueue(ZooKeeper zk) throws Exception{
			System.out.println("WATCH => /queue/start");
			zk.exists("/queue/start", true);
			if(zk.exists("/queue", false) == null){
				System.out.println("create /queue task-queue");
				zk.create("/queue", "task-queue".getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
			}else{
				System.out.println("/queue is exist!");
			}
		}
		
		public static void joinQueue(ZooKeeper zk, int x) throws Exception{
			System.out.println("create /queue/x" + x + " x" + x);
			zk.create("/queue/x" + x, ("x" + x).getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL_SEQUENTIAL);
			isCompleted(zk);
		}
		
		public static void isCompleted(ZooKeeper zk) throws Exception{
			int size = 3;
			int length = zk.getChildren("/queue", true).size();
			System.out.println("Queue Complete: " + length + "/" + size);
			if(length >= 3){
				System.out.println("create /queue/start start");
				zk.create("/queue/start", "start".getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL);
			}
		}
		/**
		 * @param args
		 * @throws Exception 
		 */
		public static void main(String[] args) throws Exception {
			doOne();
		}

+ 分布式模拟
		public class QueueZookeeper {
			public static void doAction(int client) throws Exception{
				String host1 = "192.168.159.132:2181";
				String host2 = "192.168.159.134:2181";
				String host3 = "192.168.159.135:2181";
				ZooKeeper zk = null;
				switch (client) {
				case 1:
					zk = connection(host1);
					initQueue(zk);
					joinQueue(zk, 1);
					break;
				case 2:
					zk = connection(host2);
					initQueue(zk);
					joinQueue(zk, 2);
					break;
				case 3:
					zk = connection(host3);
					initQueue(zk);
					joinQueue(zk, 3);
					break;
				}
			}
			public static ZooKeeper connection(String host) throws Exception{
				ZooKeeper zk = new ZooKeeper(host, 60000, new Watcher() {
					public void process(WatchedEvent event) {
						if(event.getPath() != null && event.getPath().equals("/queue/start") && event.getType() == Event.EventType.NodeCreated){
							System.out.println("Queue has Complete, Finish testing!!!");
						}
					}
				});
				return zk;
			}
			
			public static void initQueue(ZooKeeper zk) throws Exception{
				System.out.println("WATCH => /queue/start");
				zk.exists("/queue/start", true);
				if(zk.exists("/queue", false) == null){
					System.out.println("create /queue task-queue");
					zk.create("/queue", "task-queue".getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
				}else{
					System.out.println("/queue is exist!");
				}
			}
			
			public static void joinQueue(ZooKeeper zk, int x) throws Exception{
				System.out.println("create /queue/x" + x + " x" + x);
				zk.create("/queue/x" + x, ("x" + x).getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL_SEQUENTIAL);
				isCompleted(zk);
			}
			
			public static void isCompleted(ZooKeeper zk) throws Exception{
				int size = 3;
				int length = zk.getChildren("/queue", true).size();
				System.out.println("Queue Complete: " + length + "/" + size);
				if(length >= 3){
					System.out.println("create /queue/start start");
					zk.create("/queue/start", "start".getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL);
				}
			}
			/**
			 * @param args
			 * @throws Exception 
			 */
			public static void main(String[] args) throws Exception {
				if(args.length == 0){
					doOne();
				}else{
					doAction(Integer.parseInt(args[0]));
				}
			}
		}
参考文章:
http://blog.fens.me/zookeeper-queue/



























