---
title: Hadoop 调用HDFS
date: 2017-05-18 09:12:08 
tags: Hadoop
category: Hadoop
---
### 1 什么是HDFS
    HDFS全称是: Hadoop 分布式文件系统(Hadoop Distributed File System), 是Hadoop的核心之一。要实现MapReduce算法,数据必须上传到HDFS上。HDFS提供了一套类似Linux的命令。
    
### 2 查看HDFS的命令
        hadoop@master:~/hadoop-2.7.3$ bin/hadoop fs
        Usage: hadoop fs [generic options]
                [-appendToFile <localsrc> ... <dst>]
                [-cat [-ignoreCrc] <src> ...]
                [-checksum <src> ...]
                [-chgrp [-R] GROUP PATH...]
                [-chmod [-R] <MODE[,MODE]... | OCTALMODE> PATH...]
                [-chown [-R] [OWNER][:[GROUP]] PATH...]
                [-copyFromLocal [-f] [-p] [-l] <localsrc> ... <dst>]
                [-copyToLocal [-p] [-ignoreCrc] [-crc] <src> ... <localdst>]
                [-count [-q] [-h] <path> ...]
                [-cp [-f] [-p | -p[topax]] <src> ... <dst>]
                [-createSnapshot <snapshotDir> [<snapshotName>]]
                [-deleteSnapshot <snapshotDir> <snapshotName>]
                [-df [-h] [<path> ...]]
                [-du [-s] [-h] <path> ...]
                [-expunge]
                [-find <path> ... <expression> ...]
                [-get [-p] [-ignoreCrc] [-crc] <src> ... <localdst>]
                [-getfacl [-R] <path>]
                [-getfattr [-R] {-n name | -d} [-e en] <path>]
                [-getmerge [-nl] <src> <localdst>]
                [-help [cmd ...]]
                [-ls [-d] [-h] [-R] [<path> ...]]
                [-mkdir [-p] <path> ...]
                [-moveFromLocal <localsrc> ... <dst>]
                [-moveToLocal <src> <localdst>]
                [-mv <src> ... <dst>]
                [-put [-f] [-p] [-l] <localsrc> ... <dst>]
                [-renameSnapshot <snapshotDir> <oldName> <newName>]
                [-rm [-f] [-r|-R] [-skipTrash] <src> ...]
                [-rmdir [--ignore-fail-on-non-empty] <dir> ...]
                [-setfacl [-R] [{-b|-k} {-m|-x <acl_spec>} <path>]|[--set <acl_spec> <path>]]
                [-setfattr {-n name [-v value] | -x name} <path>]
                [-setrep [-R] [-w] <rep> <path> ...]
                [-stat [format] <path> ...]
                [-tail [-f] <file>]
                [-test -[defsz] <path>]
                [-text [-ignoreCrc] <src> ...]
                [-touchz <path> ...]
                [-truncate [-w] <length> <path> ...]
                [-usage [cmd ...]]
### 3 代码实
+ 1 mkdir
        hadoop@master:~/hadoop-2.7.3$ bin/hadoop fs -mkdir -p /tmp/new/
        hadoop@master:~/hadoop-2.7.3$ bin/hadoop fs -ls /tmp
        Found 2 items
        drwx------   - hadoop supergroup          0 2017-05-14 04:58 /tmp/hadoop-yarn
        drwxr-xr-x   - hadoop supergroup          0 2017-05-14 06:49 /tmp/new
        hadoop@master:~/hadoop-2.7.3$ bin/hadoop fs -mkdir -p /tmp/new1/one
        hadoop@master:~/hadoop-2.7.3$ bin/hadoop fs -ls /tmp
        Found 3 items
        drwx------   - hadoop supergroup          0 2017-05-14 04:58 /tmp/hadoop-yarn
        drwxr-xr-x   - hadoop supergroup          0 2017-05-14 06:49 /tmp/new
        drwxr-xr-x   - hadoop supergroup          0 2017-05-14 06:53 /tmp/new1
        hadoop@master:~/hadoop-2.7.3$ bin/hadoop fs -ls /tmp/new1
        Found 1 items
        drwxr-xr-x   - hadoop supergroup          0 2017-05-14 06:53 /tmp/new1/one

+ 2 rmr 
        hadoop@master:~/hadoop-2.7.3$ bin/hadoop fs -rmr /tmp/new/one
        rmr: DEPRECATED: Please use 'rm -r' instead.
        17/05/14 08:03:30 INFO fs.TrashPolicyDefault: Namenode trash configuration: Deletion interval = 0 minutes, Emptier interval = 0 minutes.
        Deleted /tmp/new/one
        hadoop@master:~/hadoop-2.7.3$ bin/hadoop fs -ls /tmp/new
        Found 1 items
        drwxr-xr-x   - hadoop supergroup          0 2017-05-14 07:50 /tmp/new/two
    
+ 3 copyFromLocal
        hadoop@master:~/hadoop-2.7.3$ bin/hadoop fs -copyFromLocal /home/hadoop/hadoop-2.7.3/etc/hadoop/core-site.xml  /tmp/new/
        hadoop@master:~/hadoop-2.7.3$ bin/hadoop fs -ls /tmp/new
        Found 1 items
        -rw-r--r--   1 hadoop supergroup       1019 2017-05-14 08:08 /tmp/new/core-site.xml

+ 4 cat 
        hadoop@master:~/hadoop-2.7.3$ bin/hadoop fs -cat /tmp/new/test.xml
        <RBSPMessage>
            <Version/>
            ......

+ 5 copyToLocal
        hadoop@master:~/hadoop-2.7.3$ bin/hadoop fs -copyToLocal /tmp/new/test.xml /home/hadoop/datafiles/
+ 6 touchz
        hadoop@master:~/hadoop-2.7.3$ bin/hadoop fs -touchz /tmp/new/empty
        hadoop@master:~/hadoop-2.7.3$ bin/hadoop fs -ls /tmp/new
        Found 3 items
        -rw-r--r--   1 hadoop supergroup       1019 2017-05-14 08:08 /tmp/new/core-site.xml
        -rw-r--r--   1 hadoop supergroup          0 2017-05-14 09:07 /tmp/new/empty
        -rw-r--r--   3 hadoop supergroup       1864 2017-05-14 08:15 /tmp/new/test.xml
        hadoop@master:~/hadoop-2.7.3$ bin/hadoop fs -cat /tmp/new/empty

以上内容，并非我的原创，主要参考来源地址：http://blog.fens.me/hadoop-hdfs-api/

代码: 
            /**
             * HdfsDao.java created at 2017年5月18日 上午10:15:08
             */
            package org.monuo.myhadoop.dao;
            
            import java.io.ByteArrayOutputStream;
            import java.io.IOException;
            import java.io.OutputStream;
            import java.net.URI;
            
            import org.apache.hadoop.conf.Configuration;
            import org.apache.hadoop.fs.BlockLocation;
            import org.apache.hadoop.fs.FSDataInputStream;
            import org.apache.hadoop.fs.FSDataOutputStream;
            import org.apache.hadoop.fs.FileStatus;
            import org.apache.hadoop.fs.FileSystem;
            import org.apache.hadoop.fs.Path;
            import org.apache.hadoop.io.IOUtils;
            import org.apache.hadoop.mapred.JobConf;
            import org.apache.log4j.Logger;
            import org.monuo.myhadoop.common.util.ResourceConsts;
            
            /**
             * @author saxon
             */
            public class HdfsDao {
                public static Logger logger = Logger.getLogger(HdfsDao.class);
            	public static String HDFS = "";
            	static {
            		try {
            			HDFS = ResourceConsts.getValue("hadoop_config", "HDFS_URI");
            			System.setProperty("hadoop.home.dir", "D:/appservers/hadoop-2.7.3");//可以在本地设置HADOOP_HOME环境变量
            		} catch (Exception e) {
            			e.printStackTrace();
            		}
            	}
            
                //hdfs路径
                private String hdfsPath;
                //Hadoop系统配置
                private Configuration conf;
            	
            	public HdfsDao(Configuration conf) {
                    this(HDFS, conf);
                }
                public HdfsDao(String hdfs, Configuration conf) {
                    this.hdfsPath = hdfs;
                    this.conf = conf;
                }
                
                public static JobConf config() {
                    JobConf conf = new JobConf();
                    conf.setJobName("HdfsDao");
            //      conf.addResource("classpath:/hadoop/core-site.xml");
            //      conf.addResource("classpath:/hadoop/hdfs-site.xml");
            //      conf.addResource("classpath:/hadoop/mapred-site.xml");
                    return conf;
                }
            	/**
            	 * @param args
            	 * @throws IOException 
            	 */
            	public static void main(String[] args) throws IOException {
            		JobConf conf = config();
                    HdfsDao hdfs = new HdfsDao(conf);
            //        hdfs.mkdirs("/tmp/new/two");
            //        hdfs.rmr("/tmp/new/two");
            //        hdfs.copyFile("d:/mnuo/test.xml", "/tmp/new");
            //        hdfs.cat("/tmp/new/test.xml");
            //        hdfs.download("/tmp/new/core-site.xml", "d:/");
            //        hdfs.createFile("/tmp/new/text", "Hello World!");
            //        hdfs.cat("/tmp/new/text");
            //        hdfs.renameFile("/tmp/new/text", "/tmp/new/text1");
                    hdfs.location("/tmp/new/", "text1");
                    hdfs.ls("/tmp/new");
            	}
            	/**
            	 * 遍历文件
            	 * @param folder
            	 * @throws IOException 
            	 */
            	public void ls(String folder) throws IOException{
            		Path path = new Path(folder);
            		FileSystem fs = FileSystem.get(URI.create(hdfsPath), conf);
            		FileStatus[] list = fs.listStatus(path);
            		logger.info("ls: " + folder);
            		logger.info("\n=========================");
            		for(FileStatus f : list){
            			logger.info("\n name: "+f.getPath()+", folder: "+f.isDirectory()+", size: "+f.getLen()+"\n");
            		}
            		logger.info("\n=========================");
            		fs.close();
            		
            	}
            	/**
            	 * 创建目录
            	 * @param folder
            	 * @throws IOException 
            	 */
            	public void mkdirs(String folder) throws IOException{
            		Path path = new Path(folder);
            		FileSystem fs = FileSystem.get(URI.create(hdfsPath), conf);
            		if(!fs.exists(path)){
            			fs.mkdirs(path);
            			logger.info(" create folder: " + folder);
            		}
            		fs.close();
            	}
            	/**
            	 * 删除目录或文件
            	 * @param folder
            	 * @throws IOException 
            	 */
            	public void rmr(String folder) throws IOException{
            		Path path = new Path(folder);
            		FileSystem fs = FileSystem.get(URI.create(hdfsPath), conf);
            		fs.deleteOnExit(path);
            		logger.info("Delete: " + folder);
            		fs.close();
            	}
            	/**
            	 * 复制文件到远程HDFS
            	 * @param local
            	 * @param remote
            	 * @throws IOException
            	 */
            	public void copyFile(String local, String remote) throws IOException{
            		FileSystem fs = FileSystem.get(URI.create(hdfsPath), conf);
            		fs.copyFromLocalFile(new Path(local), new Path(remote));
            		logger.info("copy from : " + local + " to : " + remote);
            		fs.close();
            	}
            	/**
            	 * 查看文件内容
            	 * @param remoteFile
            	 * @throws IOException
            	 */
            	public String cat(String remoteFile) throws IOException{
            		Path path = new Path(remoteFile);
            		FileSystem fs = FileSystem.get(URI.create(hdfsPath), conf);
            		FSDataInputStream fsdis = null;
            		logger.info("\ncat: " + remoteFile);
            		OutputStream outputStream = new ByteArrayOutputStream();
            		String string = null;
            		try {
            			fsdis = fs.open(path);
            			IOUtils.copyBytes(fsdis, outputStream, 4096, false);
            			string = outputStream.toString();
            		} finally {
            			IOUtils.closeStream(fsdis);
            			fs.close();
            		}
            		logger.info(string);
            		return string;
            	}
            	/**
            	 * 从远程下载文件
            	 * @param remote
            	 * @param local
            	 * @throws IOException
            	 */
            	public void download(String remote, String local) throws IOException{
            		Path path = new Path(remote);
            		FileSystem fs = FileSystem.get(URI.create(hdfsPath), conf);
            		fs.copyToLocalFile(path, new Path(local));
            		logger.info("\n download: from " + remote + " to " + local);
            		fs.close();
            	}
            	/**
            	 * 创建新文件
            	 * @param file
            	 * @param content
            	 * @throws IOException
            	 */
            	public void createFile(String file, String content) throws IOException{
            		FileSystem fs = FileSystem.get(URI.create(hdfsPath), conf);
            		byte[] buff = content.getBytes();
            		FSDataOutputStream fsos = null;
            		try {
            			fsos = fs.create(new Path(file));
            			fsos.write(buff, 0, buff.length);
            		} finally {
            			if(fsos != null) fsos.close();
            			fs.close();
            		}
            		logger.info("\n create file: " + file);
            	}
            	/**
            	 * 重命名文件
            	 * @param src
            	 * @param dst
            	 * @throws IOException
            	 */
            	public void renameFile(String src, String dst) throws IOException{
            		FileSystem fs = FileSystem.get(URI.create(hdfsPath), conf);
            		fs.rename(new Path(src), new Path(dst));
            		logger.info("rename file: from "+ src + " to " + dst);
            		fs.close();
            	}
            	/**
            	 * 返回文件位置
            	 * @param folder
            	 * @param file
            	 * @throws IOException
            	 */
            	public void location(String folder, String file) throws IOException{
            		FileSystem fs = FileSystem.get(URI.create(hdfsPath), conf);
            		FileStatus f = fs.getFileStatus(new Path(hdfsPath + folder + file));
            		BlockLocation[] list = fs.getFileBlockLocations(f, 0, f.getLen());
            		logger.info("file location: " + hdfsPath + folder + file);
            		for (BlockLocation bl : list) {
            		    String[] hosts = bl.getHosts();
            		    for (String host : hosts) {
            		    	logger.info("host:" + host);
            		    }
            		}
            		fs.close();
            		
            	}
            }
