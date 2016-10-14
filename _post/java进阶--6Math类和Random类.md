---
title: java进阶--6Math类和Random类
date: 2016-07-12 17:12:08 
tags: java
category: Java
---
#### 1 RandomDemo
        /**
         * @author saxon
         */
        public class RandomDemo {
        
            public static void main(String[] args) {
        	int[] arr = new int[7];	
        	handar(arr);
        	print(arr);
        	}
        	public static int[] handar(int[] target){
        		int i = 0;
        		Random random = new Random();
        		while(i < 7){
        			int temp = random.nextInt(37);
        			if(isRepeat(target, temp))
        				target[i++] = temp;
        		}
        		return target;
        	}
        	public static boolean isRepeat(int[] target, int temp){
        		for(int j = 0 ; j < target.length; j ++){
        			if(target[j] == temp)
        				return false;
        		}
        		return true;
        	}
        	public static void print(int[] arr){
        		for(int i = 0; i < arr.length; i ++){
        			System.out.println(arr[i] + ",");
        		}
        		
        	}
        }
#### 2 Math
        /**
         * @author saxon
         */
        public class MathDemo {
            /**
        	 * @param args
        	 */
        	public static void main(String[] args) {
        		System.out.println(MathUtil.round(10.12124,2));
        	}
        
        }
        class MathUtil{
        	/**
        	 * round()方法 是四舍五入
        	 * pow(double a,double b)是a的b次方
        	 * 此方法是保留指定小数点并四舍五入
        	 * @param target
        	 * @param scale
        	 * @return
        	 */
        	public static double round(double target, int scale){
        		return Math.round(target * Math.pow(10.0, scale))/Math.pow(10, scale);
        	}
        }
#### 3 BigInteger
        /**
         * @author saxon
         */
        public class BigIntegerDemo {
            /**
        	 * @param args
        	 */
        	public static void main(String[] args) {
        		BigInteger bi = new BigInteger("321123131225555234234234");
        		BigInteger bi1 = new BigInteger("4234234234234");
        		System.out.println("加法: " + bi.add(bi1));
        		System.out.println("减法: " + bi.subtract(bi1));
        		System.out.println("乘法: " + bi.multiply(bi1));
        		System.out.println("除法: " + bi.divide(bi1));
        		System.out.println("余法: " + bi.divideAndRemainder(bi1)[1]);
        	}
        
        }
#### 4 BigDecimal
        public class BigDecimalDemo {
            public static void main(String[] args){
        		System.out.println(BigDecimalUtil.round(-15.50, 0));
        		System.out.println(BigDecimalUtil.round(-15.5106, 2));
        	}
        
        }
        class BigDecimalUtil{
        	public static double round(double target, int scale){
        		return new BigDecimal(target).divide(new BigDecimal(1), scale, BigDecimal.ROUND_HALF_UP).doubleValue();
        	}
        }

