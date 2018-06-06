---
title: 7、SpringCloud之消息总线(Spring Cloud Bus)
date: 2018-06-06 13:27:08 
tags: SpringCloud
category: SpringCloud
---
Spring Cloud Bus 将分布式的节点用轻量的消息代理连接起来。它可以用于广播配置文件的更改或者服务之间的通讯,也可以用于监控。
### 1 准备工作
+ 1, 下载安装Erlang : 
			http://www.erlang.org/downloads
+ 2, 下载安装rabbit: 
			http://www.rabbitmq.com/install-windows.html
+ 3, 安装RabbitMQ-Plugins
		这个相当于是一个管理界面，方便我们在浏览器界面查看RabbitMQ各个消息队列以及exchange的工作情况，
		安装方法是：打开命令行cd进入rabbitmq的sbin目录输入：rabbitmq-plugins enable rabbitmq_management命令，
		稍等会会发现出现plugins安装成功的提示，默认是安装6个插件
		在安装完之后, 页面打不开, 解决方法: 
		首先在命令行输入：rabbitmq-service stop，
		接着输入rabbitmq-service remove，
		再接着输入rabbitmq-service install，
		接着输入rabbitmq-service start，
		最后重新输入rabbitmq-plugins enable rabbitmq_management
		
+ 4, 验证 浏览器输入: http://localhost:15672/ 输入账号/密码: guest/guest
+ 5, 安装curl: https://curl.haxx.se/download.html 下载解压进入I3682 执行 curl
### 2 改造config-client		
+ 1, pom.xml
			<!-- 消息总线rabbitmq添加依赖 -->
			<dependency>
			  <groupId>org.springframework.retry</groupId>
			  <artifactId>spring-retry</artifactId>
			</dependency>
			<dependency>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-starter-aop</artifactId>
			</dependency>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-starter-bus-amqp</artifactId>
			</dependency>
			<dependency>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-starter-actuator</artifactId>
			</dependency>
		
+ 2, bootstrap.properties
			spring.application.name=config-client
			spring.cloud.config.label=master
			spring.cloud.config.profile=dev
			#spring.cloud.config.uri= http://localhost:8888/

			eureka.client.serviceUrl.defaultZone=http://localhost:8989/eureka/
			spring.cloud.config.discovery.enabled=true
			spring.cloud.config.discovery.serviceId=config-server
			server.port=8883
			#忽略权限拦截F:\downloads\curl-7.60.0\I386>curl -X POST http://localhost:8881/bus/refresh
			# {"timestamp":1528271611945,"status":401,"error":"Unauthorized","message":"Full authentication is required to access this resource.","path":"/bus/refresh"}
			management.security.enabled: false

			spring.rabbitmq.host=localhost
			spring.rabbitmq.port=5672
			spring.rabbitmq.username=guest
			spring.rabbitmq.password=guest
+ 3	依次启动eureka-server、confg-cserver,启动两个config-client，端口为：8881、8882。
	访问http://localhost:8881/hi 或者http://localhost:8882/hi 浏览器显示：foo version 22
	
+ 4, 修改config-client-dev.properties foo= foo version 44 , 执行刷新: curl -X POST http://localhost:8881/bus/refresh 访问http://localhost:8881/hi foo version 44, 然而http://localhost:8882/hi 浏览器显示：foo version 22

参考文章: https://blog.csdn.net/forezp/article/details/70148235
		https://blog.csdn.net/dl348/article/details/79445626
		https://www.cnblogs.com/xing901022/p/4652624.html
		https://blog.csdn.net/superdangbo/article/details/78784015
