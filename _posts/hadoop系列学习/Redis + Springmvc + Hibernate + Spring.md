---
title: Redis + Springmvc + Hibernate + Spring
date: 2017-06-13 13:53:08 
tags: Hadoop
category: Hadoop
---
SSH整合Redis 使用到的jar commons-pool2-2.4.2.jar、jedis-2.3.1.jar、spring-data-redis-1.3.4.RELEASE.jar。redis桌面管理工具Redis Desktop Manager
+ 1 pom.xml
		<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
		  <modelVersion>4.0.0</modelVersion>
		  <groupId>org.monuo</groupId>
		  <artifactId>sshredis</artifactId>
		  <version>0.0.1-SNAPSHOT</version>
		  <packaging>war</packaging>
		  <name>sshredis</name>
		  <build>
			<finalName>brieflife</finalName>
			<plugins>
				<plugin>  
					 <groupId>org.apache.maven.plugins</groupId>  
					 <artifactId>maven-compiler-plugin</artifactId>  
					 <version>3.1</version>  
					 <configuration>  
						 <source>1.8</source>  
						 <target>1.8</target>  
					 </configuration>  
				 </plugin> 
			</plugins>
		  </build>
		  <properties>  
			  <!-- spring版本号 -->  
			  <spring.version>4.3.3.RELEASE</spring.version>  
			  <!-- hibernate版本号 -->  
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
			<!-- spring jar end -->
			<!-- hiberante 4 -->
			<dependency>
				<groupId>org.hibernate</groupId>
				 <artifactId>hibernate-core</artifactId>
				 <version>${hibernate.version}</version>
			 </dependency>
			 <dependency>
				 <groupId>org.hibernate</groupId>
				 <artifactId>hibernate-validator</artifactId>
				 <version>${hibernate.version}</version>
			 </dependency>
			 <dependency>
				 <groupId>org.hibernate</groupId>
				 <artifactId>hibernate-entitymanager</artifactId>
				 <version>${hibernate.version}</version>
			 </dependency>
			 <dependency>
				 <groupId>javax.validation</groupId>
				 <artifactId>validation-api</artifactId>
				 <version>1.0.0.GA</version>
				 <scope>provided</scope>
			 </dependency>
			 <!-- hibernate jar end -->
			 <!-- commons配置 -->
			 <dependency>    
			   <groupId>commons-fileupload</groupId>    
			   <artifactId>commons-fileupload</artifactId>    
			   <version>1.3.1</version>    
			 </dependency>    
			 <dependency>    
				<groupId>commons-io</groupId>    
				<artifactId>commons-io</artifactId>    
				<version>2.4</version>    
			 </dependency>    
			 <dependency>    
				<groupId>commons-codec</groupId>    
				<artifactId>commons-codec</artifactId>    
				<version>1.9</version>    
			 </dependency>
			 <!-- c3p0 -->
			 <dependency>
				 <groupId>c3p0</groupId>
				 <artifactId>c3p0</artifactId>
				 <version>0.9.1.2</version>
			 </dependency>
			 <!-- Mysql数据库链接jar包 -->    
			<dependency>    
				<groupId>mysql</groupId>    
				<artifactId>mysql-connector-java</artifactId>    
				<version>5.1.30</version>    
			</dependency> 
			<!-- 格式化对象，方便输出日志 -->    
			<dependency>    
				<groupId>com.alibaba</groupId>    
				<artifactId>fastjson</artifactId>    
				<version>1.1.41</version>    
			</dependency>    	
			<dependency>    
				<groupId>org.slf4j</groupId>    
				<artifactId>slf4j-api</artifactId>    
				<version>${slf4j.version}</version>    
			</dependency>    
			<dependency>    
				<groupId>org.slf4j</groupId>    
				<artifactId>slf4j-log4j12</artifactId>    
				<version>${slf4j.version}</version>    
			</dependency>    
			<!-- Spring 升级4+ 依赖的JSON包 -->
			<dependency>
				<groupId>com.fasterxml.jackson.core</groupId>
				<artifactId>jackson-databind</artifactId>
				<version>2.7.4</version>
			</dependency>
			<dependency>
				<groupId>com.fasterxml.jackson.core</groupId>
				<artifactId>jackson-core</artifactId>
				<version>2.7.4</version>
			</dependency>
			<dependency>
				<groupId>com.fasterxml.jackson.core</groupId>
				<artifactId>jackson-annotations</artifactId>
				<version>2.7.4</version>
			</dependency>
			<!-- /Spring 升级4+ 依赖的JSON包 -->
			<!-- https://mvnrepository.com/artifact/commons-httpclient/commons-httpclient -->
			<dependency>
				<groupId>commons-httpclient</groupId>
				<artifactId>commons-httpclient</artifactId>
				<version>3.1</version>
			</dependency>
			<dependency>
				<groupId>com.thoughtworks.xstream</groupId>
				<artifactId>xstream</artifactId>
				<version>1.4.9</version>
			</dependency>
			<!-- https://mvnrepository.com/artifact/org.codehaus.jackson/jackson-mapper-asl -->
			<dependency>
				<groupId>org.codehaus.jackson</groupId>
				<artifactId>jackson-mapper-asl</artifactId>
				<version>1.9.13</version>
			</dependency>
			<!-- perf4j的依赖包 -->  
			<dependency>  
			  <groupId>org.perf4j</groupId>  
			  <artifactId>perf4j</artifactId>  
			  <version>0.9.16</version>  
			</dependency>  
			<dependency>  
				<groupId>org.springframework</groupId>  
				<artifactId>spring-aspects</artifactId>  
				<version>${spring.version}</version>  
			</dependency> 
			<!-- redis -->
			<dependency>
				<groupId>org.apache.commons</groupId>
				<artifactId>commons-pool2</artifactId>
				<version>2.4.2</version>
			</dependency>
			<dependency>
				<groupId>redis.clients</groupId>
				<artifactId>jedis</artifactId>
				<version>2.9.0</version>
			</dependency>
			<dependency>  
				<groupId>log4j</groupId>  
				<artifactId>log4j</artifactId>  
				<version>${log4j.version}</version>  
			</dependency>
			<dependency>
				<groupId>org.springframework.data</groupId>
				<artifactId>spring-data-redis</artifactId>
				<version>1.8.4.RELEASE</version>
			</dependency>
		  </dependencies>
		</project>

代码地址:
https://github.com/mnuo/myhadoop/tree/master/sshredis

参考文章:
http://www.cnblogs.com/mstk/p/6226633.html
http://www.cnblogs.com/mstk/p/6252747.html
