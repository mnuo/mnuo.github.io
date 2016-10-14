---
title: java进阶--12文件
date: 2016-07-12 17:12:08 
tags: java
category: Java
---
#### file 

操作file流:

+ inputStream和OutputStream都是字节流
+ Reader和Writer是字符流，
> 磁盘上存储的文件是字节流，是在读取过程中将字节流转换成字符流
            /**
             * @author mnuo
             * <p>
             *     这个列子，主要实现了文件的拷贝，这个列子我们主要用到了InputStream和OutputStream的实现<br>
             * 	inputStream和OutputStream都是字节流<br>
             * 	Reader和Writer是字符流，磁盘上存储的文件是字节流，是在读取过程中将字节流转换成字符流
             * </p>
             */
            public class FileDemo {
            	/**
            	 * @param args
            	 */
            	public static void main(String[] args) {
            		copy("/home/monuo/mylog/linux-1st", "/root/Desktop/hell0 11");
            	}
            	public static void copy(String src, String target){
            		InputStream inputStream = null;
            		OutputStream outputStream = null;
            		try {
            			inputStream = new FileInputStream( new File(src));
            			File file = new File(target);
            			if(!file.getParentFile().exists()){
            				file.mkdirs();
            			}
            			byte[] b = new byte[1024];
            			int temp = 0;
            			outputStream = new FileOutputStream(file);
            			while((temp = inputStream.read(b)) != -1){
            				outputStream.write(b, 0, temp);
            			}
            			outputStream.flush();
            		} catch (FileNotFoundException e) {
            			e.printStackTrace();
            			System.out.println("not exists");
            			return;
            		} catch (IOException e) {
            			e.printStackTrace();
            		} finally{
            			close(inputStream, outputStream);
            		}
            	}
            	public static void close(InputStream inputStream, OutputStream outputStream){
            		if(inputStream != null){
            			try {
            				inputStream.close();
            			} catch (IOException e) {
            				e.printStackTrace();
            			}
            		}
            		if(outputStream != null){
            			try {
            				outputStream.close();
            			} catch (IOException e) {
            				e.printStackTrace();
            			}
            		}
            	}
            }

#### directorFile   
获取目录下的所有文件，主要根据File 的一些方法

+ isDirectory(),
+ exists();
+ delete();
+ getParentFile();

        /**
         * @author mnuo
         * <p>获取目录下的所有文件，主要根据File 的一些方法：<br>
         *     	isDirectory(),exists();delete();getParentFile();
         * </p>
         */
        public class DirectorFile {
        
        	/**
        	 * @param args
        	 */
        	public static void main(String[] args) {
        		File file = new File("/home/monuo");
        		print(file);
        	}
        	
        	public static void print(File file){
        		if(file == null)
        			return;
        		
        		if(file.isDirectory()){
        			File[] files = file.listFiles();
        			for(int i = 0; i < files.length; i ++){
        				print(files[i]);
        			}
        		}
        		System.out.println(file);
        	}
        }
