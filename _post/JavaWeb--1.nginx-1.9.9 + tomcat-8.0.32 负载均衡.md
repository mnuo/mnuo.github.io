---
title: nginx-1.9.9 + tomcat-8.0.32 负载均衡
date: 2016-07-19 14:54:22
tags: nginx
category: Java Web
---
#### 1 tomcat 安装
下载并解压两份tomcat, 修改其中一个tomcat的端口   

+ Server port="8005" shutdown="SHUTDOWN" 这是一个关闭监听端口 修改成8006
+ Connector port="8080" 指定服务器端要创建的端口号,并在这个端口监听来自客户端的请求,修改成8081
+ Connector port="8009" 修改成8010

#### 2 nginx安装
Centos6.5下安装 nginx-1.9.9

+ 下载: wget http://nginx.org/download/nginx-1.9.9.tar.gz
+ 解压: tar -zxvf nginx-1.9.9.tar.gz
+ 配置:  ./configure --prefix=/usr/local/nginx --without-http_rewrite_module --without-http_gzip_module
+ 安装到/usr/local/nginx make && mark install

#### 3 配置nginx
    # 指定用户,这里不区分用户
    user  nobody; 
    worker_processes  1;
    # 日志路径
    error_log  logs/error.log;
    error_log  logs/error.log  notice;
    error_log  logs/error.log  info;
    
    #pid        logs/nginx.pid;
    
    
    events {
        #连接数
        worker_connections  1024;
    }
    
    
    http {
        include       mime.types;
        default_type  application/octet-stream;
    
        #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
        #                  '$status $body_bytes_sent "$http_referer" '
        #                  '"$http_user_agent" "$http_x_forwarded_for"';
    
        #access_log  logs/access.log  main;
    
        sendfile        on;
        #tcp_nopush     on;
    
        #keepalive_timeout  0;
        keepalive_timeout  65;
    
        #gzip  on;
        
    	upstream mnuo {
    		#配置两台tomcat负载均衡,ip:port weight是权重
    		server 120.76.131.4:8080 weight=10;  
            server 120.76.131.4:8081 weight=10; 
    	}
    	
        server {
            listen       80;
            server_name  localhost;
    
            #charset koi8-r;
    
            #access_log  logs/host.access.log  main;
    		  
            location / {
                #root   html;  
                #index  index.jsp index.htm;  
    			#lhy是 upstream 后面的名字 lhy  
                proxy_pass http://mnuo;  
                #localhost是nginx服务器的主机地址，如果不写此句，会导致静态文件访问路径为http://lhy，导致找不到地址  
                proxy_set_header Host localhost;    
                #forwarded信息，用于告诉后端服务器终端用户的ip地址，否则后端服务器只能获取前端代理服务器的ip地址。  
                proxy_set_header Forwarded $remote_addr; 
            }
    
            #error_page  404              /404.html;
    
            # redirect server error pages to the static page /50x.html
            #
            error_page   500 502 503 504  /50x.html;
            location = /50x.html {
                root   html;
            }
    
            # proxy the PHP scripts to Apache listening on 127.0.0.1:80
            #
            #location ~ \.php$ {
            #    proxy_pass   http://127.0.0.1;
            #}
    
            # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
            #
            #location ~ \.php$ {
            #    root           html;
            #    fastcgi_pass   127.0.0.1:9000;
            #    fastcgi_index  index.php;
            #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
            #    include        fastcgi_params;
            #}
    
            # deny access to .htaccess files, if Apache's document root
            # concurs with nginx's one
            #
            #location ~ /\.ht {
            #    deny  all;
            #}
        }
    
    
        # another virtual host using mix of IP-, name-, and port-based configuration
        #
        #server {
        #    listen       8000;
        #    listen       somename:8080;
        #    server_name  somename  alias  another.alias;
    
        #    location / {
        #        root   html;
        #        index  index.html index.htm;
        #    }
        #}
    
    
        # HTTPS server
        #
        #server {
        #    listen       443 ssl;
        #    server_name  localhost;
    
        #    ssl_certificate      cert.pem;
        #    ssl_certificate_key  cert.key;
    
        #    ssl_session_cache    shared:SSL:1m;
        #    ssl_session_timeout  5m;
    
        #    ssl_ciphers  HIGH:!aNULL:!MD5;
        #    ssl_prefer_server_ciphers  on;
    
        #    location / {
        #        root   html;
        #        index  index.html index.htm;
        #    }
        #}
    
    }
