---
title: spring security 系列1
date: 2017-06-20 23:53:08 
tags: SSH
category: SSH
---
Spring Security是一种基于spring aop和servlet过滤器的安全框架;它提供全面的安全性解决方案,同时在web请求级和方法调用级处理身份确认和授权。在Spring framework基础上充分利用依赖注入(DI,dependency Injection)和面向切面技术。
### 1 Spring4+Springmvc4+SpringSecurity4入门配置
+ 用到的jar
		<!-- SpringSecurity jar-->
		<dependency>
			<groupId>org.springframework.security</groupId>
			<artifactId>spring-security-config</artifactId>
			<version>${spring.security.version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework.security</groupId>
			<artifactId>spring-security-core</artifactId>
			<version>${spring.security.version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework.security</groupId>
			<artifactId>spring-security-web</artifactId>
			<version>${spring.security.version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework.security</groupId>
			<artifactId>spring-security-acl</artifactId>
			<version>${spring.security.version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework.security</groupId>
			<artifactId>spring-security-taglibs</artifactId>
			<version>${spring.security.version}</version>
		</dependency>

+ web.xml 添加拦截配置
		<!-- spring security 过滤器-->
		<filter>
			<filter-name>springSecurityFilterChain</filter-name>
			<filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
		</filter>
		<filter-mapping>
			<filter-name>springSecurityFilterChain</filter-name>
			<url-pattern>/*</url-pattern>
		</filter-mapping>
+ spring-security.xml
		<?xml version="1.0" encoding="UTF-8"?>  
		<beans xmlns="http://www.springframework.org/schema/beans"  
			xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
			xmlns:security="http://www.springframework.org/schema/security"  
			xmlns:context="http://www.springframework.org/schema/context"  
			xsi:schemaLocation="http://www.springframework.org/schema/beans    
								http://www.springframework.org/schema/beans/spring-beans-4.3.xsd    
								http://www.springframework.org/schema/context    
								http://www.springframework.org/schema/context/spring-context-4.3.xsd
								http://www.springframework.org/schema/security
								http://www.springframework.org/schema/security/spring-security-4.0.xsd">   
			<!-- 对登录页面不进行拦截,后面带* 是因为这个请求页面后面可能有参数 -->
			<security:http pattern="/loginpage" security="none" />
			<security:http auto-config="true">
				<security:intercept-url pattern="/admin*" access="hasRole('ROLE_ADMIN')"/>
				<security:intercept-url pattern="/welcome*" access="hasAnyRole('ROLE_USER','ROLE_ADMIN')"/>
				<security:intercept-url pattern="/**" access="hasRole('ROLE_USER')"/>
				<!-- 指定登录页 -->
				<security:form-login login-page="/loginpage" login-processing-url="/login" always-use-default-target="true"
					default-target-url="/welcome" authentication-failure-url="/loginpage?error=error"/>
				<!-- 退出登录 -->
				<security:logout logout-url="/logout" logout-success-url="/loginpage" invalidate-session="true"/>
				<!-- 记住登录信息 -->
				<security:remember-me key="authorition"/>
				<!-- 登录成功后拒绝访问跳转的页面:注意 ie对自定义403的页面有个页面大小限制，必须超过512字节，如果小于512字节，ie会使用其自带的403页面代替 -->
				<security:access-denied-handler error-page="/accessDenied"/>
				<!-- 关闭csrf: 解决HTTP Status 403 - Expected CSRF token not found. Has your session expired? -->
				<security:csrf disabled="true"/>
			</security:http>       
			<!-- 配置认证管理 -->
			<security:authentication-manager>
				<security:authentication-provider>
					<!-- md5方式加密 -->
					<security:password-encoder hash="md5"/>
					<!-- <security:user-service>
						<security:user name="admin" password="21232f297a57a5a743894a0e4a801fc3" authorities="ROLE_USER"/>
					</security:user-service> -->
					<!-- 数据库方式 默认表users,authorities故而修改默认表查询jn_users... -->
					<security:jdbc-user-service data-source-ref="dataSource"
						users-by-username-query = "select username,password,enabled from jn_users where username = ?"  
					authorities-by-username-query = "select username,authority from jn_authorities where username = ?"
					/>
				</security:authentication-provider>
			</security:authentication-manager>
		</beans>  
		
+ Springmvc.xml
		<?xml version="1.0" encoding="UTF-8"?>  
		<beans xmlns="http://www.springframework.org/schema/beans"  
			xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
			xmlns:context="http://www.springframework.org/schema/context"  
			xmlns:mvc="http://www.springframework.org/schema/mvc"  
			xsi:schemaLocation="http://www.springframework.org/schema/beans    
								http://www.springframework.org/schema/beans/spring-beans-4.3.xsd    
								http://www.springframework.org/schema/context    
								http://www.springframework.org/schema/context/spring-context-4.3.xsd    
								http://www.springframework.org/schema/mvc    
								http://www.springframework.org/schema/mvc/spring-mvc-4.3.xsd "> 
			<!-- 自动扫描该包，使SpringMVC认为包下用了@controller注解的类是控制器 -->  
			<mvc:annotation-driven/>
			<context:component-scan base-package="org.monuo.jeenuo.controller" /> 
			
			<!-- 开启 Spring MVC @Controller模式 -->
			<bean id="cnManager" class="org.springframework.web.accept.ContentNegotiationManagerFactoryBean">  
				<property name="ignoreAcceptHeader" value="true"/>  
				<property name="favorPathExtension" value="true"/>  
				<property name="defaultContentType" value="text/html"/>  
				<property name="favorParameter" value="true"/>  
				<property name="mediaTypes">  
					<map>  
						<entry key="xml" value="text/xml"/>  
						<entry key="json" value="application/json"/>  
						<entry key="xls" value="application/vnd.ms-excel"/>  
					</map>  
				</property>  
			</bean>
			 <!-- 转换对象 -->
			<bean id="jaxb2Marshaller" class="org.springframework.oxm.jaxb.Jaxb2Marshaller">
				 <property name="marshallerProperties">  
					  <map>  
						   <entry key="jaxb.formatted.output">  
								<value type="boolean">true</value>  
						   </entry>  
						   <entry key="jaxb.encoding" value="UTF-8" />  
					  </map>  
				 </property>  
				 <property name="packagesToScan">  
					  <list>  
						   <value>org.monuo.jeenuo.entity</value>  
					  </list>  
				 </property>
			</bean>
			
			<!-- 输出为JSON数据 -->
			<bean id="mappingJacksonJsonView" class="org.springframework.web.servlet.view.json.MappingJackson2JsonView">
				<property name="prefixJson" value="false"/>
			</bean>
			
			<!--输出为XML数据 -->
			<bean id="marshallingView" class="org.springframework.web.servlet.view.xml.MarshallingView">
				<constructor-arg ref="jaxb2Marshaller"/>
				<property name="marshaller"> 
					 <bean class="org.springframework.oxm.xstream.XStreamMarshaller"/> 
				 </property> 
			</bean>
			 <!-- 不拦截静态文件 -->
			<mvc:resources location="/" mapping="/*.js"/>
			<mvc:resources location="/" mapping="/*.css"/>
			<mvc:resources location="/static/" mapping="/static/**"/>  
			<mvc:default-servlet-handler/>  
			
			<!-- 定义跳转的文件的前后缀 ，视图模式配置-->  
			<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">  
				<!-- 这里的配置我的理解是自动给后面action的方法return的字符串加上前缀和后缀，变成一个 可用的url地址 -->  
				<property name="prefix" value="/" />  
				<property name="suffix" value=".jsp" />  
			</bean>
			<!-- 配置文件上传，如果没有使用文件上传可以不用配置，当然如果不配，那么配置文件中也不必引入上传组件包 -->  
			<bean id="multipartResolver"    
				class="org.springframework.web.multipart.commons.CommonsMultipartResolver">    
				<!-- 默认编码 -->  
				<property name="defaultEncoding" value="utf-8" />    
				<!-- 文件大小最大值 -->  
				<property name="maxUploadSize" value="10485760000" />    
				<!-- 内存中的最大值 -->  
				<property name="maxInMemorySize" value="40960" />    
			</bean>
		</beans>
	
+ spring-mybatis.xml
		<?xml version="1.0" encoding="UTF-8"?>  
		<beans xmlns="http://www.springframework.org/schema/beans"  
			xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
			xmlns:p="http://www.springframework.org/schema/p"  
			xmlns:context="http://www.springframework.org/schema/context"  
			xsi:schemaLocation="http://www.springframework.org/schema/beans    
								http://www.springframework.org/schema/beans/spring-beans-4.3.xsd    
								http://www.springframework.org/schema/context    
								http://www.springframework.org/schema/context/spring-context-4.3.xsd    
								http://www.springframework.org/schema/tx
								http://www.springframework.org/schema/tx/spring-tx-4.3.xsd
								http://www.springframework.org/schema/aop
								http://www.springframework.org/schema/aop/spring-aop-4.3.xsd
								http://www.springframework.org/schema/util
								http://www.springframework.org/schema/util/spring-util-4.3.xsd">  
			<!-- 引入配置文件 -->  
			<bean id="propertyConfigurer"  
				class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">  
				<property name="location" value="classpath:jdbc.properties" />  
			</bean>  
			<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource"  
				destroy-method="close">  
				<property name="driverClassName" value="${jdbc.driver}" />  
				<property name="url" value="${jdbc.url}" />  
				<property name="username" value="${jdbc.username}" />  
				<property name="password" value="${jdbc.password}" />  
				<!-- 初始化连接大小 -->  
				<property name="initialSize" value="${jdbc.initialSize}"></property>  
				<!-- 连接池最大数量 -->  
				<property name="maxActive" value="${jdbc.maxActive}"></property>  
				<!-- 连接池最大空闲 -->  
				<property name="maxIdle" value="${jdbc.maxIdle}"></property>  
				<!-- 连接池最小空闲 -->  
				<property name="minIdle" value="${jdbc.minIdle}"></property>  
				<!-- 获取连接最大等待时间 -->  
				<property name="maxWait" value="${jdbc.maxWait}"></property>  
			</bean>  
		  
			<!-- spring和MyBatis完美整合，不需要mybatis的配置映射文件 -->  
			<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">  
				<property name="dataSource" ref="dataSource" />  
				<!-- 自动扫描mapping.xml文件 -->  
				<property name="mapperLocations" value="classpath:mapping/*.xml"></property>  
			</bean>  
		  
			<!-- DAO接口所在包名，Spring会自动查找其下的类 -->  
			<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">  
				<property name="basePackage" value="org.monuo.jeenuo.dao" />  
				<property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"></property>  
			</bean>  
		  
			<!-- (事务管理)transaction manager, use JtaTransactionManager for global tx -->  
			<bean id="transactionManager"  
				class="org.springframework.jdbc.datasource.DataSourceTransactionManager">  
				<property name="dataSource" ref="dataSource" />  
			</bean>  
		</beans>  
+ controller
		@RequestMapping(value = "/loginpage")  
		public String loginPage(@RequestParam(value = "error", required = false) String error, Model model) {  
			if (error != null) {
				return "loginerror";
			}
			return "login";  
		}  
		@RequestMapping(value={"/welcome"})
		public String welcome(){
			//获取上下文
			SecurityContext securityContext = SecurityContextHolder.getContext();
			//获取认证对象
			Authentication au = securityContext.getAuthentication();
			//在认证对象中获取主体对象
			Object obj = au.getPrincipal();
			String username = "";
			if(obj instanceof UserDetails)
				username = ((UserDetails) obj).getUsername();
			else
				username = obj.toString();
			System.out.println("\n======================username: " + username + " obj instanceof UserDetails: " +(obj instanceof UserDetails));
			return "index";
		}
		
		@RequestMapping(value={"/accessDenied"})
		public String accessDenied(){
			return "/accessDenied";
		}
		@RequestMapping(value={"/admin"})
		public String admin(){
			return "/admin";
		}
+ login.jsp 
		 <h2>请输入您的用户名与密码</h2>  
		<form name='f' action='${pageContext.request.contextPath }/login' method='POST'>
			<table>
				<tr>
					<td>用户:</td>
					<td><input type='text' name='username' value=''></td>
				</tr>
				<tr>
					<td>密码:</td>
					<td><input type='password' name='password' /></td>
				</tr>
				<tr>
					<td colspan='2'>
						<input id="remember-me" name="remember-me" type="checkbox" checked="checked"/>自动登录
					</td>
				</tr>
				<tr>
					<td colspan='2'><input name="submit" type="submit"
						value="登录" /></td>
				</tr>
			</table>
		</form>
+ index.jsp
		<%@include file="header.jsp" %>
		<!-- sec:authentication property="name" 是springSecurity自定义标签库中获取登录者信息 -->
		<h2>Hello <sec:authentication property="name"/>!这是首页了!</h2>
		<!-- 有选择的显示连接 -->
		<sec:authorize access="hasRole('ROLE_ADMIN')">
			<a href="admin.jsp">进入admin.jsp</a>
		</sec:authorize>

+ loginerror.jsp
		<body>
			登录失败
		</body>
+ header.jsp
		<body>
			<c:url value="/logout" var="logoutUrl"/>
			<a href="${logoutUrl }">退出系统</a>
		</body>
+ admin.jsp
		<%@include file="header.jsp" %>
		<h2>Hello World!这是admin.jsp!</h2>
+ accessDenied.jsp
		<h2>您的访问被拒绝,无权访问该资源!</h2>
		<h2>ie对自定义403的页面有个页面大小限制，必须超过512字节，如果小于512字节，ie会使用其自带的403页面代替</h2>
		<h2>ie对自定义403的页面有个页面大小限制，必须超过512字节，如果小于512字节，ie会使用其自带的403页面代替</h2>
		<h2>ie对自定义403的页面有个页面大小限制，必须超过512字节，如果小于512字节，ie会使用其自带的403页面代替</h2>
		<h2>ie对自定义403的页面有个页面大小限制，必须超过512字节，如果小于512字节，ie会使用其自带的403页面代替</h2>
		<h2>ie对自定义403的页面有个页面大小限制，必须超过512字节，如果小于512字节，ie会使用其自带的403页面代替</h2>
		<h2>ie对自定义403的页面有个页面大小限制，必须超过512字节，如果小于512字节，ie会使用其自带的403页面代替</h2>
		<h2>ie对自定义403的页面有个页面大小限制，必须超过512字节，如果小于512字节，ie会使用其自带的403页面代替</h2>
		<h2>ie对自定义403的页面有个页面大小限制，必须超过512字节，如果小于512字节，ie会使用其自带的403页面代替</h2>

参考文章: 
http://download.csdn.net/detail/u012367513/7826801
http://blog.csdn.net/yin380697242/article/details/51786388		