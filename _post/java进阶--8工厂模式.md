---
title: java进阶--8工厂模式
date: 2016-07-12 17:12:08 
tags: java
category: Java
---
#### 工厂模式

        /**
         * @author saxon
         * 工厂模式
         */
        public class FactoryModule {
            public static void main(String[] args) throws Exception {
        		Animal cat = Factory.getInstance(Cat.class);
        		cat.whoAmI();
        		Animal dog = Factory.getInstance(Dog.class);
        		dog.whoAmI();
        	}
        
        }
        
        interface Animal{
        	void whoAmI();
        }
        class Factory{
        	static Animal getInstance(Class<? extends Animal> clazz) throws Exception{
        		return clazz.newInstance();
        	}
        }
        class Cat implements Animal{
        	@Override
        	public void whoAmI() {
        		System.out.println("I'm a cat,my name is ZhangSan");
        	}
        }
        class Dog implements Animal{
        	@Override
        	public void whoAmI() {
        		System.out.println("I'm a Dog,my name is LiSi");
        	}
        }
