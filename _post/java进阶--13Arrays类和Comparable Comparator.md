---
title: java进阶--13Arrays类和对象比较
date: 2016-07-12 17:12:08 
tags: java
category: Java
---
#### 1 Arrasys
Arrays 主要用到方法:

 * Arrays.sort();//排序
 *  Arrays.banarySearch(Object[] arr, object obj);//二分查找法,但是在调用此方法之前需要先sort
 *  Arrays.equals(Object[] arr1, Object[] arr2);//比较,只有当长度,并且对应每个位置的元素值相等时才是true
 *  Arrays.fill();填充
        /**
         * ArraysDemo.java created at Jun 20, 2016 10:55:11 PM
         */
        package com.mnuocom.javapenetrateinto.arraycomparate;
        
        import java.util.Arrays;
        
        /**
         * @author saxon
         * Arrays 主要用到方法:
         *     Arrays.sort();//排序
         *  Arrays.banarySearch(Object[] arr, object obj);//二分查找法,但是在调用此方法之前需要先sort
         *  Arrays.equals(Object[] arr1, Object[] arr2);//比较,只有当长度,并且对应每个位置的元素值相等时才是true
         *  Arrays.fill();填充
         */
        public class ArraysDemo {
        
        	/**
        	 * @param args
        	 */
        	public static void main(String[] args) {
        		int[] arr1 = new int[]{30,10,4,22};
        		Arrays.sort(arr1);//比较
        		System.out.println(Arrays.toString(arr1));
        		
        		int[] arr2 = new int[]{30,10,4,22,44};
        		Arrays.sort(arr2);//比较
        		System.out.println(Arrays.binarySearch(arr2, 10));
        		System.out.println(Arrays.binarySearch(arr2, 5));
        		
        		System.out.println(arr1.equals(arr2));
        		System.out.println(Arrays.equals(arr1, arr2));
        		
        		Arrays.fill(arr1, 50);
        		System.out.println(Arrays.toString(arr1));
        	}
        
        }

#### 2 Comparable
+ 要实现对象的比较排序,对象要实现comparable接口,并且实现conparaTo(Object o)方法
        /**
         * @author saxon
         * 要实现对象的比较排序,对象要实现comparable接口,并且实现conparaTo(Object o)方法
         */
        public class ComparableDemo {
        
            /**
        	 * @param args
        	 */
        	public static void main(String[] args) {
        		Person[] arr = new Person[]{
        							new Person("张三", 22),
        							new Person("李四", 12),
        							new Person("王五", 32),
        							new Person("赵六", 21),
        							new Person("前妻", 20)
        						};
        		Arrays.sort(arr);
        		System.out.println(Arrays.toString(arr));
        	}
        
        }
        class Person implements Comparable<Object>{
        	private String name;
        	private int age;
        	
        	public Person(String name, int age) {
        		this.name = name;
        		this.age = age;
        	}
        	public String getName() {
        		return name;
        	}
        	public void setName(String name) {
        		this.name = name;
        	}
        	public int getAge() {
        		return age;
        	}
        	public void setAge(int age) {
        		this.age = age;
        	}
        
        
        	@Override
        	public String toString() {
        		return "\n{name: " + this.name + " ,age: " + this.age + "}";
        	}
        
        	@Override
        	public int compareTo(Object o) {
        		Person person = (Person) o;
        		return this.age - person.age;
        	}
        	
        }

#### 3 Comparator
+ Comparator 接口是重新编写一个比较类实现Comparator接口,重写compare方法
        /**
         * @author saxon
         * Comparator 接口是重新编写一个比较类实现Comparator接口,重写compare方法
         * Comparator和Comparable的区别:
         *     1 Comparable 和 Comparator 都是实现对象排序的接口
         *  2 java.lang.Comparable 是类在定义的时候实现comparable接口,方法里面重写comparato
         *  3 java.util.Comparator 是当对象需要比较排序的时候单独定义一个比较规则类,这个类里面有compare()和equals方法
         * 
         */
        public class ComparatorDemo {
        
        	/**
        	 * @param args
        	 */
        	public static void main(String[] args) {
        		Person1[] arr = new Person1[]{
        				new Person1("张三", 22),
        				new Person1("李四", 12),
        				new Person1("王五", 32),
        				new Person1("赵六", 21),
        				new Person1("前妻", 20)
        			};
        			Arrays.sort(arr, new PersonComparator());
        			System.out.println(Arrays.toString(arr));
        
        	}
        
        }
        class PersonComparator implements Comparator<Person1>{
        
        	@Override
        	public int compare(Person1 o1, Person1 o2) {
        		return o1.getAge() - o2.getAge();
        	}
        	
        }
        class Person1{
        	private String name;
        	private int age;
        	
        	public Person1(String name, int age) {
        		this.name = name;
        		this.age = age;
        	}
        	public String getName() {
        		return name;
        	}
        	public void setName(String name) {
        		this.name = name;
        	}
        	public int getAge() {
        		return age;
        	}
        	public void setAge(int age) {
        		this.age = age;
        	}
        
        	@Override
        	public String toString() {
        		return "\n{name: " + this.name + " ,age: " + this.age + "}";
        	}
        }

#### 4 Comparable和Comparator区别
 *  1 Comparable 和 Comparator 都是实现对象排序的接口
 *  2 java.lang.Comparable 是类在定义的时候实现comparable接口,方法里面重写comparato
 *  3 java.util.Comparator 是当对象需要比较排序的时候单独定义一个比较规则类,这个类里面有compare()和equals方法
       
#### 5 二叉数

        /**
         * @author saxon
         * 通过comparable实现二叉树
         */
        public class BinaryTreeDemo {
        
            /**
        	 * @param args
        	 */
        	public static void main(String[] args) {
        		BinaryTree bt = new BinaryTree();
        		bt.add(new Person("张三", 22));
        		bt.add(new Person("李四", 12));
        		bt.add(new Person("王五", 32));
        		bt.add(new Person("赵六", 21));
        		bt.add(new Person("前妻", 20));
        		Object[] objects = bt.toArray();
        		System.out.println(Arrays.toString(objects));
        	}
        
        }
        class BinaryTree {
        	class Node{
        		private Comparable<Object> data;
        		
        		public Node(Comparable<Object> data){
        			this.data = data;
        		}
        		private Node left;
        		private Node right;
        		
        		public void addNode(Node newNode){
        			if(this.data.compareTo(newNode.data) > 0){
        				if(this.left == null){
        					this.left = newNode;
        				}else{
        					this.left.addNode(newNode);
        				}
        			}else{
        				if(this.right == null){
        					this.right = newNode;
        				}else{
        					this.right.addNode(newNode);
        				}
        			}
        		}
        		
        		public void toArrayNode(){
        			if(this.left != null){
        				this.left.toArrayNode();
        			}
        			BinaryTree.this.result[BinaryTree.this.index ++] = this.data;
        			if(this.right != null){
        				this.right.toArrayNode();
        			}
        		}
        	}
        	private Node root;//根节点
        	private int count = 0;//node节点数
        	private int index = 0;//
        	private Object[] result;
        	
        	public void add(Comparable<Object> obj){
        		if(this.root == null){
        			this.root = new Node(obj);
        		}else{
        			this.root.addNode(new Node(obj));
        		}
        		count ++;
        	}
        	
        	public Object[] toArray(){
        		if(this.count > 0){
        			this.index = 0;
        			this.result = new Object[this.count];
        			this.root.toArrayNode();
        			return this.result;
        		}else{
        			return null;
        		}
        	}
        }
