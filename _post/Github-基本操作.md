---
title: Github 基本操作
date: 2016-09-12 17:12:08 
tags: Github
category: Github
---
### 1 安装git

### 2 配置全局用户
+ git config --global user.email "you@example.com"
+ git config --global user.name "Your Name"

### 2 配置ssh key

+ 执行: ssh-keygen -C "mnuo1115@163.com"
+ 将 C:\Users\jh\.ssh\id_rsa.pub 文件中的内容添加到远程库

### 3 建立连接

+ 1 mkdir mnuo.github.io
+ 2 cd mnuo.github.io
+ 3 git init
+ 4 建立连接: git remote add origin git@github.com:mnuo/mnuo.github.io.git
+ 5 下载代码到本地
	- 下载master分支: git pull origin master:master
	- 下载mdfile分支: git fetch origin mdfile:mdfile
+ 6 分支切换:  git checkout mdfile

### 4 上传文件

+ 1 git add <file>...
+ 2 git commit -m '注释'
+ 3 git push origin master

