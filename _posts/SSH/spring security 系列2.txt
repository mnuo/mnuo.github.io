---
title: spring security 系列2
date: 2017-06-25 23:53:08 
tags: SSH
category: SSH
---
Spring security的简单原理:
1用户登录,会被AuthenticationProcessingFilter拦截,调用AuthenticationManager的实现,而且AuthenticationManager会调用ProviderManager来获取用户验证信息(不同的provider调用的服务不同,因为这些信息可以是在数据库上,可以是在LDAP服务器上,可以是xml配置文件上等),如果验证通过会将用户的权限信息封装一个User放到Spring的全局缓存SecurityContextHolder中,以备后面资源访问使用。
2访问资源(即授权管理),访问url时,会通过AbstractSecurityIntercepter拦截器拦截,其中会调用FilterInvocationSecurityMetadataSource的方法来获取被拦截url所需的全部权限,再调用授权管理器AccessDecistionManager,这个授权管理器会通过spring的全局缓存SecurityContextHolder获取用户的权限信息,还会获取被拦截url所需的全部权限,然后根据所配的策略,如果权限足够,则返回,否则报错并调用权限不足的页面。

参考文章:
http://blog.csdn.net/u012367513/article/details/38866465

