## 工程结构 ##
![工程结构](http://i.imgur.com/xN07kRV.png)

## 问题描述 ##
在工程中通过spring aop的方式配置事务，使用hibernate做持久化。在代码实现中使用hibernate persit()方法插入数据到数据库，使用hibernate update()方法更新数据。问题是执行这两个方法没有报错，但是也没有插入数据或者更新数据。

## 原因 ##
hibernate persist()以及update()方法只有事务执行flush()或者commit()方法，才将数据写入数据库。详细内容可以阅读博客：[http://www.cnblogs.com/xiaoheike/p/5374613.html](http://www.cnblogs.com/xiaoheike/p/5374613.html "Hibernate save, saveOrUpdate, persist, merge, update 区别")。使用spring aop配置的事务，在方法运行结束之后会运行commit()方法。程序实例可以看**PersonDAOImpl.java（实现方法）**小结，**重点原因在于spring aop事务与session自己创建的事务是两个不同的事务，虽然最后spring aop 配置的事务 commit，但是session对象的事务并没有调用commit**。以下是实例程序。

## 事务配置(applicationContext.xml) ##
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
	xmlns:tx="http://www.springframework.org/schema/tx"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.0.xsd
		http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.0.xsd">

	<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource"
		destroy-method="close">
		<property name="driverClassName" value="com.mysql.jdbc.Driver" />
		<property name="url" value="jdbc:mysql://localhost:3306/Testdb" />
		<property name="username" value="root" />
		<property name="password" value="123456" />
	</bean>

	<!-- Hibernate 4 SessionFactory Bean definition -->
	<bean id="sessionFactory" class="org.springframework.orm.hibernate4.LocalSessionFactoryBean">
		<property name="dataSource" ref="dataSource"/>
		<property name="hibernateProperties">
			<props>
				<prop key="hibernate.dialect">org.hibernate.dialect.MySQLDialect</prop>
				<prop key="hibernate.hbm2ddl.auto">update</prop> 
                <prop key="hibernate.show_sql">true</prop>
                <prop key="hiberante.format_sql">true</prop>
			</props>
		</property>
		<!-- hibernate配置文件放置位置，这个配置文件似乎也没有多大的作用了 -->
		<property name="configLocations">
			<list>
				<value>
					classpath:/hibernate.cfg.xml
				</value>
			</list>
		</property>
	</bean>
	
	<bean id="transactionManager" class="org.springframework.orm.hibernate4.HibernateTransactionManager">
		<property name="sessionFactory" ref="sessionFactory"></property>
	</bean>
	
	<tx:advice id="personServiceTxAdvice" transaction-manager="transactionManager"> 
        <tx:attributes>
        	<!-- 表达式中的这些方法会执行如下的规则 --> 
            <tx:method name="delete*" isolation="DEFAULT" read-only="false" propagation="REQUIRED" /> 
            <tx:method name="update*" isolation="DEFAULT" read-only="false" propagation="REQUIRED" /> 
            <tx:method name="save*" isolation="DEFAULT" read-only="false" propagation="REQUIRED" />  
            <tx:method name="*" isolation="DEFAULT" read-only="false" propagation="REQUIRED" />
        </tx:attributes> 
    </tx:advice> 

    <aop:config> 
        <aop:pointcut id="personServiceTxPointcut" expression="execution(* com.journaldev.dao.PersonDAO.*(..))" /> 
        <aop:advisor id="personServiceTxAdvisor" advice-ref="personServiceTxAdvice" pointcut-ref="personServiceTxPointcut" /> 
    </aop:config>
	
	<bean id="personDAO" class="com.journaldev.dao.PersonDAOImpl">
		<property name="sessionFactory1" ref="sessionFactory" />
		<property name="sessionFactory2" ref="sessionFactory" />
	</bean>
</beans>
```

## PersonDAOImpl.java（实现方法） ##
```JAVA
package com.journaldev.dao;

import java.util.List;

import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.Transaction;

import com.journaldev.model.Person;

public class PersonDAOImpl implements PersonDAO {

    private SessionFactory sessionFactory1;
    private SessionFactory sessionFactory2;

    public void setSessionFactory1(SessionFactory sessionFactory1) {
        this.sessionFactory1 = sessionFactory1;
    }

    public void setSessionFactory2(SessionFactory sessionFactory2) {
        this.sessionFactory2 = sessionFactory2;
    }

    public void save1(Person person) {
        Session session1 = this.sessionFactory1.openSession();
        session1.persist(person);
        session1.close();
    }

    public void save2(Person person) {
        Session session1 = this.sessionFactory1.openSession();
        Session session2 = sessionFactory2.openSession();
        Transaction tx = session2.beginTransaction();
        session1.persist(person);
        tx.commit();
        session2.close();
        session1.close();
    }

    public void save3(Person person) {
        Session session1 = this.sessionFactory1.openSession();
        Transaction tx = session1.beginTransaction();
        session1.persist(person);
        tx.commit();
        session1.close();
    }

    @SuppressWarnings("unchecked")
    public List<Person> list() {
        Session session = this.sessionFactory1.openSession();
        List<Person> personList = session.createQuery("from Person").list();
        session.close();
        return personList;
    }

    public Person findOne(Integer id) {
        Session session = this.sessionFactory1.openSession();
        Person p = (Person) session.get(Person.class, id);
        session.close();
        return p;
    }

    public void delete(Person person) {
        Session session = this.sessionFactory1.openSession();
        session.delete(person);
        session.close();
    }

    public void update1(Person person) {
        Session session = this.sessionFactory1.openSession();
        session.update(person);
        session.close();
    }

    public void update2(Person person) {
        Session session1 = this.sessionFactory1.openSession();
        Session session2 = this.sessionFactory1.openSession();
        Transaction tx = session2.beginTransaction();
        session1.update(person);
        tx.commit();
        session2.close();
        session1.close();
    }

    public void update3(Person person) {
        Session session1 = this.sessionFactory1.openSession();
        Transaction tx = session1.beginTransaction();
        session1.update(person);
        tx.commit();
        session1.close();
    }
}
```

## 测试类 ##
```JAVA
package com.journaldev.main;

import java.util.List;

import org.springframework.context.support.ClassPathXmlApplicationContext;

import com.journaldev.dao.PersonDAO;
import com.journaldev.model.Person;

public class SpringHibernateMain {

    public static void main(String[] args) {
        test1();
        test2();
        test3();
    }

    public static void test1() {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        PersonDAO personDAO = context.getBean(PersonDAO.class);
        System.out.println("=================save1()==================");
        // 增加一条数据
        Person person = new Person();
        person.setName("Pankaj");
        person.setCountry("India");
        personDAO.save1(person);
        System.out.println("================listAll()===================");
        // 检索所有数据
        List<Person> list = personDAO.list();
        for (Person p : list) {
            System.out.println("所有记录：" + p);
        }
        System.out.println("================update1()===================");
        // 更新一条Person记录
        person.setCountry("zhongguo");
        personDAO.update1(person);
        System.out.println("更新一条记录India-->zhongguo：" + personDAO.findOne(person.getId()));
        System.out.println("================delete()===================");
        // 删除一条Person记录
        personDAO.delete(person);

        context.close();
    }

    public static void test2() {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        PersonDAO personDAO = context.getBean(PersonDAO.class);
        System.out.println("=================save3()==================");
        // 增加一条数据
        Person person = new Person();
        person.setName("Pankaj");
        person.setCountry("India");
        personDAO.save1(person);
        System.out.println("================listAll()===================");
        // 检索所有数据
        List<Person> list = personDAO.list();
        for (Person p : list) {
            System.out.println("所有记录：" + p);
        }
        System.out.println("================update2()===================");
        // 更新一条Person记录
        person.setCountry("zhongguo");
        personDAO.update1(person);
        System.out.println("更新一条记录India-->zhongguo：" + personDAO.findOne(person.getId()));
        System.out.println("================delete()===================");
        // 删除一条Person记录
        personDAO.delete(person);

        context.close();
    }

    public static void test3() {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        PersonDAO personDAO = context.getBean(PersonDAO.class);
        System.out.println("=================save3()==================");
        // 增加一条数据
        Person person = new Person();
        person.setName("Pankaj");
        person.setCountry("India");
        personDAO.save3(person);
        System.out.println("================listAll()===================");
        // 检索所有数据
        List<Person> list = personDAO.list();
        for (Person p : list) {
            System.out.println("所有记录：" + p);
        }
        System.out.println("================update3()===================");
        // 更新一条Person记录
        person.setCountry("zhongguo");
        personDAO.update3(person);
        System.out.println("更新一条记录India-->zhongguo：" + personDAO.findOne(person.getId()));
        System.out.println("================delete()===================");
        // 删除一条Person记录
        personDAO.delete(person);

        context.close();
    }
}
```

## 运行结果 ##
```JAVA
=================save1()==================
================update1()===================
Hibernate: select person0_.id as id1_0_0_, person0_.country as country2_0_0_, person0_.name as name3_0_0_ from PERSON person0_ where person0_.id=?
更新一条记录India-->zhongguo：null

=================save2()==================
================update2()===================
Hibernate: select person0_.id as id1_0_0_, person0_.country as country2_0_0_, person0_.name as name3_0_0_ from PERSON person0_ where person0_.id=?
更新一条记录India-->zhongguo：null

=================save3()==================
Hibernate: insert into PERSON (country, name) values (?, ?)
================update3()===================
Hibernate: update PERSON set country=?, name=? where id=?
Hibernate: select person0_.id as id1_0_0_, person0_.country as country2_0_0_, person0_.name as name3_0_0_ from PERSON person0_ where person0_.id=?
更新一条记录India-->zhongguo：id=8, name=Pankaj, country=zhongguo
```

## 原因分析 ##

一共有三个测试例子，第一个例子test1()方法，调用save1()方法，使用spring aop配置的事务，从输出结果可以看出，数据没有插入数据库。update1()方法与save1()方法是相同情况。

第二个例子test2()方法，调用save2()方法，persist()方法被包围在spring aop配置的事务和session2的事务中(事务有提交)，从输出结果可以看出，数据没有插入数据库。update2()方法与save2()方法是相同情况。

第三个例子test3()方法，persist()方法被包围在spring aop配置的事务和session1的事务中(事务有提交)，从输出结果可以看出，数据成功插入数据库。update3()方法与save3()方法是相同情况。

通过实例程序可以看出，persist()，以及update()方法需要在调用它们的session中的事务中执行，最后该session的事务需要commit。


教程结束，感谢阅读。

欢迎转载，但请注明本文链接，谢谢。

2016/4/15 18:38:01 