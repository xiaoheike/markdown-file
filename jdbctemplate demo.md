## 可运行程序 ##



## 结构 ##
![总体结构](http://i.imgur.com/6Trq53t.png)

## pom.xml ##
加载需要的jar包。
```XML
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.javacodegeeks.snippets.enterprise</groupId>
	<artifactId>SpringJdbcTemplateExample</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<properties>
		<spring.version>4.0.3.RELEASE</spring.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-core</artifactId>
			<version>${spring.version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-context</artifactId>
			<version>${spring.version}</version>
		</dependency>
		<!-- mysql jar -->
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<version>5.1.26</version>
		</dependency>
		<!-- jdbcTemplate jar -->
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-jdbc</artifactId>
			<version>${spring.version}</version>
		</dependency>
	</dependencies>

</project>
```

## applicationContext.xml ##
spring jdbcTemplate 配置
```XML
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
	xmlns:aop="http://www.springframework.org/schema/aop" xmlns:context="http://www.springframework.org/schema/context"
	xmlns:jee="http://www.springframework.org/schema/jee" xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:task="http://www.springframework.org/schema/task"
	xsi:schemaLocation="http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-3.2.xsd http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.2.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.2.xsd http://www.springframework.org/schema/jee http://www.springframework.org/schema/jee/spring-jee-3.2.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-3.2.xsd http://www.springframework.org/schema/task http://www.springframework.org/schema/task/spring-task-3.2.xsd">


	
	<bean id="jdbcEmployeeDAO" class="com.javacodegeeks.snippets.enterprise.dao.impl.JDBCEmployeeDAOImpl">
		<property name="jdbcTemplate" ref="jdbcTemplate" />
	</bean>
	
	<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="dataSource"></property>
    </bean>
    
	<bean id="dataSource"
		class="org.springframework.jdbc.datasource.DriverManagerDataSource">

		<property name="driverClassName" value="com.mysql.jdbc.Driver" />
		<property name="url" value="jdbc:mysql://localhost:3306/testdb" />
		<property name="username" value="root" />
		<property name="password" value="123456" />
	</bean>
</beans>
```
## JDBCExamployeeDAOImpl.java实现 ##
spring jdbcTemplate 使用
```java
package com.javacodegeeks.snippets.enterprise.dao.impl;

import java.sql.PreparedStatement;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.List;
import java.util.Map;

import org.springframework.jdbc.core.BatchPreparedStatementSetter;
import org.springframework.jdbc.core.BeanPropertyRowMapper;
import org.springframework.jdbc.core.JdbcTemplate;

import com.javacodegeeks.snippets.enterprise.Employee;
import com.javacodegeeks.snippets.enterprise.dao.JDBCEmployeeDAO;

public class JDBCEmployeeDAOImpl implements JDBCEmployeeDAO {
    private JdbcTemplate jdbcTemplate;

    public void setJdbcTemplate(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    public JdbcTemplate getJdbcTemplate() {
        return jdbcTemplate;
    }

    public void insert(Employee employee) {
        String sql = "INSERT INTO EMPLOYEE(empid, empname, salary) VALUES (?, ?, ?)";
        jdbcTemplate.update(sql, new Object[] { employee.getEmpid(), employee.getEmpname(), employee.getSalary() });
    }

    @SuppressWarnings({ "unchecked", "rawtypes" })
    public Employee findById(int id) {
        String sql = "SELECT * FROM EMPLOYEE WHERE empid = ?";
        Employee employee = (Employee) jdbcTemplate.queryForObject(sql, new Object[] { id }, new BeanPropertyRowMapper(Employee.class));
        return employee;
    }

    @SuppressWarnings("rawtypes")
    public List<Employee> findAll() {
        String sql = "SELECT * FROM EMPLOYEE";
        List<Employee> employees = new ArrayList<Employee>();
        List<Map<String, Object>> rows = jdbcTemplate.queryForList(sql);
        for (Map row : rows) {
            Employee employee = new Employee();
            employee.setEmpid(Integer.parseInt(String.valueOf(row.get("empid"))));
            employee.setEmpname((String) row.get("empname"));
            employee.setSalary(Integer.parseInt(String.valueOf(row.get("salary"))));
            employees.add(employee);
        }
        return employees;
    }

    public String findNameById(int id) {
        String sql = "SELECT empname FROM EMPLOYEE WHERE empid = ?";
        String name = (String) jdbcTemplate.queryForObject(sql, new Object[] { id }, String.class);
        return name;
    }

    public void insertBatchSQL(final String sql) {
        jdbcTemplate.batchUpdate(new String[] { sql });
    }

    public void insertBatch1(final List<Employee> employees) {
        String sql = "INSERT INTO EMPLOYEE(empid, empname, salary) VALUES (?, ?, ?)";

        jdbcTemplate.batchUpdate(sql, new BatchPreparedStatementSetter() {

            public void setValues(PreparedStatement ps, int i) throws SQLException {
                Employee employee = employees.get(i);
                ps.setLong(1, employee.getEmpid());
                ps.setString(2, employee.getEmpname());
                ps.setInt(3, employee.getSalary());
            }

            public int getBatchSize() {
                return employees.size();
            }
        });
    }

    public void insertBatch2(final String sql) {
        jdbcTemplate.batchUpdate(new String[] { sql });
    }
}
```
