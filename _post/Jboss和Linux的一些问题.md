---
title: Jboss和Linux的一些问题
date: 2017-01-10 1:12:08 
tags: Linux
category: Linux
---
### 1 Jboss IP访问问题
+ ./standalone.sh 之后只能用127.0.0.1或者localhost访问
    + 方法一: ./standalone.sh -b 0.0.0.0
    + 方法二:  如下
            <interface name="public">
                <!--<inet-address value="${jboss.bind.address:127.0.0.1}"/>-->
                <inet-address value="0.0.0.0"/>
            </interface>

### 2 Linux设置端口

    [root@localhost etc]# cat /etc/sysconfig/iptables
        # Firewall configuration written by system-config-firewall
        # Manual customization of this file is not recommended.
        *filter
        :INPUT ACCEPT [0:0]
        :FORWARD ACCEPT [0:0]
        :OUTPUT ACCEPT [0:0]
        -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
        -A INPUT -p icmp -j ACCEPT
        -A INPUT -i lo -j ACCEPT
        -A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -j ACCEPT
        -A INPUT -j REJECT --reject-with icmp-host-prohibited
        -A FORWARD -j REJECT --reject-with icmp-host-prohibited
        -A INPUT -m state --state NEW -m tcp -p tcp --dport 8080 -j ACCEPT
        COMMIT


### 3 本机无法访问虚拟机启动的jboss(wilk)
+ 虚拟机设置成了桥接模式或者仅主机模式
+ 关闭了enforce 临时关闭命令setenforce 0 
+ 关闭iptables 临时关闭命令 service iptables stop
        开启： chkconfig iptables on  
 
        关闭： chkconfig iptables off
