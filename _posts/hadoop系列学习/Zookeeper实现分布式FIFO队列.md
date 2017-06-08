---
title: Zookeeper实现分布式FIFO队列
date: 2017-05-25 14:53:08 
tags: Hadoop
category: Hadoop
---
在计算机科学中,消息队列(Message Queue)是一种进程间通信或者是同一进程中不同线程间的通信方式。消息队列提供了异步的通信协议,消息的发送者和接受者不需要同时与消息队列交互,消息会保存在队列中知道接收者取回它。先进先出(FIFO first in first out)队列是消息队列的一种实现形式,先发出的先消费。

### 1 设计思路
+ 实现思路:
		在/queue-fifo目录下创建SEQUENTIAL类型的子目录/x(i),这样就能保证所有成员加入队列时都是有编号的,出队列时通过getChildren()方法可以返回当前队列中的元素,然后消费最小的一个,这样就能保证FIFO
		
### 2 JAVA 实例
+ 单点模拟
		public class FIFOQueueOne {
			public static void doOne() throws Exception{
				String host = "192.168.159.132:2181";
				ZooKeeper zk = connection(host);
				initQueue(zk);
				produce(zk, 1);
				produce(zk, 2);
				
				cosume(zk);
				cosume(zk);
				cosume(zk);
				
				zk.close();
			}
			public static ZooKeeper connection(String host) throws Exception{
				 return new ZooKeeper(host, 60000, new Watcher() {
						public void process(WatchedEvent event) {
						}
				  });
			}
			public static void initQueue(ZooKeeper zk) throws Exception {
				if(zk.exists("/queue-fifo", false) == null){
					System.out.println("create /queue-fifo task-queue-fifo");
					zk.create("/queue-fifo", "task-queue-fifo".getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
				}else{
					System.out.println("/queue-fifo is exist");
				}
			}
			public static void produce(ZooKeeper zk, int x) throws Exception {
				System.out.println("create /queue-fifo/x" + x + " x" + x);
				zk.create("/queue-fifo/x"+x, ("x"+x).getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL_SEQUENTIAL);
			}
			public static void cosume(ZooKeeper zk) throws Exception {
				List<String> list = zk.getChildren("/queue-fifo", true);
				if(list.size() > 0){
					long min = Long.MAX_VALUE;
					for(String num : list){
						if(min > Long.parseLong(num.substring(1))){
							min = Long.parseLong(num.substring(1));
						}
					}
					System.out.println("delete /queue-fifo/x" + min);
					zk.delete("/queue-fifo/x"+min, 0);
				} else {
					System.out.println("No node to cosume");
				}
			}
			/**
			 * @param args
			 * @throws Exception 
			 */
			public static void main(String[] args) throws Exception {
				doOne();
			}
		}
 
+ 分布式模拟
		public static void doAction(int client) throws Exception {
			String host1 = "192.168.159.132:2181";
			String host2 = "192.168.159.134:2181";
			String host3 = "192.168.159.135:2181";

			ZooKeeper zk = null;
			switch (client) {
			case 1:
				zk = connection(host1);
				initQueue(zk);
				produce(zk, 1);
				break;
			case 2:
				zk = connection(host2);
				initQueue(zk);
				produce(zk, 2);
				break;
			case 3:
				zk = connection(host3);
				initQueue(zk);
				cosume(zk);
				cosume(zk);
				cosume(zk);
				break;
			}
		}
		public static ZooKeeper connection(String host) throws Exception{
			 return new ZooKeeper(host, 60000, new Watcher() {
					public void process(WatchedEvent event) {
					}
			  });
		}
		public static void initQueue(ZooKeeper zk) throws Exception {
			if(zk.exists("/queue-fifo", false) == null){
				System.out.println("create /queue-fifo task-queue-fifo");
				zk.create("/queue-fifo", "task-queue-fifo".getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
			}else{
				System.out.println("/queue-fifo is exist");
			}
		}
		public static void produce(ZooKeeper zk, int x) throws Exception {
			System.out.println("create /queue-fifo/x" + x + " x" + x);
			zk.create("/queue-fifo/x"+x, ("x"+x).getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL_SEQUENTIAL);
		}
		public static void cosume(ZooKeeper zk) throws Exception {
			List<String> list = zk.getChildren("/queue-fifo", true);
			if(list.size() > 0){
				long min = Long.MAX_VALUE;
				for(String num : list){
					if(min > Long.parseLong(num.substring(1))){
						min = Long.parseLong(num.substring(1));
					}
				}
				System.out.println("delete /queue-fifo/x" + min);
				zk.delete("/queue-fifo/x"+min, 0);
			} else {
				System.out.println("No node to cosume");
			}
		}
		/**
		 * @param args
		 * @throws Exception 
		 */
		public static void main(String[] args) throws Exception {
			 doAction(Integer.parseInt(args[0]));
		}








