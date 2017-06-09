---
title: Redis 入门基础(2)
date: 2017-06-09 12:53:08 
tags: Hadoop
category: Hadoop
---
我们除了可以使用redis.clients.jedis.Jedis.set(String key, String value) insert string之外，还可以使用redis.clients.jedis.BinaryJedis.set(byte[] key, byte[] value) 保存我们自定义的POJO类
+ 代码如下: 
		public class JedisPojoDemo {
			/**
			 * @param args
			 * @throws Exception 
			 */
			public static void main(String[] args) throws Exception {
				Jedis jedis = null;
				try {
					jedis = JedisPoolManager.getInstance().getResource();
					jedis.auth("123123");
					
					Person person = new Person("Ricky", 27);
					//序列化
					byte[] byteArray = serialize(person);
					
					//set 
					jedis.set("Ricky".getBytes(), byteArray);
					
					//get
					byteArray = jedis.get("Ricky".getBytes());
					
					//反序列化
					person = deserialize(byteArray);
					
					System.out.println(person);
				} finally {
					if(jedis != null) jedis.close();
				}
				JedisPoolManager.getInstance().destroy();
			}
			public static Person deserialize(byte[] bytes) throws Exception{
				ObjectInputStream ois = null;
				try {
					ByteArrayInputStream bis = new ByteArrayInputStream(bytes);
					ois = new ObjectInputStream(bis);
					return (Person) ois.readObject();
				} finally {
					ois.close();
				}
			}
			public static byte[] serialize(Person person) throws IOException{
				ByteArrayOutputStream bos = null;
				ObjectOutputStream oos = null;
				try {
					bos = new ByteArrayOutputStream();
					oos = new ObjectOutputStream(bos);
					oos.writeObject(person);
					oos.flush();
					return bos.toByteArray();
				} finally {
					oos.close();
					bos.close();
				}
			}
		}
		class Person implements Serializable{
			private static final long serialVersionUID = 5735319877294608703L;
			private String name;
			private int age;
			
			public Person() {
			}
			public Person(String name, int age){
				this.name = name;
				this.age = age;
			}
			public String getName() {
				return name;
			}
			public void setName(String name) {
				this.name = name;
			}
			public int getAge() {
				return age;
			}
			public void setAge(int age) {
				this.age = age;
			}
			@Override
			public String toString() {
				return "name: " + this.name + " age: " + this.age;
			}
		}

+ 运行结果: 
		name: Ricky age: 27

### 2 高级用法
+ 事务
		所谓事务,即一个连续操作,是否是执行一个事务,要么成功,要么失败,没有中间状态。而Redis事务很简单他主要目的是保障,一个client发起的事务中的命令可以连续的执行而中间不会插入其他的client的命令,也就是事务的连贯性。
		我们调用jedis.watch()方法来监控key,如果调用后key值发生变化,则整个事务会执行失败,另外事务中某个操作失败并不会回滚,可以使用discard()取消事务
		
		Jedis jedis = null;
		try {
			jedis = JedisPoolManager.getInstance().getResource();
			jedis.auth("123123");
			jedis.watch("foo");
			Transaction transaction = jedis.multi();
			transaction.set("foo", "bar");//断点到这里，先不执行。然后用redis-cli连接redis，修改foo的值为444，修改完后，该断点往下执行
			transaction.set("you", "you1");
			transaction.exec();//能执行，不会抛异常，但是foo=444，you=you。说明该事务被打断，不执行，设置you=you1失败了
			System.out.println(jedis.get("foo"));
			System.out.println(jedis.get("you"));
		} catch (Exception e) {
			e.printStackTrace();
		}finally {
			if(jedis != null) jedis.close();
		}
		JedisPoolManager.getInstance().destroy();
		运行结果: 
			4444
			you
+ 管道
		有时候你需要发送一系列不同的命令,其中最笨也是最有效的方法是,通过管道流水式发送,并且每次不需要等待响应,而是在这系列命令结束之后在获取响应
		Jedis jedis = null;
		try {
			jedis = JedisPoolManager.getInstance().getResource();
			jedis.auth("123123");
			Pipeline p = jedis.pipelined();
			p.set("fool", "bar");
			p.zadd("foo", 1, "barowitch");
			p.zadd("foo", 0, "barinasky");
			p.zadd("foo", 0, "barikoviev");
			Response<String> pipeString = p.get("fool");
			Response<Set<String>> setpipe = p.zrange("foo", 0, -1);
			p.sync();
			int setsize = setpipe.get().size();
			System.out.println("fool: " + pipeString.get());
			System.out.println("foo size: " + setsize);
			Set<String> set = setpipe.get();
			
			for(String string : set){
				System.out.println(string);
			}
			
		} finally {
			if(jedis != null) jedis.close();
		}
		JedisPoolManager.getInstance().getResource();
		运行结果:
		2017-06-09 11:11:51 [com.monuo.jedis.util.JedisPoolManager]-[INFO] host-> 192.168.159.132, port-> 6379
		fool: bar
		foo size: 3
		barikoviev
		barinasky
		barowitch
+ Publish/Subscribe
		To subscribe to a channel in Redis, create an instance of JedisPubSub and call subscribe on the Jedis instance:
		public class JedisSubsribeDemo {
			/**
			 * @param args
			 */
			public static void main(String[] args) {
				Jedis jedis = null;
				try {
					jedis = JedisPoolManager.getInstance().getResource();
					jedis.auth("123123");
					MyListener l = new MyListener();
					jedis.subscribe(l, "foo");
				}finally {
					if(jedis != null) jedis.close();
				}
				JedisPoolManager.getInstance().getResource();
			}

		}
		class MyListener extends JedisPubSub {
			public void onMessage(String channel, String message) {
				System.out.println("onMessage: channel: " + channel + " message: " + message);
			}

			public void onSubscribe(String channel, int subscribedChannels) {
				System.out.println("onSubscribe: channel: " + channel + " subscribedChannels: " + subscribedChannels);
			}

			public void onUnsubscribe(String channel, int subscribedChannels) {
				System.out.println("onUnsubscribe: channel: " + channel + " subscribedChannels: " + subscribedChannels);
			}

			public void onPSubscribe(String pattern, int subscribedChannels) {
				System.out.println("onPSubscribe: pattern: " + pattern + " subscribedChannels: " + subscribedChannels);
			}

			public void onPUnsubscribe(String pattern, int subscribedChannels) {
				System.out.println("onPUnsubscribe: pattern: " + pattern + " subscribedChannels: " + subscribedChannels);
			}

			public void onPMessage(String pattern, String channel,
				String message) {
				System.out.println("onPMessage: pattern: " + pattern + " channel: " + channel + " message: " + message);
			}
		}
		运行结果: 
		onSubscribe: channel: foo subscribedChannels: 1
### 3 redis 集群搭建
系统： Ubuntu 16.04
Redis版本：3.2.9
机器：
		master：192.168.159.132:7000(主节点)、192.168.159.132:7001(不一定是192.168.159.132:7000的从节点)
        slave1：192.168.159.134:7000(主节点)、192.168.159.134:7001(不一定是192.168.159.134:7000的从节点)
		slave2：192.168.159.135:7000(主节点)、192.168.159.135:7001(不一定是192.168.159.135:7000的从节点)
		
+ 1 master创建目录
		hadoop@master:~$ mkdir rediscluster
		hadoop@master:~$ chmod -R 777 rediscluster
		hadoop@master:~$ cd rediscluster
		hadoop@master:~/rediscluster$ mkdir 7000 7001 //分别对应redis的端口
		
+ 2 创建redis.config并加入以下内容
		hadoop@master:~/rediscluster/7000$ vim redis.config
		# 端口7000
		port 7000
		cluster-enabled yes
		cluster-config-file nodes.conf

		cluster-node-timeout 5000
		appendonly yes
		# 非保护模式,不设置这个本来没有设置这个参数结果创建集群抛出异常:
		# (error) DENIED Redis is running in protected mode because protected mode is enabled, ...
		# 1) ...
		# 2) Alternatively you can just disable the protected mode by editing the Redis configuration file,
		# and setting the protected mode option to 'no', and then restarting the server. 
		# 3) If you started the server manually just for testing, restart it with the '--protected-mode no' option. 
		# 4) Setup a bind address or an authentication password.
		# NOTE: You only need to do one of the above things in order for the server to start accepting connections from the outside.
		protected-mode no
		
		将redis.config cp到7001目录下修改port为7001
		hadoop@master:~/rediscluster/7000$ cp ./redis.conf ../7001/redis.conf
+ 3 将master上的rediscluster scp到slave1和slave2：
		scp -r rediscluster/ hadoop@slave1:~/
		scp -r rediscluster/ hadoop@slave2:~/
+ 4 安装redis(参考上篇文章),然后将redis都scp到slave1,slave2	
		hadoop@master:~$ scp -r redis-3.2.9/ hadoop@slave1:~/	
		hadoop@master:~$ scp -r redis-3.2.9/ hadoop@slave2:~/
+ 5 三台机器分别复制redis-3.2.9/src/redis-server到rediscluster
+ 6 分别进入master,slave1,slave2 的 7000和7001 启动redis
		../redis-server ./redis.conf 
+ 7 在master安装ruby:
		sudo apt-get update
		sudo apt-get install ruby
+ 8 在master 上安装ruby程序和redis的第三方接口(redis-3.3.0.gem下载地址: https://rubygems.org/gems/redis/versions/3.3.0)
		hadoop@master:~$ sudo gem install --local redis-3.3.0.gem
+ 9 在master的redis安装目录的src下创建集群
		hadoop@master:~/redis-3.2.9/src$ ./redis-trib.rb create --replicas 1 192.168.159.132:7000 192.168.159.132:7001  192.168.159.134:7000 192.168.159.134:7001 192.168.159.135:7000 192.168.159.135:7001
		>>> Creating cluster
		>>> Performing hash slots allocation on 6 nodes...
		Using 3 masters:
		192.168.159.132:7000
		192.168.159.134:7000
		192.168.159.135:7000
		Adding replica 192.168.159.134:7001 to 192.168.159.132:7000
		Adding replica 192.168.159.132:7001 to 192.168.159.134:7000
		Adding replica 192.168.159.135:7001 to 192.168.159.135:7000
		...
		[OK] All 16384 slots covered.

+ 10 查看是否成功
		.hadoop@master:~/redis-3.2.9/src$ ./redis-trib.rb check 192.168.159.132:7001
		>>> Performing Cluster Check (using node 192.168.159.132:7001)
		S: 82b2491d66302ae11ecf4b27eb8e532237988818 192.168.159.132:7001
		   slots: (0 slots) slave
		   replicates 35c429fefd3c1bbcb35ceae9a617be814c9e72b6
		S: 86d798e4f7bce7cf9f339aafaa56910bcd3310c5 192.168.159.134:7001
		   slots: (0 slots) slave
		   replicates 561f416f5e9127ccad6381d2e409ea642c31bab1
		M: 561f416f5e9127ccad6381d2e409ea642c31bab1 192.168.159.132:7000
		   slots:0-5460 (5461 slots) master
		   1 additional replica(s)
		M: be564fe41ddd6472318b2191a7c52f8ee3f9a979 192.168.159.135:7000
		   slots:10923-16383 (5461 slots) master
		   1 additional replica(s)
		M: 35c429fefd3c1bbcb35ceae9a617be814c9e72b6 192.168.159.134:7000
		   slots:5461-10922 (5462 slots) master
		   1 additional replica(s)
		S: 890de0abc46d1d6ed12d63a103ccc7460949daf5 192.168.159.135:7001
		   slots: (0 slots) slave
		   replicates be564fe41ddd6472318b2191a7c52f8ee3f9a979
		[OK] All nodes agree about slots configuration.
		>>> Check for open slots...
		>>> Check slots coverage...
		[OK] All 16384 slots covered.

### 4 使用JedisCluster操作集群,使用ShardedJedisPool时报错,网上说使用JedisCluster
https://samebug.io/exceptions/271929/redis.clients.jedis.exceptions.JedisMovedDataException/moved-1539-17231591036379#
ShardedJedis doesn't work with Redis Cluster. Use JedisCluster instead.
+ 测试代码
		String key = "2";  
		// 这东西 可以直接看到key 的分片数，就能知道放哪个 节点  
		System.out.println(JedisClusterCRC16.getSlot(key));  
		Set<HostAndPort> jedisClusterNodes = new HashSet<HostAndPort>();  
		jedisClusterNodes.add(new HostAndPort("192.168.159.132", 7000));  
		jedisClusterNodes.add(new HostAndPort("192.168.159.134", 7001));  
		jedisClusterNodes.add(new HostAndPort("192.168.159.135", 7002));  
		jc  = new JedisCluster(jedisClusterNodes);  
		System.out.println(jc .get(key));  
		jc .setnx(key, "bar");  
		String value = jc .get(key);  
		System.out.println(value);  
		for (int i = 1; i <= 10000; i++) {  
	         long start = System.currentTimeMillis();  
	         jc.set("k:" + i, "v" + i);  
	         System.out.print("set " + i +"th value in " + (System.currentTimeMillis() - start) + " ms");  
	         start = System.currentTimeMillis();  
	         String value = jc.get("k:" + i);  
	         System.out.println(", get " + i +"th value: " + value + " in " + (System.currentTimeMillis() - start) + " ms");  
	     }  
java代码地址: 
https://github.com/mnuo/myhadoop/tree/master/redis
参考文章:
http://blog.csdn.net/top_code/article/details/51292240
http://greemranqq.iteye.com/blog/2230137
http://blog.csdn.net/chenyuangege/article/details/51519370
http://blog.csdn.net/u012810317/article/details/51272432
http://blog.csdn.net/xiaoyangsavvy/article/details/72391844
		
