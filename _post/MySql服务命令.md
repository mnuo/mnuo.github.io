---
title: MySql服务命令
date: 2016-06-09 15:20:08
tags: 
category: MySQL
---
#### MySql服务命令

- 查看mysql的运行状态:
	+ service mysqld status
	+ /etc/init.d/mysqld status
- 启动服务:
	+ service mysqld start
- 关闭服务
	+ service mysqld stop
- 连接mysql
	+ mysql -h 127.0.0.1 -P 3306 -uroot -p123123
