## [理解 Hibernate 一级缓存](http://howtodoinjava.com/hibernate/understanding-hibernate-first-level-cache-with-example/) ##
Hibernate 一级缓存默认是打开，不需要任何的配置。实际上，你无法强制禁止它的使用。
如果你理解了一级缓存实际上和**会话**是关联的，就很容易理解一级缓存。总所周知，会话是当我们需要时从会话工厂创建并且一旦会话关闭，缓存就会丢失。相似的，一级缓存与会话对象相关联，在会话存活期间是可用的。相同应用中的不同会话是无法相互访问的。

## 重点##
- 一级缓存和会话相关联，应用中的会话无法知道其他会话中的缓存
- 缓存的范围是在会话范围内。一旦会话被关闭，缓存将永远消失
- 一级缓存默认是打开的，并无法禁止
- 第一次查询一个实体会从数据库中检索，并被存放在与 hibernate 会话关联的一级缓存中
- 如果在一个会话中再次查询该实体，它将从一级缓存中加载，不会发送 sql 查询到数据库
- 加载的实体可以从会话中被移除，通过使用 `evict()` 方法。如果实体已经使用 `evict` 下次加载该实体将会再次调用数据库查询
- 整个会话缓存可以通过 `clear()` 方法移除。它将移除缓存中的所有实体

## 从一级缓存检索的例子 ##
在下面的例子中，将通过 hibernate 会话从数据库检索 `Department` 实体。多次检索该实体，观察 sql 语句是否被发出去。
```JAVA
//Open the hibernate session
Session session = HibernateUtil.getSessionFactory().openSession();
session.beginTransaction();

//fetch the department entity from database first time
DepartmentEntity department = (DepartmentEntity) session.load(DepartmentEntity.class, new Integer(1));
System.out.println(department.getName());

//fetch the department entity again
department = (DepartmentEntity) session.load(DepartmentEntity.class, new Integer(1));
System.out.println(department.getName());

session.getTransaction().commit();
HibernateUtil.shutdown();


Output:
Hibernate: select department0_.ID as ID0_0_, department0_.NAME as NAME0_0_ from DEPARTMENT department0_ where department0_.ID=?
Human Resource
Human Resource
```
从输出来看，第二次 `session.load()` 语句并没有执行 `select` 查询，而是直接加载 `department` 实体。说明实体对象却是被缓存了。

## 新会话测试一级缓存 ##
如果实体已经在一个会话中被获取，在新会话中，该实体将再次从数据库中获取。
```JAVA
//Open the hibernate session
Session session = HibernateUtil.getSessionFactory().openSession();
session.beginTransaction();

Session sessionTemp = HibernateUtil.getSessionFactory().openSession();
sessionTemp.beginTransaction();
try
{
	//fetch the department entity from database first time
	DepartmentEntity department = (DepartmentEntity) session.load(DepartmentEntity.class, new Integer(1));
	System.out.println(department.getName());
	
	//fetch the department entity again
	department = (DepartmentEntity) session.load(DepartmentEntity.class, new Integer(1));
	System.out.println(department.getName());
	
	department = (DepartmentEntity) sessionTemp.load(DepartmentEntity.class, new Integer(1));
	System.out.println(department.getName());
}
finally
{
	session.getTransaction().commit();
	HibernateUtil.shutdown();
	
	sessionTemp.getTransaction().commit();
	HibernateUtil.shutdown();
}


Output:
Hibernate: select department0_.ID as ID0_0_, department0_.NAME as NAME0_0_ from DEPARTMENT department0_ where department0_.ID=?
Human Resource
Human Resource

Hibernate: select department0_.ID as ID0_0_, department0_.NAME as NAME0_0_ from DEPARTMENT department0_ where department0_.ID=?
Human Resource
```
从输出可以发现及时 `department` 实体已经被存储在会话中，但是 `sessionTemp` 会话还是发出了一条数据库查询语句。说明不同会话之间的缓存是相互不可见的。

## 将实体对象从一级缓存中移除 ##
虽然无法禁用 hibernate 一级缓存，但是如果需要的话，可以移除该缓存对象。通过使用一下两个方法：
- `evict()`
- `clear()`

`evict()` 用于移除会话中的指定缓存对象，`clear()` 方法则用于移除会话中的所有缓存对象。一下代码展示移除一个缓存对象和移除所有缓存对象。
```JAVA
//Open the hibernate session
Session session = HibernateUtil.getSessionFactory().openSession();
session.beginTransaction();
try
{
	//fetch the department entity from database first time
	DepartmentEntity department = (DepartmentEntity) session.load(DepartmentEntity.class, new Integer(1));
	System.out.println(department.getName());
	
	//fetch the department entity again
	department = (DepartmentEntity) session.load(DepartmentEntity.class, new Integer(1));
	System.out.println(department.getName());
	
	session.evict(department);
	//session.clear(); 
	
	department = (DepartmentEntity) session.load(DepartmentEntity.class, new Integer(1));
	System.out.println(department.getName());
}
finally
{
	session.getTransaction().commit();
	HibernateUtil.shutdown();
}
		

Output:
Hibernate: select department0_.ID as ID0_0_, department0_.NAME as NAME0_0_ from DEPARTMENT department0_ where department0_.ID=?
Human Resource
Human Resource

Hibernate: select department0_.ID as ID0_0_, department0_.NAME as NAME0_0_ from DEPARTMENT department0_ where department0_.ID=?
Human Resource
```
从输出结果很明显可以看出，`evict()` 方法将 `department` 实体从一级缓存中移除，所以他再次从数据库中获取。

本文内容来自：[http://howtodoinjava.com/hibernate/understanding-hibernate-first-level-cache-with-example/](http://howtodoinjava.com/hibernate/understanding-hibernate-first-level-cache-with-example/)。本文只是翻译以及润色

欢迎转载，但请注明本文链接，谢谢你。
2016.9.21 9:08
