---
title: java进阶--4String
date: 2016-07-12 17:12:08 
tags: java
category: Java
---
#### StringBuffer,String,StringBuilder
String,StringBuffer,StringBuilder的区别   

* String,StringBuffer,StringBuilder都是CharSequence接口的子类
* String 的内容一旦生成是不可以改变的,StringBuffer和StringBuilder是可以改变的
* StringBuffer是JDK1.0提供,而StringBuilder是JDK1.5之后才提供
* StringBuffer是线程安全的,性能不高,而StringBuilder是非线程安全的,性能更高


    /**
     * @author saxon
     *
     */
    public class StringBufferDemo {
        /**
    	 * @param args
    	 */
    	public static void main(String[] args) {
    		StringBuffer sb = new StringBuffer("Hello world!");
    		System.out.println(sb.reverse());
    		System.out.println(sb.reverse().delete(6, 11));
    		System.out.println(sb.insert(6, "Ameria"));
    	}
    
    }
