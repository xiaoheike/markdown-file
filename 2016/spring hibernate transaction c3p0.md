数据库连接池c3p0相比较容易集成到项目中，所以就不给自己留源码工程了。不过还是需要参考自己之前的几篇笔记：

## 工程结构 ##
![spring hibernate transaction c3p0](http://i.imgur.com/4P0fK4n.png)

## applicationContext.xml ##
```XML
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
	xmlns:tx="http://www.springframework.org/schema/tx"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.0.xsd
		http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.0.xsd">

	<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource"
		destroy-method="close">
		<property name="driverClass" value="com.mysql.jdbc.Driver" />
		<property name="jdbcUrl" value="jdbc:mysql://localhost:3306/test" />
		<property name="user" value="root" />
		<property name="password" value="root" />
		
		<property name="minPoolSize" value="1" />
		<property name="maxPoolSize" value="20" />
		<property name="initialPoolSize" value="1" />
		<property name="maxIdleTime" value="25000" />
		<property name="acquireIncrement" value="1" />
		<property name="acquireRetryAttempts" value="30" />
		<property name="acquireRetryDelay" value="1000" />
		<property name="testConnectionOnCheckin" value="true" />
		<property name="automaticTestTable" value="c3p0TestTable" />
		<property name="idleConnectionTestPeriod" value="18000" />
		<property name="checkoutTimeout" value="3000" />
	</bean>

	<!-- Hibernate 5 SessionFactory Bean definition -->
	<bean id="sessionFactory"
		class="org.springframework.orm.hibernate5.LocalSessionFactoryBean">
		<property name="dataSource" ref="dataSource" />
		<property name="hibernateProperties">
			<props>
				<prop key="hibernate.dialect">org.hibernate.dialect.MySQLDialect</prop>
				<prop key="hibernate.hbm2ddl.auto">update</prop>
				<prop key="hibernate.show_sql">true</prop>
				<prop key="hiberante.format_sql">true</prop>
				<!-- <prop key="connection.autocommit">true</prop> -->
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

	<bean id="transactionManager"
		class="org.springframework.orm.hibernate5.HibernateTransactionManager">
		<property name="sessionFactory" ref="sessionFactory"></property>
	</bean>

	<tx:advice id="personServiceTxAdvice" transaction-manager="transactionManager">
		<tx:attributes>
			<!-- 表达式中的这些方法会执行如下的规则 -->
			<tx:method name="delete*" isolation="DEFAULT" read-only="false"
				propagation="REQUIRED" />
			<tx:method name="update*" isolation="DEFAULT" read-only="false"
				propagation="REQUIRED" />
			<tx:method name="save*" isolation="DEFAULT" read-only="false"
				propagation="REQUIRED" />
			<tx:method name="*" isolation="DEFAULT" read-only="false"
				propagation="REQUIRED" />
		</tx:attributes>
	</tx:advice>

	<aop:config>
		<aop:pointcut id="personServiceTxPointcut"
			expression="execution(* com.journaldev.dao.PersonDAO.*(..))" />
		<aop:advisor id="personServiceTxAdvisor" advice-ref="personServiceTxAdvice"
			pointcut-ref="personServiceTxPointcut" />
	</aop:config>

	<bean id="personDAO" class="com.journaldev.dao.PersonDAOImpl">
		<property name="sessionFactory" ref="sessionFactory" />
	</bean>
</beans>
```
## pom.xml ##
```XML
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>org.springframework.samples</groupId>
	<artifactId>SpringHibernateWithTransactionWithC3p0Example</artifactId>
	<version>0.0.1-SNAPSHOT</version>

	<properties>
		<!-- Generic properties -->
		<!-- <java.version>1.6</java.version> -->
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>

		<!-- Spring -->
		<spring-framework.version>4.2.5.RELEASE</spring-framework.version>

		<!-- Hibernate / JPA -->
		<hibernate.version>5.1.0.Final</hibernate.version>

		<!-- Logging -->
		<!-- <logback.version>1.0.13</logback.version> -->
		<slf4j.version>1.7.5</slf4j.version>

	</properties>

	<dependencies>
		<!-- Spring and Transactions -->
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-context</artifactId>
			<version>${spring-framework.version}</version>
		</dependency>
		
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-tx</artifactId>
			<version>${spring-framework.version}</version>
		</dependency>
		
		<!-- Spring ORM support -->
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-orm</artifactId>
			<version>${spring-framework.version}</version>
		</dependency>

		<!-- Logging with SLF4J & LogBack -->
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-api</artifactId>
			<version>${slf4j.version}</version>
			<scope>compile</scope>
		</dependency>
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-log4j12</artifactId>
			<version>${slf4j.version}</version>
		</dependency>


		<!-- Hibernate -->
		<dependency>
			<groupId>org.hibernate</groupId>
			<artifactId>hibernate-entitymanager</artifactId>
			<version>${hibernate.version}</version>
		</dependency>
		<dependency>
			<groupId>org.hibernate</groupId>
			<artifactId>hibernate-core</artifactId>
			<version>${hibernate.version}</version>
		</dependency>
		<!-- <dependency>
			<groupId>org.hibernate</groupId>
			<artifactId>hibernate-c3p0</artifactId>
			<version>${hibernate.version}</version>
		</dependency> -->

		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<version>5.1.9</version>
		</dependency>
		<!-- <dependency>
			<groupId>commons-dbcp</groupId>
			<artifactId>commons-dbcp</artifactId>
			<version>1.4</version>
		</dependency> -->
		
		<!-- 配置事务时使用到 -->
		<dependency>
			<groupId>org.aspectj</groupId>
			<artifactId>aspectjweaver</artifactId>
			<version>1.8.9</version>
		</dependency>
		<!-- c3p0-databasejar -->
		<dependency>
			<groupId>c3p0</groupId>
			<artifactId>c3p0</artifactId>
			<version>0.9.1.2</version>
		</dependency>


	</dependencies>
</project>
```