---
title: Redis 入门基础(1)
date: 2017-06-08 23:53:08 
tags: Hadoop
category: Hadoop
---
Redis 是一个开源的,高效的key-value存储系统,也是nosql中最常见的一种,Redis非常适合用来做缓存系统。
Redis支持多种语言的调用,官方推荐的java版客户端是Jedis,它非常强大和稳定,支持事务,管道及有Jedis自己的实现,我们对Redis数据操作都可以用Jedis来完成。

### Redis 安装
+ 下载编译
		hadoop@master:~$ wget http://download.redis.io/releases/redis-3.2.9.tar.gz
		hadoop@master:~$ tar -zxvf redis-3.2.9.tar.gz
		hadoop@master:~$ cd redis-3.2.9/
		hadoop@master:~/redis-3.2.9$ make
+ 启动
		hadoop@master:~/redis-3.2.9$ src/redis-server
		26094:C 16 May 22:15:10.234 # Warning: no config file specified, using the default config. In order to specify a config file use src/redis-server /path/to/redis.conf
		26094:M 16 May 22:15:10.236 * Increased maximum number of open files to 10032 (it was originally set to 1024).
						_._
				   _.-``__ ''-._
			  _.-``    `.  `_.  ''-._           Redis 3.2.9 (00000000/0) 64 bit
		  .-`` .-```.  ```\/    _.,_ ''-._
		 (    '      ,       .-`  | `,    )     Running in standalone mode
		 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
		 |    `-._   `._    /     _.-'    |     PID: 26094
		  `-._    `-._  `-./  _.-'    _.-'
		 |`-._`-._    `-.__.-'    _.-'_.-'|
		 |    `-._`-._        _.-'_.-'    |           http://redis.io
		  `-._    `-._`-.__.-'_.-'    _.-'
		 |`-._`-._    `-.__.-'    _.-'_.-'|
		 |    `-._`-._        _.-'_.-'    |
		  `-._    `-._`-.__.-'_.-'    _.-'
			  `-._    `-.__.-'    _.-'
				  `-._        _.-'
					  `-.__.-'

		...
		26094:M 16 May 22:15:10.293 * The server is now ready to accept connections on port 6379
+ 验证
		hadoop@master:~/redis-3.2.9$ src/redis-cli
		127.0.0.1:6379> set foo bar
		OK
		127.0.0.1:6379> get foo
		"bar"

### Redis快速入门
+ 字符串
		127.0.0.1:6379> set name 'baidu'
		OK
		127.0.0.1:6379> get name
		"baidu"
+ 哈希
		127.0.0.1:6379> HMSET user:1 username yiibai password yiibai points 200
		OK
		127.0.0.1:6379> HGETALL user:1
		1) "username"
		2) "yiibai"
		3) "password"
		4) "yiibai"
		5) "points"
		6) "200"
+ 列表
		127.0.0.1:6379> lpush tutoriallist redis
		(integer) 1
		127.0.0.1:6379> lpush tutoriallist mongodb
		(integer) 2
		127.0.0.1:6379> lpush tutoriallist rabitmq
		(integer) 3
		127.0.0.1:6379> lrange tutoriallist 0 10
		1) "rabitmq"
		2) "mongodb"
		3) "redis"
		127.0.0.1:6379>
+ 集合
		127.0.0.1:6379> sadd tutoriallist redis
		(error) WRONGTYPE Operation against a key holding the wrong kind of value
		127.0.0.1:6379> del tutoriallist
		(integer) 1
		127.0.0.1:6379> sadd tutoriallist redis
		(integer) 1
		127.0.0.1:6379> sadd tutoriallist mongodb
		(integer) 1
		127.0.0.1:6379> sadd tutoriallist rabitmq
		(integer) 1
		127.0.0.1:6379> sadd tutoriallist rabitmq
		(integer) 0
		127.0.0.1:6379> smembers tutoriallist
		1) "rabitmq"
		2) "mongodb"
		3) "redis"
+ 有序集合
		127.0.0.1:6379> del tutoriallist
		(integer) 1
		127.0.0.1:6379> zadd tutoriallist 0 redis
		(integer) 1
		127.0.0.1:6379> zadd tutoriallist 0 mongodb
		(integer) 1
		127.0.0.1:6379> zadd tutoriallist 0 rabitmq
		(integer) 1
		127.0.0.1:6379> zadd tutoriallist 0 rabitmq
		(integer) 0
		127.0.0.1:6379> ZRANGEBYSCORE tutoriallist 0 1000
		1) "mongodb"
		2) "rabitmq"
		3) "redis"
+ Redis-keys
		
+ Redis-Strings
		127.0.0.1:6379> set yiibai redis
		OK
		127.0.0.1:6379> get yiibai
		"redis"
		127.0.0.1:6379> del yiibai
		(integer) 1
		127.0.0.1:6379> get yiibai

+ Redis-哈希
		127.0.0.1:6379> HMSET yiibai name "redis tutorial" description "redis basic commands for caching" likes 20 visitors 23000
		OK
		127.0.0.1:6379> hgetall yiibai
		1) "name"
		2) "redis tutorial"
		3) "description"
		4) "redis basic commands for caching"
		5) "likes"
		6) "20"
		7) "visitors"
		8) "23000"

+ Redis-列表
+ Redis-集合
+ Redis-有序集
		127.0.0.1:6379> ZADD tutorials 1 redis
		(integer) 1
		127.0.0.1:6379> ZADD tutorials 2 mongodb
		(integer) 1
		127.0.0.1:6379> ZADD tutorials 3 rabitmq
		(integer) 1
		127.0.0.1:6379> ZADD tutorials 3 rabitmq
		(integer) 0
		127.0.0.1:6379> ZADD tutorials 4 rabitmq
		(integer) 0
		127.0.0.1:6379> zrange tutorials 0 10 withscores
		1) "redis"
		2) "1"
		3) "mongodb"
		4) "2"
		5) "rabitmq"
		6) "4"

+ Redis- HyperLogLog
		127.0.0.1:6379> PFADD tutorials "redis"
		(integer) 1
		127.0.0.1:6379> PFADD tutorials "mongodb"
		(integer) 1
		127.0.0.1:6379> PFADD tutorials "mysql"
		(integer) 1
		127.0.0.1:6379> pfcount tutorials

+ Redis-订阅
		127.0.0.1:6379> SUBSCRIBE redisChat
		Reading messages... (press Ctrl-C to quit)
		1) "subscribe"
		2) "redisChat"
		3) (integer) 1
+ Redis - 事务
		27.0.0.1:6379> MULTI
		OK
		127.0.0.1:6379> SET tutorial redis
		QUEUED
		127.0.0.1:6379> get turorial
		QUEUED
		127.0.0.1:6379> incr visitors
		QUEUED
		127.0.0.1:6379> EXEC
		1) OK
		2) (nil)
		3) (integer) 1
+ Redis - 脚本
		127.0.0.1:6379> EVAL "return {KEYS[1],KEYS[2],ARGV[1],ARGV[2]}" 2 key1 key2 first second
		1) "key1"
		2) "key2"
		3) "first"
		4) "second"
+ Redis - 连接
		127.0.0.1:6379> PING
		PONG
+ Redis - 备份
		127.0.0.1:6379> SAVE
		OK
+ 还原Redis数据
		要恢复Redis的数据只需移动 Redis 的备份文件（dump.rdb）到 Redis 目录，然后启动服务器。为了得到你的 Redis 目录，使用配置命令如下所示：
		127.0.0.1:6379> CONFIG get dir
		1) "dir"
		2) "/user/yiibai/redis-2.8.13/src"
		在上述命令的输出在 /user/yiibai/redis-2.8.13/src 目录，在安装redis的服务器安装位置。
+ Bgsave
		要创建Redis的备份备用命令BGSAVE也可以。这个命令将开始执行备份过程，并在后台运行。
		127.0.0.1:6379> BGSAVE
		Background saving started
+ Redis - 安全
		可以Redis的数据库更安全，所以相关的任何客户端都需要在执行命令之前进行身份验证。客户端输入密码匹配需要使用Redis设置在配置文件中的密码。
		127.0.0.1:6379> CONFIG get requirepass
		1) "requirepass"
		2) ""
		默认情况下，此属性为空，表示没有设置密码，此实例。您可以通过执行以下命令来更改这个属性
		127.0.0.1:6379> CONFIG set requirepass "yiibai"
		OK
		127.0.0.1:6379> CONFIG get requirepass
		1) "requirepass"
		2) "yiibai"

+ 设置密码，
		如果客户端运行命令没有验证，会提示（错误）NOAUTH，需要通过验证。错误将返回客户端。因此，客户端需要使用AUTHcommand进行认证。
		127.0.0.1:6379> AUTH password
+ Redis - 基准
		Redis基准是公用工具同时运行Ñ命令检查Redis的性能。
		下面给出的例子检查redis调用100000命令。
		redis-benchmark -n 100000

		PING_INLINE: 141043.72 requests per second
		PING_BULK: 142857.14 requests per second
		SET: 141442.72 requests per second
		GET: 145348.83 requests per second
		INCR: 137362.64 requests per second
		LPUSH: 145348.83 requests per second
		LPOP: 146198.83 requests per second
		SADD: 146198.83 requests per second
		SPOP: 149253.73 requests per second
		LPUSH (needed to benchmark LRANGE): 148588.42 requests per second
		LRANGE_100 (first 100 elements): 58411.21 requests per second
		LRANGE_300 (first 300 elements): 21195.42 requests per second
		LRANGE_500 (first 450 elements): 14539.11 requests per second
		LRANGE_600 (first 600 elements): 10504.20 requests per second
		MSET (10 keys): 93283.58 requests per second
+ Redis - 客户端连接
		Redis接受配置监听TCP端口和Unix套接字客户端的连接，如果启用。当一个新的客户端连接被接受以下操作进行：
		客户端套接字置于非阻塞状态，因为Redis使用复用和非阻塞I/O操作。
		TCP_NODELAY选项设定是为了确保我们没有在连接时延迟。
		创建一个可读的文件时，这样Redis能够尽快收集客户端的查询作为新的数据可供读取的套接字。

		客户端的最大数量
		在Redis的配置（redis.conf）属性调用maxclients，它描述客户端可以连接到Redis的最大数量。命令的基本语法是：
		config get maxclients

		1) "maxclients"
		2) "10000"

		默认情况下，此属性设置为10000（这取决于操作系统的文件描述符限制最大数量），但你可以改变这个属性。
		例子
		在下面给出的例子中，在启动服务器我们设置客户端的最大数量为10万。
		redis-server --maxclients 100000
+ Redis - 管道传输
		Redis是一个TCP服务器，并支持请求/响应协议。在redis一个请求完成下面的步骤：
			客户端发送一个查询到服务器，并从套接字中读取，通常在阻塞的方式，对服务器的响应。
			服务器处理命令并将响应返回给客户端。
		管道传输的含义
			管道的基本含义是，客户端可以发送多个请求给服务器，而无需等待答复所有，并最后读取在单个步骤中的答复。
		要检查redis的管道，只要启动Redis实例，然后在终端键入以下命令。
		$(echo -en "PING\r\n SET tutorial redis\r\nGET tutorial\r\nINCR visitor\r\nINCR visitor\r\nINCR visitor\r\n"; sleep 10) | nc localhost 6379
		+PONG
		+OK
		redis
		:1
		:2
		:3
### Jedis 客户端
#### 基本用法
+ pom.xml
		<dependency>
			<groupId>redis.clients</groupId>
				<artifactId>jedis</artifactId>
				<version>2.9.0</version>
		</dependency>

+ JedisDemo
		public static void main(String[] args) {
			Jedis jedis = null;
			try {
				jedis = JedisPoolManager.getInstance().getResource();
				jedis.auth("123123");
				
				//simple key - value
				jedis.set("redis", "myredis");
				System.out.println(jedis.get("redis"));
				
				jedis.append("redis", "yourredis");
				jedis.append("mq", "RabitMq");
				System.out.println("mq: " + jedis.get("redis"));
				
				//incr
				String pv = jedis.set("pv", "0");
				System.out.println("pv: " + pv);
				jedis.incr("pv");
				jedis.incrBy("pv", 10);
				System.out.println("pv: " + pv);
				
				//mset
				jedis.mset("firstname", "ricky", "lastname", "Fung");
				System.out.println(jedis.mget("firstname", "lastname"));
				
				//map
				Map<String, String> city = new HashMap<>();
				city.put("beijing", "1");
				city.put("shanghai", "2");
				
				jedis.hmset("city", city);
				System.out.println(jedis.hget("city", "beijing"));
				System.out.println(jedis.hlen("city"));
				System.out.println(jedis.hmget("city", "beijing","shanghai"));
				
				//list
				jedis.lpush("hobbies", "reading");
				jedis.lpush("hobbies", "basketball");
				jedis.lpush("hobbies", "shopping");
				
				List<String> hobbies = jedis.lrange("hobbies", 0, -1);
				System.out.println("hobbies: " + hobbies);
				
				//set 
				jedis.sadd("name", "ricky");
				jedis.sadd("name", "kings");
				jedis.sadd("name", "demon");
				
				System.out.println("size:"+jedis.scard("name"));
				System.out.println("exists:"+jedis.sismember("name", "ricky"));
				System.out.println(String.format("all members: %s", jedis.smembers("name")));
				System.out.println(String.format("rand member: %s", jedis.srandmember("name")));

				//remove
				jedis.srem("name", "demon");
				
				//hset
				jedis.hset("address", "country", "CN");
				jedis.hset("address", "province", "BJ");
				jedis.hset("address", "city", "Beijing");
				jedis.hset("address", "district", "Chaoyang");
				
				System.out.println("city:"+jedis.hget("address", "city"));
				System.out.println("keys:"+jedis.hkeys("address"));
				System.out.println("values:"+jedis.hvals("address"));
				
				  //zadd
				jedis.zadd("gift", 0, "car"); 
				jedis.zadd("gift", 0, "bike"); 
				Set<String> gift = jedis.zrange("gift", 0, -1);
				System.out.println("gift:"+gift);
			} finally {
				if(jedis != null) {
				  jedis.close();
				}
			}
			JedisPoolManager.getInstance().destroy();
		}

+ JedisPoolManager
		public class JedisPoolManager {
			public static Logger logger = Logger.getLogger(JedisPoolManager.class);
			private volatile static JedisPoolManager manager;
			private final JedisPool pool;
			
			private JedisPoolManager() {
				try {
					//加载redis配置
					Properties props = PropertyUtils.load("redis/redis.properties");
					
					//创建Jedis池配置实例
					JedisPoolConfig config = new JedisPoolConfig();
					
					//设置池配置项
					String maxTotal = props.getProperty("redis.pool.maxTotal", "4");
					config.setMaxTotal(Integer.parseInt(maxTotal));
					
					String maxIdle = props.getProperty("redis.pool.maxIdle", "4");
					config.setMaxIdle(Integer.parseInt(maxIdle));
					
					String minIdle = props.getProperty("redis.pool.minIdle", "1");
					config.setMinIdle(Integer.parseInt(minIdle));
					
					String maxWaitMillis = props.getProperty("redis.pool.maxWaitMillis ", "1");
					config.setMaxWaitMillis (Long.parseLong(maxWaitMillis));
					
					String testOnBorrow = props.getProperty("redis.pool.testOnBorrow", "true");
					config.setTestOnBorrow("true".equals(testOnBorrow));
					
					String testOnReturn = props.getProperty("redis.pool.testOnReturn", "true");
					config.setTestOnReturn("true".equals(testOnReturn));
					
					String server = props.getProperty("redis.server");
					if(StringUtils.isEmpty(server)){
						throw new IllegalArgumentException("JedisPool redis.server is empty!");
					}
					
					String[] host_arr = server.split(",");
					if(host_arr.length>1){
						throw new IllegalArgumentException("JedisPool redis.server length > 1");
					}

					String[] arr = host_arr[0].split(":");
					//根据配置实例化jedis池
					logger.info("********** init Jedis pool ***********");
					logger.info("host-> " + arr[0] + ", port-> " + arr[1]);
					
					pool = new JedisPool(config, arr[0], Integer.parseInt(arr[1]));
				} catch (Exception e) {
					throw new IllegalArgumentException("init JedisPool error", e);
				}
			}
			
			public static JedisPoolManager getInstance(){
				if(manager == null){
					synchronized (JedisPoolManager.class) {
						if(manager == null){
							manager = new JedisPoolManager();
						}
					}
				}
				return manager;
			}
			
			public Jedis getResource(){
				return pool.getResource();
			}
			
			public void destroy(){
				pool.destroy();
			}
			
			public void close(){
				pool.close();
			}
		}
+ 问题解决
	- Caused by: java.util.NoSuchElementException: Unable to validate object
			testOnBorrow：在borrow一个jedis实例时，是否提前进行validate操作；如果为true，则得到的jedis实例均是可用的,改成false
			修改redis.config 注释掉bind: 127.0.0.1
	- Exception in thread "main" 
			redis.clients.jedis.exceptions.JedisDataException: 
			DENIED Redis is running in protected mode because protected mode is enabled,
			 no bind address was specified, 
			 no authentication password is requested to clients. 
			 In this mode connections are only accepted from the loopback interface.
			 If you want to connect from external computers to Redis you may adopt one of the following solutions: 
			 1) Just disable protected mode sending the command 'CONFIG SET protected-mode no' from the loopback interface by connecting to Redis from the same host the server is running, however MAKE SURE Redis is not publicly accessible from internet if you do so.
			 Use CONFIG REWRITE to make this change permanent. 
			 2) Alternatively you can just disable the protected mode by editing the Redis configuration file, and setting the protected mode option to 'no', and then restarting the server. 
			 3) If you started the server manually just for testing, restart it with the '--protected-mode no' option.
			 4) Setup a bind address or an authentication password. NOTE: You only need to do one of the above things in order for the server to start accepting connections from the outside.
			
			根据4添加密码
			redis 127.0.0.1:6379> 
			redis 127.0.0.1:6379> config get requirepass
			1) "requirepass"
			2) (nil)
			显示密码是空的，
			然后设置密码：
			redis 127.0.0.1:6379> config set requirepass test123
			OK
			再次查询密码：
			redis 127.0.0.1:6379> config get requirepass
			(error) ERR operation not permitted
			此时报错了！
			现在只需要密码认证就可以了。
			redis 127.0.0.1:6379> auth test123
			OK
			再次查询密码：
			redis 127.0.0.1:6379> config get requirepass
			1) "requirepass"
			2) "test123"
			密码已经得到修改。
			
参考文章
