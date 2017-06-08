---
title: Hadoop 分布式安装
date: 2017-05-17 13:12:08 
tags: Hadoop
category: Hadoop
---
### 1 添加用户 
    + sudo adduser hadoop
    
##### 1.1 添加用户成为sudoers
    > chmod u+w /etc/sudoers
    > 编辑/etc/sudoers文件："root ALL=(ALL) ALL"在下面添加"hadoop ALL=(ALL) ALL"(这里的zhc是你的用户名)，然后保存。
    > 最后恢复没有写权限模式，撤销文件的写权限，chmod u-w /etc/sudoers
    
### 2 修改主机名 master
    sudo vim /etc/hostname

### 3 修改 /etc/hosts
    sudo vim /etc/hosts
    如下:   
        127.0.0.1       localhost
        127.0.1.1       localhost.localdomain   localhost

        
        # The following lines are desirable for IPv6 capable hosts
        ::1     ip6-localhost ip6-loopback
        fe00::0 ip6-localnet
        ff00::0 ip6-mcastprefix
        ff02::1 ip6-allnodes
        ff02::2 ip6-allrouters
        
        # hadoop nodes
        192.168.159.132 master
        192.168.159.134 slave1
        192.168.159.135 slave2
### 4 安装jdk
    hadoop@master:~$ java -version
    java version "1.8.0_111"
    Java(TM) SE Runtime Environment (build 1.8.0_111-b14)
    Java HotSpot(TM) 64-Bit Server VM (build 25.111-b14, mixed mode)
### 5 安装 openssh-server
    sudo apt-get install openssh-server
    sudo /etc/init.d/ssh start
    如下: 
    hadoop@master:~$ ssh localhost
    The authenticity of host 'localhost (127.0.0.1)' can't be established.
    ECDSA key fingerprint is SHA256:SgoGKGREm5ugeBD2QHLd9rSiETfU6JLHo7iGJeCpDTg.
    Are you sure you want to continue connecting (yes/no)? yes
    Warning: Permanently added 'localhost' (ECDSA) to the list of known hosts.
    hadoop@localhost's password:
    Welcome to Ubuntu 16.04 LTS (GNU/Linux 4.4.0-21-generic x86_64)
    
     * Documentation:  https://help.ubuntu.com/
    
    455 packages can be updated.
    191 updates are security updates.
    
    Last login: Sat May 13 14:30:02 2017 from 192.168.159.1
    hadoop@master:~$ ps -e | grep ssh
      832 ?        00:00:00 sshd
     1171 ?        00:00:00 sshd
     1227 ?        00:00:00 sshd
     1308 pts/0    00:00:00 ssh
     1309 ?        00:00:00 sshd
     1338 ?        00:00:00 sshd
     
### 6 配置slave1, slave2
    通过虚拟机克隆,修改slave1,slave2 /etc/hostname
 
### 7 配置无密码登录ssh
    ssh-keygen -t rsa
    cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
    chmod 600 ~/.ssh/authorized_keys
    
    ssh localhost
    将生成的公钥发给slave1,slave2
    scp .ssh/authorized_keys hadoop@slave1:~/.ssh
    scp .ssh/authorized_keys hadoop@slave2:~/.ssh
    测试连接是否成功:
    hadoop@master:~$ ssh slave1
        Welcome to Ubuntu 16.04 LTS (GNU/Linux 4.4.0-21-generic x86_64)
        
         * Documentation:  https://help.ubuntu.com/
        
        455 packages can be updated.
        191 updates are security updates.
        
        Last login: Sat May 13 23:50:34 2017 from 192.168.159.1
    
    hadoop@master:~$ ssh slave2
        Welcome to Ubuntu 16.04 LTS (GNU/Linux 4.4.0-21-generic x86_64)
        
         * Documentation:  https://help.ubuntu.com/
        
        455 packages can be updated.
        191 updates are security updates.
        
        Last login: Sat May 13 22:16:44 2017 from 192.168.159.1

       
### 8 解压 hadoop2.7.3
    tar -zxvf hadoop-2.7.3.tar.gz
    
### 9 配置hadoop
+ 1 配置hadoop(slave1,slave2,master)环境变量: sudo vim /etc/profile 
        
        JAVA_HOME=/usr/local/jdk1.8.0_111
        JRE_HOME=$JAVA_HOME/jre
        JAVA_BIN=$JAVA_HOME/bin
        CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
        PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
        # set hadoop classpath
        export HADOOP_HOME=/home/hadoop/hadoop-2.7.3
        export HADOOP_MAPRED_HOME=$HADOOP_HOME
        export HADOOP_COMMON_HOME=$HADOOP_HOME
        export HADOOP_HDFS_HOME=$HADOOP_HOME
        export YARN_HOME=$HADOOP_HOME
        export JAVA_LIBRARY_PATH=$HADOOP_HOME/lib/native
        export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
        export YARN_CONF_DIR=$HADOOP_HOME/etc/hadoop
        export HADOOP_PREFIX=$HADOOP_HOME
        export CLASSPATH=$CLASSPATH:.:$HADOOP_HOME/bin
        
        export JAVA_HOME JRE_HOME PATH CLASSPATH

+ 2 修改配置文件$HADOOP_HOME/etc/hadoop/,将.template文件的这个后缀去掉
    - vim core-size.xml
            <configuration>
                <property>
                    <name>fs.defaultFS</name>
                    <!-- master: /etc/hosts 配置的域名 master -->
                    <value>hdfs://master:9000/</value>
                </property>
            </configuration>
    - vim hdfs-site.xml
            <configuration>
                <property>
                    <name>dfs.namenode.name.dir</name>
                    <value>/home/hadoop/hadoop-2.7.3/dfs/namenode</value>
                </property>
                <property>
                    <name>dfs.datanode.data.dir</name>
                    <value>/home/hadoop/hadoop-2.7.3/dfs/datanode</value>
                </property>
                <property>
                    <name>dfs.replication</name>
                    <value>1</value>
                </property>
                <property>
                    <name>dfs.namenode.secondary.http-address</name>
                    <value>master:9001</value>
                </property>
            </configuration>
    - vim mapred-site.xml
            <configuration>
                <property>
                    <name>mapreduce.framework.name</name>
                    <value>yarn</value>
                </property>
                <property>
                    <name>mapreduce.jobhistory.address</name>
                    <value>master:10020</value>
                </property>
                <property>
                    <name>mapreduce.jobhistory.webapp.address</name>
                    <value>master:19888</value>
                </property>
            </configuration>
    - vim yarn-site.xml
            <configuration>
                <property>
                    <name>yarn.nodemanager.aux-services</name>
                    <value>mapreduce_shuffle</value>
                </property>
                <property>                                                             
                    <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
                    <value>org.apache.hadoop.mapred.ShuffleHandler</value>
                </property>
                <property>
                    <name>yarn.resourcemanager.address</name>
                    <value>master:8032</value>
                </property>
                <property>
                    <name>yarn.resourcemanager.scheduler.address</name>
                    <value>master:8030</value>
                </property>
                <property>
                    <name>yarn.resourcemanager.resource-tracker.address</name>
                    <value>master:8031</value>
                </property>
                <property>
                    <name>yarn.resourcemanager.admin.address</name>
                    <value>master:8033</value>
                </property>
                <property>
                    <name>yarn.resourcemanager.webapp.address</name>
                    <value>master:8088</value>
                </property>
            </configuration>
+ 修改env环境变量文件: hadoop-env.sh、mapred-env.sh、yarn-env.sh 文件添加 JAVA_HOME：
        export JAVA_HOME=/usr/local/jdk1.8.0_111/
+ 修改 slaves文件
        vim slaves
            slave1
            slave2
+ 将配置scp到salve1,slave2
        scp -r hadoop-2.7.0/ slave1:~/software
        scp -r hadoop-2.7.0/ slave2:~/software
### 10 format hadoop 在master节点
        hadoop@master:~/hadoop-2.7.3$ ./bin/hdfs namenoade -format
        
        17/05/13 23:53:59 INFO namenode.NameNode: STARTUP_MSG:
        /************************************************************
        STARTUP_MSG: Starting NameNode
        STARTUP_MSG:   host = master/192.168.159.132
        STARTUP_MSG:   args = [-format]
        STARTUP_MSG:   version = 2.7.3
        STARTUP_MSG:   classpath = /home/scheduler/*.jar...
        STARTUP_MSG:   build = https://git-wip-us.apache.org/repos/asf/hado...
        STARTUP_MSG:   java = 1.8.0_111
        ************************************************************/
        ...
        /************************************************************
        SHUTDOWN_MSG: Shutting down NameNode at master/192.168.159.132
        ************************************************************/
        
        ☆ 如果不是不是第一次,需要将dfs,logs文件目录先rm掉

### 11 启动或者关闭Hadoop
+ 启动
        hadoop@master:~/hadoop-2.7.3$ ./sbin/start-all.sh
        This script is Deprecated. Instead use start-dfs.sh and start-yarn.sh
        Starting namenodes on [master]
        master: starting namenode, logging to ~/logs/hadoop-hadoop-namenode-master.out
        slave2: starting datanode, logging to ~/logs/hadoop-hadoop-datanode-slave2.out
        slave1: starting datanode, logging to ~/logs/hadoop-hadoop-datanode-slave1.out
        Starting secondary namenodes [master]
        master: starting secondarynamenode, logging to ~/logs/hadoop-hadoop-secondarynamenode-master.out
        starting yarn daemons
        starting resourcemanager, logging to ~/logs/yarn-hadoop-resourcemanager-master.out
        slave2: starting nodemanager, logging to ~/logs/yarn-hadoop-nodemanager-slave2.out
        slave1: starting nodemanager, logging to ~/logs/yarn-hadoop-nodemanager-slave1.out
        hadoop@master:~/hadoop-2.7.3$ jps
        10161 Jps
        9746 SecondaryNameNode
        9891 ResourceManager
        9542 NameNode
+ 关闭
        hadoop@master:~/hadoop-2.7.3$ ./sbin/start-all.sh

+ 验证 
        hadoop@master:~/hadoop-2.7.3$ jps
        10161 Jps
        9746 SecondaryNameNode
        9891 ResourceManager
        9542 NameNode
    
        hadoop@slave2:~/hadoop-2.7.3$ jps
        4512 NodeManager
        4645 Jps
        4391 DataNode

        hadoop@slave1:~/hadoop-2.7.3$ jps
        6064 Jps
        5813 DataNode
        5935 NodeManager

+ 浏览器访问:
        http://192.168.159.132:50070/ --master
        http://192.168.159.132:8088/ --master
        http://192.168.159.134:50075/  --slave1
        http://192.168.159.135:50075/  -- slave1

主要参考文章: http://www.linuxdiyf.com/linux/27090.html



