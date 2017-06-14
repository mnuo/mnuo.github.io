---
title: Hibernate 二级缓存
date: 2017-06-14 13:53:08 
tags: SSH
category: SSH
---
缓存是介于物理数据源与应用程序之间,是对数据库中的数据临时复制一份到内存中的容器,起作用就是为了减少应用程序与物理数据源的访问的次数,从而提高了应用程序的运行性能。
Hibernate在进行读取数据的时候,会根据缓存机制在相应的缓存中查找,如果再缓存中找到了需要的数据(缓存命中),则直接把命中的数据作为结果加以利用,避免了大量发送SQL语句到数据库的查询的性能消耗。

### 1 hibernate缓存策略提供商
提供了HashTable缓存，EHCache，OSCache，SwarmCache，jBoss Cathe2，这些缓存机制，其中EHCache，OSCache是不能用于集群环境（Cluster Safe）的，而SwarmCache，jBoss Cathe2是可以的。HashTable缓存主要是用来测试的，只能把对象放在内存中，EHCache，OSCache可以把对象放在内存（memory）中，也可以把对象放在硬盘（disk）上。

### 2 一级缓存
一级缓存是Session缓存,它属于事务范围的缓存;这一级别的缓存有Hibernate管理,一般情况下无需干预。
缓存范围：缓存只能被当前Session对象访问。缓存的生命周期依赖于Session的生命周期，当Session被关闭后，缓存也就结束生命周期。
+ 数据放入缓存：
	- 1. save()。当session对象调用save()方法保存一个对象后，该对象会被放入到session的缓存中。
	- 2. get()和load()。当session对象调用get()或load()方法从数据库取出一个对象后，该对象也会被放入到session的缓存中。
	- 3. 使用HQL和QBC等从数据库中查询数据。
+ 使用get()或load()证明缓存的存在：
		public class Client {
			public static void main(String[] args) {
				Session session = HibernateUtil.getSessionFactory().openSession();
				Transaction tx = null;
				try {
					//开启一个事务
					tx = session.beginTransaction();
					//从数据库中获取id="402881e534fa5a440134fa5a45340002"的Customer对象
					Customer customer1 = (Customer)session.get(Customer.class, "402881e534fa5a440134fa5a45340002");
					System.out.println("customer.getUsername is"+customer1.getUsername());
					//事务提交
					tx.commit();
					System.out.println("-------------------------------------");
					//开启一个新事务
					tx = session.beginTransaction();
					//从数据库中获取id="402881e534fa5a440134fa5a45340002"的Customer对象
					Customer customer2 = (Customer)session.get(Customer.class, "402881e534fa5a440134fa5a45340002");
					System.out.println("customer2.getUsername is"+customer2.getUsername());
					//事务提交
					tx.commit();
					System.out.println("-------------------------------------");
					//比较两个get()方法获取的对象是否是同一个对象
					System.out.println("customer1 == customer2 result is "+(customer1==customer2));
				}catch (Exception e){
					if(tx!=null){
						tx.rollback();
					}
				}finally{
					session.close();
				}
			}
		}
		程序控制台输出结果：
		Hibernate:
			select
				customer0_.id as id0_0_,
				customer0_.username as username0_0_,
				customer0_.balance as balance0_0_
			from
				customer customer0_
			where
				customer0_.id=?
		customer.getUsername islisi
		-------------------------------------
		customer2.getUsername islisi
		-------------------------------------
		customer1 == customer2 result is true
		其原理是：在同一个Session里面，第一次调用get()方法， Hibernate先检索缓存中是否有该查找对象，发现没有，Hibernate发送SELECT语句到数据库中取出相应的对象，然后将该对象放入缓存中，以便下次使用，第二次调用get()方法，Hibernate先检索缓存中是否有该查找对象，发现正好有该查找对象，就从缓存中取出来，不再去数据库中检索，没有再次发送select语句。
+ 数据从缓存中清除：
	- 1. evit()将指定的持久化对象从缓存中清除，释放对象所占用的内存资源，指定对象从持久化状态变为脱管状态，从而成为游离对象。 
	- 2. clear()将缓存中的所有持久化对象清除，释放其占用的内存资源。

+ 其他缓存操作：
	- 1. contains()判断指定的对象是否存在于缓存中。
	- 2. flush()刷新缓存区的内容，使之与数据库数据保持同步。

### 3 二级缓存
+ 测试问题示例:
		@Test
		public void testCache2() {
			Session session1 = sf.openSession();//获得Session1
			session1.beginTransaction();
			Category c = (Category)session1.load(Category.class, 1);
			System.out.println(c.getName());
			session1.getTransaction().commit();
			session1.close();
			Session session2 = sf.openSession();//获得Session2
			session2.beginTransaction();
			Category c2 = (Category)session2.load(Category.class, 1);
			System.out.println(c2.getName());
			session2.getTransaction().commit();
			session2.close();
		}
		当我们重启一个Session，第二次调用load或者get方法检索同一个对象的时候会重新查找数据库，会发select语句信息。
		原因：一个session不能取另一个session中的缓存。
		性能上的问题：假如是多线程同时去取Category这个对象，load一个对象，这个对像本来可以放到内存中的，可是由于是多线程，是分布在不同的session当中的，所以每次都要从数据库中取，这样会带来查询性能较低的问题。
+ 定义
		SessionFactory级别的缓存，可以跨越Session存在，可以被多个Session所共享。
+ 什么样的数据适合存放到第二级缓存中
	- 1) 很少被修改的数据 
	- 2) 不是很重要的数据，允许出现偶尔并发的数据 
	- 3) 不会被并发访问的数据 
	- 4) 参考数据,指的是供应用参考的常量数据，它的实例数目有限，它的实例会被许多其他类的实例引用，实例极少或者从来不会被修改
+ 什么样的数据不适合存放二级缓存
	- 1) 经常被修改的数据 
	- 2) 财务数据，绝对不允许出现并发 
	- 3) 与其他应用共享的数据。
+ 常用的缓存插件 Hibernater二级缓存是一个插件，下面是几种常用的缓存插件：
	- EhCache：可作为进程范围的缓存，存放数据的物理介质可以是内存或硬盘，对Hibernate的查询缓存提供了支持。
	- OSCache：可作为进程范围的缓存，存放数据的物理介质可以是内存或硬盘，提供了丰富的缓存数据过期策略，对Hibernate的查询缓存提供了支持。
	- SwarmCache：可作为群集范围内的缓存，但不支持Hibernate的查询缓存。
	- JBossCache：可作为群集范围内的缓存，支持事务型并发访问策略，对Hibernate的查询缓存提供了支持。
+ 二级缓存实现原理
	　Hibernate如何将数据库中的数据放入到二级缓存中？注意，你可以把缓存看做是一个Map对象，它的Key用于存储对象OID，Value用于存储POJO。首先，当我们使用Hibernate从数据库中查询出数据，获取检索的数据后，Hibernate将检索出来的对象的OID放入缓存中key 中，然后将具体的POJO放入value中，等待下一次再次向数据查询数据时，Hibernate根据你提供的OID先检索一级缓存，若有且配置了二级缓存，则检索二级缓存，如果还没有则才向数据库发送SQL语句，然后将查询出来的对象放入缓存中。

### 4 使用二级缓存
+ 打开二级缓存：
		Hibernate配置二级缓存,在主配置文件中hibernate.cfg.xml ：
			<!-- 使用二级缓存 -->
			<property name="cache.use_second_level_cache">true</property>
			<!--设置缓存的类型，设置缓存的提供商-->
			<property name="cache.provider_class">org.hibernate.cache.EhCacheProvider</property>
+ 配置ehcache.xml
		<ehcache>
		<!-- 缓存到硬盘的路径 -->
		<diskStore path="d:/ehcache"/>
		<defaultCache 
			maxElementsInMemory="200"	<!-- 最多缓存多少个对象 -->
			eternal="false"				<!-- 内存中的对象是否永远不变 -->
			timeToIdleSeconds="50"		<!--发呆了多长时间，没有人访问它，这么长时间清除 -->
			timeToLiveSeconds="60"		<!--活了多长时间，活了1200秒后就可以拿走，一般Live要比Idle设置的时间长 -->
			overflowToDisk="true"/> 	<!--内存中溢出就放到硬盘上 -->
		<!-- 指定缓存的对象，缓存哪一个实体类下面出现的属性覆盖上面出现的，没出现的继承上面的。-->
		<cache name="com.suxiaolei.hibernate.pojos.Order"
			maxElementsInMemory="200"
			eternal="true"
			timeToIdleSeconds="0"
			timeToLiveSeconds="0"
			overflowToDisk="false"/> 
		</ehcache>

+ 使用二级缓存需要在实体类中加入注解：
		需要ehcache-1.2.jar包, commons_loging1.1.1.jar包
		@Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
		Load默认使用二级缓存，就是当查一个对象的时候，它先会去二级缓存里面去找，如果找到了就不去数据库中查了。
		Iterator默认的也会使用二级缓存，有的话就不去数据库里面查了，不发送select语句了。
		List默认的往二级缓存中加数据，假如有一个query，把数据拿出来之后会放到二级缓存，但是执行查询的时候不会到二级缓存中查，会在数据库中查。原因每个query中查询条件不一样。

+ 也可以在需要被缓存的对象中hbm文件中的<class>标签下添加一个<cache>子标签:
		<hibernate-mapping>
			<class name="com.suxiaolei.hibernate.pojos.Order" table="orders">
				<cache usage="read-only"/>
				<id name="id" type="string">
					<column name="id"></column>
					<generator class="uuid"></generator>
				</id>
				<property name="orderNumber" column="orderNumber" type="string"></property>
				<property name="cost" column="cost" type="integer"></property>
			   
				<many-to-one name="customer" class="com.suxiaolei.hibernate.pojos.Customer" 
					column="customer_id" cascade="save-update">
				</many-to-one>       
			</class>
		</hibernate-mapping>
		存在一对多的关系，想要在在获取一方的时候将关联的多方缓存起来，需要再集合属性下添加<cache>子标签，这里需要将关联的对象的 hbm文件中必须在存在<class>标签下也添加<cache>标签，不然Hibernate只会缓存OID。
		<hibernate-mapping>
			<class name="com.suxiaolei.hibernate.pojos.Customer" table="customer">
				<!-- 主键设置 -->
				<id name="id" type="string">
					<column name="id"></column>
					<generator class="uuid"></generator>
				</id>
			    <!-- 属性设置 -->
				<property name="username" column="username" type="string"></property>
				<property name="balance" column="balance" type="integer"></property>
			   	<set name="orders" inverse="true" cascade="all" lazy="false" fetch="join">
					<cache usage="read-only"/>
					<key column="customer_id" ></key>
					<one-to-many class="com.suxiaolei.hibernate.pojos.Order"/>
				</set>
			</class>
		</hibernate-mapping>

 

+ 在程序中必须手动启用查询缓存
		query.setCacheable(true);// 启用查询缓存
		query.setCacheRegion("queryCache"); // 设置查询缓存区域
		如：
		Query query = getSession().createQuery(queryString);
		query.setCacheable(true);
		query.setCacheRegion("queryCache");
		if (values != null) {
			query.setProperties(values);
		}
		List<T> result = query.list();
		因为ehcache不支持集群，一般用在管理后台中。做集群可选用memcache。

文章来源: 
http://www.cnblogs.com/shz365/p/3759125.html
		