---
title: Maven 私服(Nexus)
date: 2017-02-27 1:12:08 
tags: Maven
category: Mave
---
### 1 私服是什么
+ 私服，私有服务器，是公司内部Maven项目经常需要的东东
+ 常用功能: 
    - Nexus常用功能就是：指定私服的中央地址、将自己的Maven项目指定到私服地址、从私服下载中央库的项目索引、从私服仓库下载依赖组件、将第三方项目jar上传到私服供其他项目组使用。
    - 一般用到的仓库种类是hosted、proxy。Hosted代表宿主仓库，用来发布一些第三方不允许的组件，比如Oracle驱动、比如商业软件jar包。Proxy代表代理远程的仓库，最典型的就是Maven官方中央仓库、JBoss仓库等等。如果构建的Maven项目本地仓库没有依赖包，那么就会去这个代理站点去下载，那么如果代理站点也没有此依赖包，就回去远程中央仓库下载依赖，这些中央仓库就是proxy。代理站点下载成功后再下载至本机。
        - hosted   类型的仓库，内部项目的发布仓库
        - releases 内部的模块中release模块的发布仓库
        - snapshots 发布内部的SNAPSHOT模块的仓库
        - 3rd party 第三方依赖的仓库，这个数据通常是由内部人员自行下载之后发布上去
        - proxy   类型的仓库，从远程中央仓库中寻找数据的仓库
        - group   类型的仓库，组仓库用来方便我们开发人员进行设置的仓库

### 2 参考文章
    maven--私服的搭建（Nexus的使用）
[链接] [1] 

  [1]: http://blog.csdn.net/shenshen123jun/article/details/9084293        "链接"
 