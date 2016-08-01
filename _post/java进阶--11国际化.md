---
title: java进阶--11国际化
date: 2016-07-12 17:12:08 
tags: java
category: Java
---
#### 简述
国际化主要涉及到两个方面:

+ 一个是Locale类,这个类可以改变或者获取语言
+ 另一个是ResourceBunlde
        
        /**
         * @author saxon
         * <p>国际化主要涉及到两个方面:<br>一个是Locale类,这个类可以改变或者获取语言<br>另一个是ResourceBunlde</p>
         * 
         */
        public class InternationalDemo {
            /**
        	 * @param args
        	 */
        	public static void main(String[] args) {
        //		Locale locale = new Locale("en","US");
        		Locale locale = new Locale("zh_CN");
        		
        		ResourceBundle resourceBundle = ResourceBundle.getBundle("com.mnuocom.javapenetrateinto.international.International", locale);
        		String str = resourceBundle.getString("welcome");
        		String fomatterStr = resourceBundle.getString("welcome.all");
        		str = MessageFormat.format(fomatterStr, "Tom");
        		
        		System.out.println(str);
        
        	}
        
        }
    international_zh_CN.properties
        welcome=\u4F60\u597D
        welcome.all={0},\u4F60\u597D
     international_zh_US.properties
       welcome=Hello wold!
        welcome.all={0},Hello!
