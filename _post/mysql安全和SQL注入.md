---
title: mysql安全和SQL注入
date: 2016-07-12 13:35:08 
tags: MySql
category: MySQL
---
####  SQL注入简介:

> 产生:  
主要是由于程序对用户输入的数据没有进行严格的过滤,导致非法数据库查询语句的执行 SQL注入(SQL injection) 攻击具有很大的危害,攻击者可以利用它读取,修改或者删除数据库内的数据,获取数据库中的用户名和密码等敏感信息,甚至可以获得数据库管理员的权限,而且SQL Injection很难防护  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;应用开发中可以才去的应对措施: 

+ 1,PrepareStatement+Bind-variables   
在开发的时候强烈建议使用PrepareStatement+Bind-variable来实现,尽量不要使用拼接的SQL:

    	String sql = "select * from users u where u.id=? and u.password=? ";
    
    	PrepareStatement ps = conn.prepareSatatement(sql);
    
    	ps.setInt(1,id);
    
    	ps.setString(2,password);

+ 2,使用应用程序提供的转换函数

+ 3,自定义函数进行校验   
途径:
    - 整理数据使之变得有效
    - 拒绝已知的非法输入
    - 只接受已知的合法输入

 所以如果想要获得最好的安全状态,目前最好的解决方法就是对用户提交或者可能改变的数据进行简单分类,分别应用正则表达式来对用户提供的输入数据进行严格的检测和验证。
    > 已知符号:

	    “’”、“;”、“=”、“(”、“)”、“/*”、“*/”、“%”、“+”、“”、“>”、“<”、“--”、“[”、“]”；

    > 正则表达式:

    	(|\'|(\%27)|\;|(\%3b)|\=|(\%3d)|\(|(\%28)|\)|(\%29)|(\/*)|(\%2f%2a)|(\*/)|(\%2a%2f)|\+| (\%2b)|\<|(\%3c)|\>|(\%3e)|\(--))|\[|\%5b|\]|\%5d)

			