## 工程结构 ##
![spring jpa hibernate](http://i.imgur.com/baf7C6q.png)

## persistence.xml ##

persistence.xml用于配置orm的工具是什么，比如下面的配置指明是hibernate。这个文件不是必须的，可以通过在其他的配置文件中添加相关找内容可以替代。
```JAVA
<?xml version="1.0" encoding="UTF-8"?>
<persistence version="2.0"
	xmlns="http://java.sun.com/xml/ns/persistence" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://java.sun.com/xml/ns/persistence 
	http://java.sun.com/xml/ns/persistence/persistence_2_0.xsd">

	<persistence-unit name="testPU"  transaction-type="RESOURCE_LOCAL">
		<!-- 扫描User对象的注解 -->
		<class>com.byteslounge.spring.tx.model.User</class>
		<provider>org.hibernate.ejb.HibernatePersistence</provider>
		<properties>
		    <property name="hibernate.dialect" value="org.hibernate.dialect.MySQL5Dialect"/>
            <property name="hibernate.hbm2ddl.auto" value="update"/>
            <property name="hibernate.show_sql" value="true"/>
            <property name="hiberante.format_sql" value="true"/>
		</properties>
	</persistence-unit>

</persistence>
```

## spring.xml ##

事务采用注解方式实现，bean id 必须为transactionManager
```XML
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
	xmlns:tx="http://www.springframework.org/schema/tx" xmlns:p="http://www.springframework.org/schema/p"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context-3.0.xsd 
    http://www.springframework.org/schema/tx 
    http://www.springframework.org/schema/tx/spring-tx.xsd">

	<!-- 扫描包中注解 -->
	<context:component-scan base-package="com.byteslounge.spring.tx.dao.impl" />
	<context:component-scan base-package="com.byteslounge.spring.tx.service.impl" />

	<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource"
		destroy-method="close">
		<property name="driverClassName" value="com.mysql.jdbc.Driver" />
		<property name="url" value="jdbc:mysql://localhost:3306/test" />
		<property name="username" value="root" />
		<property name="password" value="root" />
	</bean>

	<!-- jpa 设置 -->
	<bean id="entityManagerFactory"
		class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
		<!-- 加载persistence.xml文件 -->
		<property name="persistenceUnitName" value="testPU" />
		<property name="dataSource" ref="dataSource" />
	</bean>
	
	<!-- 使事务注解生效，例如@Transactional -->
	<tx:annotation-driven />
	<!-- id 必须为transactionManager,使用其他名字事务无法找到 -->
	<bean id="transactionManager" class="org.springframework.orm.jpa.JpaTransactionManager">
		<property name="entityManagerFactory" ref="entityManagerFactory" />
	</bean>

</beans>
```
### Dao ###

EntityManager 并没有再spring.xml中配置，这个应该是属于jpa自带的类
```JAVA
@Repository
public class UserDAOImpl implements UserDAO {

	@PersistenceContext
	private EntityManager entityManager;

	public void insertUser(User user) {
		entityManager.persist(user);
	}

	public List<User> findAllUsers() {
		CriteriaBuilder builder = entityManager.getCriteriaBuilder();
		CriteriaQuery<User> cq = builder.createQuery(User.class);
		Root<User> root = cq.from(User.class);
		cq.select(root);
		return entityManager.createQuery(cq).getResultList();
	}

}
```
