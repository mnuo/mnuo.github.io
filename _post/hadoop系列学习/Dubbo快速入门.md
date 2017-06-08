---
title: Dubbo快速入门
date: 2017-06-07 23:53:08 
tags: Hadoop
category: Hadoop
---
Dubbo是一个分布式服务框架,致力于提供高性能和透明化的BRC远程服务调用方案, 以及SOA服务治理方案。
其核心部分包含: 
+ 远程通讯: 提供对多种基于长连接的NIO框架抽象封装,包括多种线程模型,序列化,以及"请求-响应" 模式的信息交换方式
+ 集群容错: 提供基于接口方法的透明远程过程调用,包过多种协议支持,以及软负载均衡,失败容错,地址路由,动态配置等集群支持
+ 自动发现: 基于注册中心目录服务,使服务消费方能动态的查找服务提供方,试地址透明,使服务提供方可以平滑增加或减少机器

### Dobbo + Zookeeper + spring集成
+ 服务提供dubboserver
	- pom.xml
			<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
			  <modelVersion>4.0.0</modelVersion>
			  <groupId>com.monuo</groupId>
			  <artifactId>dubboserver</artifactId>
			  <version>0.0.1-SNAPSHOT</version>
			  <packaging>war</packaging>
			  <properties>  
				  <!-- spring版本号 -->  
				  <spring.version>4.3.3.RELEASE</spring.version>  
				  <!-- mybatis版本号 -->  
				  <hibernate.version>4.3.1.Final</hibernate.version>  
				  <!-- log4j日志文件管理包版本 -->  
				  <slf4j.version>1.7.7</slf4j.version>  
				  <log4j.version>1.2.17</log4j.version>  
				  <commons.fileupload.version>1.2.1</commons.fileupload.version>  
				  <!-- 文件拷贝时的编码 -->  
				  <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>  
				  <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>  
				  <!-- 编译时的编码 -->  
				  <maven.compiler.encoding>UTF-8</maven.compiler.encoding>
					<!-- <project.deploy>deploy</project.deploy>  
					<project.tomcat.version>8.0.33</project.tomcat.version>   -->
			  </properties>
				<dependencies>
				<dependency>    
				   <groupId>junit</groupId>    
				   <artifactId>junit</artifactId>    
				   <version>4.11</version>    
				   <!-- 表示开发的时候引入，发布的时候不会加载此包 -->    
				   <scope>test</scope>    
			   </dependency> 
				<!-- Servlet+JSP+JSTL -->
				<dependency>
					<groupId>javax.servlet</groupId>
					<artifactId>javax.servlet-api</artifactId>
					<version>3.1.0</version>
				</dependency>
				<dependency>
					<groupId>javax.servlet.jsp</groupId>
					<artifactId>javax.servlet.jsp-api</artifactId>
					<version>2.3.1</version>
				</dependency>
				<dependency>
					<groupId>javax.servlet</groupId>
					<artifactId>jstl</artifactId>
					<version>1.2</version>
				</dependency>
				 <!-- spring核心包 -->    
				<dependency>    
					<groupId>org.springframework</groupId>    
					<artifactId>spring-core</artifactId>    
					<version>${spring.version}</version>    
				</dependency>    
				<dependency>    
					<groupId>org.springframework</groupId>    
					<artifactId>spring-web</artifactId>    
					<version>${spring.version}</version>    
				</dependency>    
				<dependency>    
					<groupId>org.springframework</groupId>    
					<artifactId>spring-oxm</artifactId>    
					<version>${spring.version}</version>    
				</dependency>    
				<dependency>    
					<groupId>org.springframework</groupId>    
					<artifactId>spring-tx</artifactId>    
					<version>${spring.version}</version>    
				</dependency>    
				<dependency>    
					<groupId>org.springframework</groupId>    
					<artifactId>spring-jdbc</artifactId>    
					<version>${spring.version}</version>    
				</dependency>    
				<dependency>    
					<groupId>org.springframework</groupId>    
					<artifactId>spring-webmvc</artifactId>    
					<version>${spring.version}</version>    
				</dependency>    
				<dependency>    
					<groupId>org.springframework</groupId>    
					<artifactId>spring-aop</artifactId>    
					<version>${spring.version}</version>    
				</dependency>    
				<dependency>    
					<groupId>org.springframework</groupId>    
					<artifactId>spring-orm</artifactId>    
					<version>${spring.version}</version>    
				</dependency>    
				<dependency>    
					<groupId>org.springframework</groupId>    
					<artifactId>spring-context-support</artifactId>    
					<version>${spring.version}</version>    
				</dependency>
				<dependency>    
					<groupId>org.springframework</groupId>    
					<artifactId>spring-test</artifactId>    
					<version>${spring.version}</version>    
				</dependency> 
			   <dependency>
					<groupId>com.alibaba</groupId>
					<artifactId>dubbo</artifactId>
					<version>2.5.3</version>
					<exclusions>
							<exclusion>
								<artifactId>spring</artifactId>   
								<groupId>org.springframework</groupId>  
							</exclusion>
							<exclusion>
								<artifactId>javassist</artifactId>   
								<groupId>org.javassist</groupId>  
							</exclusion>
							<exclusion>
								<artifactId>netty</artifactId>   
								<groupId>io.netty</groupId>  
							</exclusion>
						</exclusions>
				</dependency> 
				<dependency>
					<groupId>org.javassist</groupId>
					<artifactId>javassist</artifactId>
					<version>3.20.0-GA</version>
				</dependency> 
				<!-- https://mvnrepository.com/artifact/io.netty/netty-all -->
				<dependency>
					<groupId>io.netty</groupId>
					<artifactId>netty-all</artifactId>
					<version>4.1.11.Final</version>
				</dependency>
				<dependency>  
					<groupId>log4j</groupId>  
					<artifactId>log4j</artifactId>  
					<version>${log4j.version}</version>  
				</dependency>  
				<dependency>
					<groupId>com.101tec</groupId>
					<artifactId>zkclient</artifactId>
					<version>0.9</version>
				</dependency>
				<dependency>
					<groupId>org.apache.zookeeper</groupId>
					<artifactId>zookeeper</artifactId>
					<version>3.4.9</version>
					<exclusions>
							<exclusion>
								<artifactId>netty</artifactId>   
								<groupId>io.netty</groupId>  
							</exclusion>
						</exclusions>
				</dependency> 
				<dependency>
						<groupId>org.slf4j</groupId>
						<artifactId>slf4j-log4j12</artifactId>
						<version>${slf4j.version}</version>
					</dependency>
			  </dependencies>  		
				<!-- <build>  
				<plugins>  
					<plugin>  
						<groupId>org.apache.maven.plugins</groupId>  
						<artifactId>maven-jar-plugin</artifactId>  
						<version>2.6</version>  
						<configuration>  
							<archive>  
								<manifest>  
									<addClasspath>true</addClasspath>  
									<classpathPrefix>lib/</classpathPrefix>  
									<mainClass>com.xxg.Main</mainClass>  
								</manifest>  
							</archive>  
						</configuration>  
					</plugin>  
					<plugin>  
						<groupId>org.apache.maven.plugins</groupId>  
						<artifactId>maven-dependency-plugin</artifactId>  
						<version>2.10</version>  
						<executions>  
							<execution>  
								<id>copy-dependencies</id>  
								<phase>package</phase>  
								<goals>  
									<goal>copy-dependencies</goal>  
								</goals>  
								<configuration>  
									<outputDirectory>${project.build.directory}/lib</outputDirectory>  
								</configuration>  
							</execution>  
						</executions>  
					</plugin>  
			  
				</plugins>  
			</build>   -->
			<build>
				<plugins>  
					<plugin>  
						<groupId>org.apache.maven.plugins</groupId>  
						<artifactId>maven-compiler-plugin</artifactId>  
						<configuration>  
							<source>1.8</source>  
							<target>1.8</target>  
						</configuration>  
					</plugin>
					<plugin>  
							<groupId>org.apache.maven.plugins</groupId>  
							<artifactId>maven-resources-plugin</artifactId>  
							<version>2.6</version>  
							<configuration>  
								<encoding>UTF-8</encoding><!-- 指定编码格式，否则在DOS下运行mvn命令时当发生文件资源copy时将使用系统默认使用GBK编码 -->  
							</configuration>  
						</plugin>    
				</plugins> 
			</build>
			</project>
	- web.xml
			<?xml version="1.0" encoding="UTF-8"?>    
			<web-app xmlns="http://java.sun.com/xml/ns/javaee"  
				xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
				xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"  
				version="3.0"  
				metadata-complete="false">  
				
				<context-param>
					<param-name>contextConfigLocation</param-name>
					<param-value>classpath:spring/springmvc-config.xml;classpath:dubbo/*.xml</param-value>
				</context-param>
				<listener>
					<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
				</listener>
				<servlet>
					<servlet-name>DispatcherServlet</servlet-name>
					<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
					<init-param>
						<param-name>contextConfigLocation</param-name>
						<param-value>classpath:spring/springmvc-config.xml</param-value>
					</init-param>
					<load-on-startup>1</load-on-startup>
					<async-supported>true</async-supported>
				</servlet>
				<servlet-mapping>
					<servlet-name>DispatcherServlet</servlet-name>
					<url-pattern>/</url-pattern>
				</servlet-mapping>
				<filter>
					<filter-name>CharacterEncodingFilter</filter-name>
					<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
					<async-supported>true</async-supported>
					<init-param>
						<param-name>encoding</param-name>
						<param-value>UTF-8</param-value>
					</init-param>
					<init-param>
						<param-name>forceEncoding</param-name>
						<param-value>true</param-value>
					</init-param>
				</filter>
				<filter-mapping>
					<filter-name>CharacterEncodingFilter</filter-name>
					<url-pattern>/*</url-pattern>
				</filter-mapping>
			</web-app>
	- sprimgmvc-config.xml
			<?xml version="1.0" encoding="UTF-8"?>  
			<beans xmlns="http://www.springframework.org/schema/beans"  
				xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
				xmlns:p="http://www.springframework.org/schema/p"  
				xmlns:context="http://www.springframework.org/schema/context"  
				xmlns:tx="http://www.springframework.org/schema/tx" 
				xmlns:mvc="http://www.springframework.org/schema/mvc" 
				xmlns:cache="http://www.springframework.org/schema/cache" 
				xsi:schemaLocation="http://www.springframework.org/schema/beans    
									http://www.springframework.org/schema/beans/spring-beans-4.3.xsd    
									http://www.springframework.org/schema/context    
									http://www.springframework.org/schema/context/spring-context-4.3.xsd  
									http://www.springframework.org/schema/tx 
									http://www.springframework.org/schema/tx/spring-tx-4.3.xsd   
									http://www.springframework.org/schema/mvc    
									http://www.springframework.org/schema/mvc/spring-mvc-4.3.xsd
									http://www.springframework.org/schema/cache
									http://www.springframework.org/schema/cache/spring-cache-4.3.xsd">
				<!-- Handles HTTP GET requests for /resources/** by efficiently serving up static resources in the ${webappRoot}/resources directory -->  
				<mvc:resources mapping="/resources/**" location="/resources/" /> 
				<!-- Enables the Spring MVC @Controller programming model -->  
				<mvc:annotation-driven/>
				<!--spring 自动扫描org.monuo下边的所有类  -->  
				<context:component-scan base-package="com.monuo" />  
				<context:property-placeholder location="classpath:config.properties" />
				<!-- 自定义配置Service -->  
			  <!--   <beans:bean id="sayHelloService" class="org.monuo.dubboserver.service.DemoServiceImpl"/> --> 
			</beans>
	- dubbo-config.xml
			<?xml version="1.0" encoding="UTF-8"?>    
			<beans xmlns="http://www.springframework.org/schema/beans"    
				xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"    
				xsi:schemaLocation="http://www.springframework.org/schema/beans    
					http://www.springframework.org/schema/beans/spring-beans.xsd    
					http://code.alibabatech.com/schema/dubbo    
					http://code.alibabatech.com/schema/dubbo/dubbo.xsd">    
				<!--   系统项目名  
				<dubbo:application name="DubboDemo" /> 
				 注册中心    
				<dubbo:registry protocol="zookeeper" address="${able_zookeeper}"  />  
				 是否纳入调用统计报表（可选）  
				<dubbo:monitor protocol="registry"/>  
				 协议  
				<dubbo:protocol name="dubbo" port="31581" /> 
				
				服务者与消费者的默认配置  
				延迟到Spring初始化完成后，再暴露服务,服务调用超时设置为6秒,超时不重试      
				<dubbo:provider delay="-1" timeout="6000" retries="0"/>  
				<dubbo:consumer timeout="6000" retries="0"/>  
				
				 -->
				<!-- 具体的实现bean -->  
				<bean id="demoService" class="com.monuo.dubboserver.service.DemoServiceImpl" />  
			  
				<!-- 提供方应用信息，用于计算依赖关系 -->  
				<dubbo:application name="xs_provider" />  
			  
				<!-- 使用multicast广播注册中心暴露服务地址 -->  
				<!-- <dubbo:registry address="multicast://224.5.6.7:1234" />  --> 
				  
				<!-- 使用zookeeper注册中心暴露服务地址 ,即zookeeper的所在服务器ip地址和端口号 -->  
				<dubbo:registry address="${able_zookeeper}" />  
			  
				<!-- 用dubbo协议在20880端口暴露服务 -->  
				<dubbo:protocol name="dubbo" port="20880"/>  
				<dubbo:provider delay="-1" timeout="1200000" retries="0"/>  
				<!-- 声明需要暴露的服务接口 -->  
				<dubbo:service interface="com.monuo.dubboserver.service.DemoService"  ref="demoService"/>  
			</beans>
	- java 类
			public class User implements Serializable {
				private static final long serialVersionUID = -1776596657814853993L;

				private long id;
				private String name;
				private int age;
				private String sex;
				...
			}
			public interface DemoService {
				public String sayHello(String name);
				public List<User> getUsers();  
			}

			public class DemoServiceImpl implements DemoService {
				@Override
				public List<User> getUsers() {
					List<User> list = new ArrayList<User>();  
					User u1 = new User();  
					u1.setName("hejingyuan");  
					u1.setAge(20);  
					u1.setSex("f");  
			  
					User u2 = new User();  
					u2.setName("xvshu");  
					u2.setAge(21);  
					u2.setSex("m");  
			  
					  
					list.add(u1);  
					list.add(u2);  
					  
					return list;  
				}
				@Override
				public String sayHello(String name) {
					return "Hello " + name;  
				}
			}
+ 消费者端: 将服务端的接口打成jar引入消费端
		- consumer.xml
				<?xml version="1.0" encoding="UTF-8"?>    
				<beans xmlns="http://www.springframework.org/schema/beans"    
					xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"    
					xsi:schemaLocation="http://www.springframework.org/schema/beans    
						http://www.springframework.org/schema/beans/spring-beans.xsd    
						http://code.alibabatech.com/schema/dubbo    
						http://code.alibabatech.com/schema/dubbo/dubbo.xsd">  
						
					<!-- 消费方应用名，用于计算依赖关系，不是匹配条件，不要与提供方一样 -->  
					<dubbo:application name="monuo_consumer" />  
				  
					<!-- 使用zookeeper注册中心暴露服务地址 -->  
					<!-- <dubbo:registry address="multicast://10.185.49.17:1234" />   -->
					<dubbo:registry address="${able_zookeeper}" />  
				  
					<!-- 生成远程服务代理，可以像使用本地bean一样使用demoService -->  
					<dubbo:reference id="demoService" interface="com.monuo.dubboserver.service.DemoService" />  
				</beans>
		- java 代码		
				public static void main(String[] args) {
					ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext(new String[]{"spring/springmvc-config.xml","dubbo/consumer.xml"});
					context.start();
					DemoService demoService = (DemoService) context.getBean("demoService");
					String hello = demoService.sayHello("hejingyuan");  
					System.out.println(hello);  
			  
					List<User> list = demoService.getUsers();  
					if (list != null && list.size() > 0) {  
						for (int i = 0; i < list.size(); i++) {  
							System.out.println(list.get(i));  
						}  
					} 
				}						
+ 运行结果:
		...
		Hello hejingyuan
		2017-06-08 13:19:29 [com.alibaba.dubbo.remoting.transport.DecodeHandler]-[DEBUG]  [DUBBO] Decode decodeable message com.alibaba.dubbo.rpc.protocol.dubbo.DecodeableRpcResult, dubbo version: 2.5.3, current host: 10.185.49.17
		User [id=0, name=hejingyuan, age=20]
		User [id=0, name=xvshu, age=21]
		...

参考文章:
http://blog.csdn.net/top_code/article/details/51010614