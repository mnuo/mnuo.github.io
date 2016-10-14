---
title: java进阶--1线程
date: 2016-07-12 17:12:08 
tags: java
category: Java
---
#### 1 概述
进程是指一个内存中运行的应用程序，每个进程都有自己独立的一块内存空间，一个进程中可以启动多个线程。比如在Windows系统中，一个运行的exe就是一个进程。

线程是指进程中的一个执行流程，一个进程中可以运行多个线程。比如java.exe进程中可以运行很多线程。线程总是属于某个进程，进程中的多个线程共享进程的内存。
#### 2 两种方式和一种返回值的方式
+ Runnable接口
+ Thread类  
+ Callable接口(带有返回值)
> 使用Runnable,Callable接口接口实现线程时,通过Thread启动线程,Callable是带有返回值,通过实现call方法,FactureTask新建线程   

    Runnable例子:

        public class RunableTest {
    
            /**
        	 * @param args
        	 */
        	public static void main(String[] args) {
        		test2();
        		test1();
        
        	}
        	public static void test1(){
        		String name = "out test1";
        		new Thread(new Runnable() {
        			@Override
        			public void run() {
        				System.out.println(name + " running " + name);
        			}
        		}).start();
        	}
        	
        	public static void test2(){
        		String name = "test2";
        		new Thread(()->{
        			System.out.println(name + "running");
        		}).start();
        	}
        }

    Thread例子:
    
        public class ThreadTest {
                public static void main(String[] args) {
            		Thread thread1 = new myThread("thread1");
            		Thread thread2 = new myThread("thread2");
            		Thread thread3 = new myThread("thread3");
            		Thread thread4 = new myThread("thread4");
            		Thread thread5 = new myThread("thread5");
            		Thread thread6 = new myThread("thread6");
            		thread1.start();
            		thread2.start();
            		thread3.start();
            		thread4.start();
            		thread5.start();
            		thread6.start();
            	}
        }
        class myThread extends Thread{
        	String name;
        	public myThread(String name) {
        		this.name = name;
        	}
        	@Override
        	public void run() {
        		for(int i = 0; i < 10; i ++){
        			System.out.println(this.name +" runing");
        		}
        	}
        }
    Callable例子:
        /**
         * @author saxon
         * 带有返回值的线程Callable
         */
        public class CallableTest {
            public static void main(String[] args) throws InterruptedException, ExecutionException {
        		Callable<String> mc = new MyCallable();
        		FutureTask<String> task = new FutureTask<String>(mc);
        		Thread thread11 = new Thread(task);
        		thread11.start();
        		System.out.println(task.get());
        	}
        }
        
        class MyCallable implements Callable<String>{
        	private int ticker = 5;
        	@Override
        	public String call() {
        		for (int i = 0; i < 10; i++) {
        			if(ticker > 0 ){
        				System.out.println("runing" + ticker--);
        			}
        		}
        		return  "run over";
        	}
        }

#### 3 线程共享
> 线程共享建议使用Runnable,使用Thread实现比较麻烦

        /**
         * @author saxon
         * <p>线程共享</p>
         */
        public class ThreadRunnable {
            
        	public static void main(String[] args) {
        		try {
        			new ThreadRunnable().test3();
        		} catch (Exception e) {
        			e.printStackTrace();
        		}
        	}
        	//线程不共享
        	public void test1(){
        		Thread t1 = new MyThread2();
        		Thread t2 = new MyThread2();
        		Thread t3 = new MyThread2();
        		Thread t4 = new MyThread2();
        		Thread t5 = new MyThread2();
        		t1.start();
        		t2.start();
        		t3.start();
        		t4.start();
        		t5.start();
        	}
        	//Runnable实现共享
        	public void test2() throws Exception {
        		MyThread1 mt = new MyThread1();
        		new Thread(mt).start();
        		new Thread(mt).start();
        		new Thread(mt).start();
        		new Thread(mt).start();
        		new Thread(mt).start();
        		new Thread(mt).start();
        	}
        	//Thread实现共享
        	public void test3() throws Exception {
        		MyThread2 mt = new MyThread2();
        		new Thread(mt).start();
        		new Thread(mt).start();
        		new Thread(mt).start();
        		new Thread(mt).start();
        		new Thread(mt).start();
        		new Thread(mt).start();
        	}
        	
        }
        
        class MyThread2 extends Thread{
        	private int ticker = 5;
        	@Override
        	public void run() {
        		for (int i = 0; i < 10; i++) {
        			if(ticker > 0){
        				System.out.println("run" + ticker --);
        			}
        		}
        	}
        }
        class MyThread1 implements Runnable{
        	private int ticker = 5;
        	@Override
        	public void run() {
        		for (int i = 0; i < 10; i++) {
        			if(ticker > 0){
        				System.out.println("run" + ticker --);
        			}
        		}
        	}
        }
#### 4 获取线程名称
>  Thread.currentThread().getName()

        /**
         * @author saxon
         */
        public class GetThreadName {
            public static void main(String[] args) {
        		GetThreadNameTest gt = new GetThreadNameTest();
        		new Thread(gt,"MyThread").start();
        		gt.run();
        	}
        
        }
        class GetThreadNameTest implements Runnable{
        	@Override
        	public void run() {
        		System.out.println("Thread:" + Thread.currentThread().getName());
        	}
        }
#### 5 线程优先级
> 线程的优先级分成1-10,但是设置了优先级不一定最先执行

        /**
        * @author saxon
        * <p>线程优先级</p>
        */
        public class ThreadPro {
        
            /**
            * @param args
            */
            public static void main(String[] args) {
                System.out.println(Thread.currentThread().getPriority());
                TestPriority tp = new TestPriority();
                
                Thread t1 = new Thread(tp, "ThreadA");
                Thread t2 = new Thread(tp, "ThreadB");
                Thread t3 = new Thread(tp, "ThreadC");
                Thread t4 = new Thread(tp, "ThreadD");
                
                t1.setPriority(Thread.MAX_PRIORITY);
                t2.setPriority(Thread.MIN_PRIORITY);
                t3.setPriority(Thread.MIN_PRIORITY);
                t4.setPriority(Thread.NORM_PRIORITY);
                
                t1.start();
                t2.start();
                t3.start();
                t4.start();
            
            }
        
        }
        class TestPriority implements Runnable{
            @Override
            public void run() {
            	for (int i = 0; i < 20; i++) {
            		try {
            			Thread.sleep(500);
            		} catch (InterruptedException e) {
            			e.printStackTrace();
            		}
            		System.out.println(Thread.currentThread().getName() + ": " + i);
            	}
            }
        }
#### 6 线程sleep和interrupt
> 线程睡眠时间到了之后,自动唤醒,而线程等待是需要手动唤醒object方法中的notify()方法
        
        /**
         * @author saxon
         */
        public class ThreadSleepInterruple {
        
            /**
        	 * @param args
        	 * @throws InterruptedException 
        	 */
        	public static void main(String[] args) throws InterruptedException {
        		Thread t = new Thread(new ThreadSleep(), "ThreadA");
        		t.start();
        		new Thread(new ThreadSleep(), "ThreadB").start();
        		new Thread(new ThreadSleep(), "ThreadC").start();
        		Thread.sleep(3000);
        		t.interrupt();
        	}
        
        }
        class ThreadSleep implements Runnable {
        	@Override
        	public void run() {
        		for(int i = 0; i < 10; i ++){
        			try {
        				Thread.sleep(1000);
        				System.err.println(Thread.currentThread().getName() +":"+ i);
        			} catch (InterruptedException e) {
        				System.err.println("Interrupte " + Thread.currentThread().getName());
        			}
        		}
        	}
        }
    
#### 7 一个生产和消费者模型
        /**
         * <p>这是模仿生产者和消费者模型:<br>
         *     用于了解线程间的通信</p>
         * @author saxon
         */
        public class ProducterCustomer {
        
        	/**
        	 * @param args
        	 */
        	public static void main(String[] args) {
        		Info info = new Info();
        		Thread product = new Thread(new Producter(info));
        		Thread customer = new Thread(new Customer(info));
        		product.start();
        		customer.start();
        	}
        
        }
        class Info{
        	private String title;
        	private String content;
        	
        	private boolean flag = true;
        	//false 表示仓库为红灯,说明里面有数据,不应该生产,应该取走
        	//true 表示仓库为绿灯,说明里面没有数据,应该生产,不应该取走
        	public synchronized void setInfo(String title, String content){
        		if(flag == false){//表示仓库中有数据,不应该生产,等待取走数据
        			try {
        				super.wait();
        			} catch (InterruptedException e) {
        				e.printStackTrace();
        			}
        		}
        		this.title = title;
        		this.content = content;
        		try {
        			Thread.sleep(100);
        		} catch (InterruptedException e) {
        			e.printStackTrace();
        		}
        		//生产完成之后,仓库有数据,应该把绿灯变成红灯,并且唤醒取数据的线程
        		flag = false;
        		super.notify();
        	}
        	public synchronized void getInfo(){
        		if(flag == true){//表示仓库中没有数据,不应该取东西,应该等待生成完数据
        			try {
        				super.wait();
        			} catch (InterruptedException e) {
        				e.printStackTrace();
        			}
        		}
        		//如果是红灯,说明仓库中有数据,进行取数据操作
        		System.out.println(this.title + "---->" + this.content);
        		try {
        			Thread.sleep(200);
        		} catch (InterruptedException e) {
        			e.printStackTrace();
        		}
        		//数据取完成,将红灯变成绿灯,并唤醒生产线程进行生产
        		flag = true;
        		super.notify();
        	}
        }
        
        class Producter implements Runnable{
        	private Info info;
        	public Producter(Info info) {
        		this.info = info;
        	}
        	@Override
        	public void run() {
        		for(int i = 0; i < 50; i ++){
        			if(i % 2 == 0){
        				info.setInfo("张三", "张三是帅哥");
        			}else{
        				info.setInfo("李四", "李四是美女");
        			}
        		}
        	}
        }
        class Customer implements Runnable{
        	private Info info;
        	public Customer(Info info) {
        		this.info = info;
        	}
        	@Override
        	public void run() {
        		for(int i = 0; i < 50; i ++){
        			info.getInfo();
        		}
        	}
        }

    
        
        
        
