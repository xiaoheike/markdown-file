## Hibernate Save ##

hibernate save()方法能够保存实体到数据库，正如方法名称save这个单词所表明的意思。我们能够在事务之外调用这个方法，这也是我不喜欢使用这个方法保存数据的原因。**假如两个实体之间有关系（例如employee表和address表有一对一关系），如果在没有事务的情况下调用这个方法保存employee这个实体，除非调用flush()这个方法，否则仅仅employee实体会被保存。**

### Employee.java ###

为了方便理解，简化Employee.java的属性。
```JAVA	
/* 
 * @(#)Employee.java    Created on 2016年4月10日
 * Copyright (c) 2016. All rights reserved.
 */
package nd.esp.com.hibernate.model;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.OneToOne;
import javax.persistence.Table;

import org.hibernate.annotations.Cascade;

@Entity
@Table(name = "EMPLOYEE")
public class Employee {
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	@Column(name = "emp_id")
	private long id;
	@Column(name = "emp_name")
	private String name;
	@OneToOne(mappedBy = "employee")
	@Cascade(value = org.hibernate.annotations.CascadeType.ALL)
	private Address address;
	
	public long getId() {
		return id;
	}
	public void setId(long id) {
		this.id = id;
	}
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public Address getAddress() {
		return address;
	}
	public void setAddress(Address address) {
		this.address = address;
	}
	@Override
	public String toString() {
		return "Employee [id=" + id + ", name=" + name + "]";
	}
}
```

### Address.java ###

为了方便理解，简化Address.java的属性。
```JAVA
/* 
 * @(#)Address.java    Created on 2016年4月10日
 * Copyright (c) 2016. All rights reserved.
 */
package nd.esp.com.hibernate.model;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.persistence.OneToOne;
import javax.persistence.PrimaryKeyJoinColumn;
import javax.persistence.Table;

import org.hibernate.annotations.GenericGenerator;
import org.hibernate.annotations.Parameter;

@Entity
@Table(name = "ADDRESS")
public class Address {
	@Id
	@Column(name = "emp_id", unique = true, nullable = false)
	@GeneratedValue(generator = "gen")
	@GenericGenerator(name = "gen", strategy = "foreign", parameters = { @Parameter(name = "property", value = "employee") })
	private long id;
	@Column(name = "city")
	private String city;
	@OneToOne
	@PrimaryKeyJoinColumn
	private Employee employee;
	
	public long getId() {
		return id;
	}
	public void setId(long id) {
		this.id = id;
	}
	public String getCity() {
		return city;
	}
	public void setCity(String city) {
		this.city = city;
	}
	public Employee getEmployee() {
		return employee;
	}
	public void setEmployee(Employee employee) {
		this.employee = employee;
	}
	@Override
	public String toString() {
		return "Address [id=" + id + ", city=" + city + "]";
	}
}
```

### HibernateSaveExample.java ###

以下是简单的hibernate程序，演示save()方法的使用。
```JAVA	
/* 
 * @(#)HibernateSaveExample.java    Created on 2016年4月10日
 * Copyright (c) 2016. All rights reserved.
 */
package nd.esp.com.hibernate.example;

import nd.esp.com.hibernate.model.Address;
import nd.esp.com.hibernate.model.Employee;
import nd.esp.com.hibernate.utils.HibernateUtil;

import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.Transaction;

public class HibernateSaveExample {
	public static void main(String[] args) {
		SessionFactory sessionFactory = HibernateUtil.getSessionFactory();
		System.out.println("***********************************************");
		// save example - without transaction
		Session session = sessionFactory.openSession();
		Employee emp = getTestEmployee();
		long id = (Long) session.save(emp);
		System.out.println("1. Employee save called without transaction, id=" + id);
		session.flush(); // address will not get saved without this
		System.out.println("***********************************************");

		// save example - with transaction
		Transaction tx1 = session.beginTransaction();
		Session session1 = sessionFactory.openSession();
		Employee emp1 = getTestEmployee();
		long id1 = (Long) session1.save(emp1);
		System.out.println("2. Employee save called with transaction, id=" + id1);
		System.out.println("3. Before committing save transaction");
		tx1.commit();
		System.out.println("4. After committing save transaction");
		System.out.println("***********************************************");

		// save example - existing row in table
		Session session6 = sessionFactory.openSession();
		Transaction tx6 = session6.beginTransaction();
		Employee emp6 = (Employee) session6.load(Employee.class, new Long(3));
		// update some data
		System.out.println("Employee Details=" + emp6);
		emp6.setName("New Name");
		emp6.getAddress().setCity("New City");
		long id6 = (Long) session6.save(emp6);
		emp6.setName("New Name1"); // will get updated in database
		System.out.println("5. Employee save called with transaction, id=" + id6);
		System.out.println("6. Before committing save transaction");
		tx6.commit();
		System.out.println("7. After committing save transaction");
		System.out.println("***********************************************");

		// Close resources
		sessionFactory.close();
	}
	public static Employee getTestEmployee() {
		Employee emp = new Employee();
		Address add = new Address();
		emp.setName("Test Emp");
		add.setCity("Test City");
		emp.setAddress(add);
		add.setEmployee(emp);
		return emp;
	}
}
```

执行上述示例程序，输出结果（第一次）。
```JAVA	
	***********************************************
	Hibernate: insert into EMPLOYEE (emp_name) values (?)
	1. Employee save called without transaction, id=15
	Hibernate: insert into ADDRESS (city, emp_id) values (?, ?)
	***********************************************
	Hibernate: insert into EMPLOYEE (emp_name) values (?)
	2. Employee save called with transaction, id=16
	3. Before committing save transaction
	4. After committing save transaction
	***********************************************
	Hibernate: select employee0_.emp_id as emp1_1_1_, employee0_.emp_name as emp2_1_1_, address1_.emp_id as emp1_0_0_, address1_.city as city2_0_0_ from EMPLOYEE employee0_ left outer join ADDRESS address1_ on employee0_.emp_id=address1_.emp_id where employee0_.emp_id=?
	Employee Details=Employee [id=3, name=Test Emp]
	5. Employee save called with transaction, id=3
	6. Before committing save transaction
	Hibernate: update ADDRESS set city=? where emp_id=?
	Hibernate: update EMPLOYEE set emp_name=? where emp_id=?
	7. After committing save transaction
	***********************************************
```

执行上述示例程序，输出结果（第二次）。
```JAVA
	***********************************************
	Hibernate: insert into EMPLOYEE (emp_name) values (?)
	1. Employee save called without transaction, id=17
	Hibernate: insert into ADDRESS (city, emp_id) values (?, ?)
	***********************************************
	Hibernate: insert into EMPLOYEE (emp_name) values (?)
	2. Employee save called with transaction, id=18
	3. Before committing save transaction
	4. After committing save transaction
	***********************************************
	Hibernate: select employee0_.emp_id as emp1_1_1_, employee0_.emp_name as emp2_1_1_, address1_.emp_id as emp1_0_0_, address1_.city as city2_0_0_ from EMPLOYEE employee0_ left outer join ADDRESS address1_ on employee0_.emp_id=address1_.emp_id where employee0_.emp_id=?
	Employee Details=Employee [id=3, name=New Name1]
	5. Employee save called with transaction, id=3
	6. Before committing save transaction
	7. After committing save transaction
	***********************************************
```

从上述的输出可以得到以下重要的几点：

- 应该避免在事务之外调用save()方法，否则关联实体(例如employee和address是一对一关系，相互关联)将不会被保存从而导致不一致。很容易忘记在最后调用flush()方法，因为不会有任务的异常或者警告抛出。
- hibernate save()方法会立即返回id，原因很可能是调用save()的同时这个实体对象已经被写入数据库(立即执行sql语句insert into)
- **提交事务或者调用flush()方法**，save()方法才会将关联对象也写入数据库。
- save()方法保存持久化状态的对象，hibernate会通过update操作完成。注意这个会发生在提交事务的时候。如果该持久化对象没有改变，hibernate不会发出update语句。如果多次运行示例程序`HibernateSaveExample.java`，会发现从第二次开始程序就不会发送update语句。因为hibernate在更新之前会先select，查询该持久化对象，发现该对象和数据库中的一致，就不会做update操作。

## Hibernate Persist ##

hibernate persist()方法与save()方法（在事务中执行）类似，**persist()方法会将实体对象添加到持久化上下文中，如此被保存的实体后续改变会被记录。如果在提交事务或者会话flush()，对象的属性被重新赋值，那么这个变化也会被保存到数据库中。**

persist()方法只能够在事务内执行，不然数据无法保存到数据库中。

最后，persist()方法返回值是void，也就是说不会返回任何的值。

### HibernatePersistExample.java ###

以下是简单的hibernate程序，演示persist()方法的使用。
```JAVA
/* 
 * @(#)HibernatePersistExample.java    Created on 2016年4月10日
 * Copyright (c) 2016. All rights reserved.
 */
package nd.esp.com.hibernate.example;

import nd.esp.com.hibernate.model.Address;
import nd.esp.com.hibernate.model.Employee;
import nd.esp.com.hibernate.utils.HibernateUtil;

import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.Transaction;

public class HibernatePersistExample {
	public static void main(String[] args) {
		// Prep Work
		SessionFactory sessionFactory = HibernateUtil.getSessionFactory();
		System.out.println("***********************************************");
		// persist example - with transaction
		Session session2 = sessionFactory.openSession();
		Transaction tx2 = session2.beginTransaction();
		Employee emp2 = getTestEmployee();
		session2.persist(emp2);
		System.out.println("Persist called");
		emp2.setName("Kumar"); // will be updated in database too
		System.out.println("Employee Name updated");
		System.out.println("8. Employee persist called with transaction, id=" + emp2.getId() + ", address id="
				+ emp2.getAddress().getId());
		tx2.commit();
		System.out.println("***********************************************");
		// Close resources
		sessionFactory.close();
	}

	public static Employee getTestEmployee() {
		Employee emp = new Employee();
		Address add = new Address();
		emp.setName("Test Emp");
		add.setCity("Test City");
		emp.setAddress(add);
		add.setEmployee(emp);
		return emp;
	}
}
```

执行上述示例程序，输出结果。
```JAVA
	***********************************************
	Hibernate: insert into EMPLOYEE (emp_name) values (?)
	Persist called
	Employee Name updated
	8. Employee persist called with transaction, id=19, address id=19
	Hibernate: insert into ADDRESS (city, emp_id) values (?, ?)
	Hibernate: update EMPLOYEE set emp_name=? where emp_id=?
	***********************************************
```

需要注意，第一次employee对象被插入数据库，提交事务的时候执行address实体的插入操作，由于employee实体name属性重新赋值，所以执行update操作。

## Hibernate saveOrUpdate ##
hibernate saveOrUpdate()方法会执行插入或者更新操作。如果该对象在数据库中已经存在则更新，不存在则插入。

saveOrUpdate()方法可以在没有事务的情况下执行，但是如果没有手动调用flush()方法会面临关联对象不被保存的问题

save()方法与saveOrUpdate()方法最大的不同点在于，saveOrUpdate()方法会将实体对象添加到持久化上下文中，该实体的后续改变会被跟踪。

### HibernateSaveOrUpdateExample.java ###
以下是简单的hibernate程序，演示saveOrUpdate()方法的使用。
```JAVA
/* 
 * @(#)HibernateSaveOrUpdateExample.java    Created on 2016年4月10日
 * Copyright (c) 2016. All rights reserved.
 */
package nd.esp.com.hibernate.example;

import nd.esp.com.hibernate.model.Address;
import nd.esp.com.hibernate.model.Employee;
import nd.esp.com.hibernate.utils.HibernateUtil;

import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.Transaction;

public class HibernateSaveOrUpdateExample {
	public static void main(String[] args) {
		// Prep Work
		SessionFactory sessionFactory = HibernateUtil.getSessionFactory();
		System.out.println("***********************************************");
		// saveOrUpdate example - without transaction
		Session session5 = sessionFactory.openSession();
		Employee emp5 = getTestEmployee();
		session5.saveOrUpdate(emp5);
		System.out.println("***********************************************");

		// saveOrUpdate example - with transaction
		Session session3 = sessionFactory.openSession();
		Transaction tx3 = session3.beginTransaction();
		Employee emp3 = getTestEmployee();
		session3.saveOrUpdate(emp3);
		emp3.setName("Kumar"); // will be saved into DB
		System.out.println("9. Before committing saveOrUpdate transaction. Id=" + emp3.getId());
		tx3.commit();
		System.out.println("10. After committing saveOrUpdate transaction");
		System.out.println("***********************************************");

		Transaction tx4 = session3.beginTransaction();
		emp3.setName("Updated Test Name"); // Name changed
		emp3.getAddress().setCity("Updated City");
		session3.saveOrUpdate(emp3);
		emp3.setName("Kumar"); // again changed to previous value, so no Employee update
		System.out.println("11. Before committing saveOrUpdate transaction. Id=" + emp3.getId());
		tx4.commit();
		System.out.println("12. After committing saveOrUpdate transaction");
		System.out.println("***********************************************");

		// Close resources
		sessionFactory.close();
	}
	public static Employee getTestEmployee() {
		Employee emp = new Employee();
		Address add = new Address();
		emp.setName("Test Emp");
		add.setCity("Test City");
		emp.setAddress(add);
		add.setEmployee(emp);
		return emp;
	}
}
```
执行上述示例程序，输出结果。
```JAVA
	***********************************************
	Hibernate: insert into EMPLOYEE (emp_name) values (?)
	***********************************************
	Hibernate: insert into EMPLOYEE (emp_name) values (?)
	9. Before committing saveOrUpdate transaction. Id=21
	Hibernate: insert into ADDRESS (city, emp_id) values (?, ?)
	Hibernate: update EMPLOYEE set emp_name=? where emp_id=?
	10. After committing saveOrUpdate transaction
	***********************************************
	11. Before committing saveOrUpdate transaction. Id=21
	Hibernate: update ADDRESS set city=? where emp_id=?
	12. After committing saveOrUpdate transaction
	***********************************************
```

注意如果没有事务，仅仅是employee实体被保存到数据库，而address的信息丢失了。

在事务tx4中的几行代码employee实体的name属性先被修改为“Updated Test Name”，之后又被赋值为原来的值“Kumar”，因此employee这个实体在事务提交之前并没有改变，所以并没有update操作。

## Hibernate update ##

当确定只更新实体信息时使用Hibernate update()方法。update()方法会将实体添加到持久化上下文，实体后续的改变会被跟踪并且当事务提交时这些改变会被保存到数据库中。

### HibernateUpdateExample.java ###

以下是简单的hibernate程序，update()方法的使用。
```JAVA
/* 
 * @(#)HibernateUpdateExample.java    Created on 2016年4月10日
 * Copyright (c) 2016. All rights reserved.
 */
package nd.esp.com.hibernate.example;

import nd.esp.com.hibernate.model.Employee;
import nd.esp.com.hibernate.utils.HibernateUtil;

import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.Transaction;

public class HibernateUpdateExample {
	public static void main(String[] args) {
		// Prep Work
		SessionFactory sessionFactory = HibernateUtil.getSessionFactory();
		System.out.println("***********************************************");
		Session session = sessionFactory.openSession();
		Transaction tx = session.beginTransaction();
		Employee emp = (Employee) session.load(Employee.class, new Long(19));
		System.out.println("Employee object loaded. " + emp);
		tx.commit();
		System.out.println("***********************************************");
		// update example
		emp.setName("Updated name");
		emp.getAddress().setCity("Bangalore");
		Transaction tx7 = session.beginTransaction();
		session.update(emp);
		emp.setName("Final updated name");
		System.out.println("13. Before committing update transaction");
		tx7.commit();
		System.out.println("14. After committing update transaction");
		System.out.println("***********************************************");
		// Close resources
		sessionFactory.close();
	}
}
```

执行上述示例程序，输出结果（第一次）。
```JAVA
	***********************************************
	Hibernate: select employee0_.emp_id as emp1_1_1_, employee0_.emp_name as emp2_1_1_, address1_.emp_id as emp1_0_0_, address1_.city as city2_0_0_ from EMPLOYEE employee0_ left outer join ADDRESS address1_ on employee0_.emp_id=address1_.emp_id where employee0_.emp_id=?
	Employee object loaded. Employee [id=19, name=Kumar]
	***********************************************
	13. Before committing update transaction
	Hibernate: update ADDRESS set city=? where emp_id=?
	Hibernate: update EMPLOYEE set emp_name=? where emp_id=?
	14. After committing update transaction
	***********************************************
```
执行上述示例程序，输出结果（第二次）。
```JAVA	
	***********************************************
	Hibernate: select employee0_.emp_id as emp1_1_1_, employee0_.emp_name as emp2_1_1_, address1_.emp_id as emp1_0_0_, address1_.city as city2_0_0_ from EMPLOYEE employee0_ left outer join ADDRESS address1_ on employee0_.emp_id=address1_.emp_id where employee0_.emp_id=?
	Employee object loaded. Employee [id=19, name=Final updated name]
	***********************************************
	13. Before committing update transaction
	14. After committing update transaction
	***********************************************
```
注意第二次运行并没有执行update操作，因为employee实体与数据库中的一致。还有语句：
```JAVA
	session.update(emp);
	emp.setName("Final updated name");
```
修改employee实体name属性值为：“Final updated name”，是在update()方法之后，而最后保存到数据库中是“Final updated name”，表明hibernate update()方法会跟踪实体的改变，在提交事务时保存到数据库中。

## Hibernate Merge ##

**hibernate merge()方法被用于更新数据库中的记录，然而merge()方法通过创建一个传递进来的实体对象副本并且将这个副本作为返回值返回。返回值属于持久化上下文，能够跟踪实体的改变，而传递进来的实体并不能被跟踪**。这一点是merge()方法与其他方法最大的不同。
### HibernateMergeExample.java ###

以下是简单的hibernate程序，merge()方法的使用。
```JAVA
/* 
 * @(#)HibernateMergeExample.java    Created on 2016年4月10日
 * Copyright (c) 2016. All rights reserved.
 */
package nd.esp.com.hibernate.example;

import nd.esp.com.hibernate.model.Employee;
import nd.esp.com.hibernate.utils.HibernateUtil;

import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.Transaction;

public class HibernateMergeExample {
	public static void main(String[] args) {
		// Prep Work
		SessionFactory sessionFactory = HibernateUtil.getSessionFactory();
		System.out.println("***********************************************");
		Session session = sessionFactory.openSession();
		Transaction tx = session.beginTransaction();
		Employee emp = (Employee) session.load(Employee.class, new Long(19));
		System.out.println("Employee object loaded. " + emp);
		tx.commit();
		System.out.println("***********************************************");
		// merge example - data already present in tables
		emp.setName("test1");
		Transaction tx8 = session.beginTransaction();
		Employee emp4 = (Employee) session.merge(emp);
		System.out.println(emp4 == emp); // returns false
		emp.setName("test2");
		emp4.setName("merge");
		System.out.println("15. Before committing merge transaction");
		tx8.commit();
		System.out.println("16. After committing merge transaction");
		System.out.println("***********************************************");
		// Close resources
		sessionFactory.close();
	}
}
```

执行上述示例程序，输出结果（第一次）。
```JAVA
	***********************************************
	Hibernate: select employee0_.emp_id as emp1_1_1_, employee0_.emp_name as emp2_1_1_, address1_.emp_id as emp1_0_0_, address1_.city as city2_0_0_ from EMPLOYEE employee0_ left outer join ADDRESS address1_ on employee0_.emp_id=address1_.emp_id where employee0_.emp_id=?
	Employee object loaded. Employee [id=19, name=Kumar]
	***********************************************
	false
	15. Before committing merge transaction
	Hibernate: update EMPLOYEE set emp_name=? where emp_id=?
	16. After committing merge transaction
	***********************************************
```
执行上述示例程序，输出结果（第二次）。
```JAVA
	***********************************************
	Hibernate: select employee0_.emp_id as emp1_1_1_, employee0_.emp_name as emp2_1_1_, address1_.emp_id as emp1_0_0_, address1_.city as city2_0_0_ from EMPLOYEE employee0_ left outer join ADDRESS address1_ on employee0_.emp_id=address1_.emp_id where employee0_.emp_id=?
	Employee object loaded. Employee [id=19, name=merge]
	***********************************************
	false
	15. Before committing merge transaction
	16. After committing merge transaction
	***********************************************
```
注意merge()方法传递进入的实体与返回值实体是不一样的，
```JAVA
	Employee emp4 = (Employee) session.merge(emp);
	emp.setName("test2");
	emp4.setName("merge");
```
上述代码会将employee表的name属性赋值为“merge”，因为返回实体emp4属于持久化上下文，会被跟踪改变。

以上内容翻译自[http://www.journaldev.com/3481/hibernate-save-vs-saveorupdate-vs-persist-vs-merge-vs-update-explanation-with-examples](http://www.journaldev.com/3481/hibernate-save-vs-saveorupdate-vs-persist-vs-merge-vs-update-explanation-with-examples "Hibernate save, saveOrUpdate, persist, merge, update explanation with examples")。例子代码做了一点简化，并且工程化，可以在如下地址找到：[https://github.com/xiaoheike/HibernateSavePersistUpdateMergeDiff](https://github.com/xiaoheike/HibernateSavePersistUpdateMergeDiff "HibernateSavePersistUpdateMergeDiff")

## 总结 ##

save()方法：

- 应该避免在事务之外调用save()方法，否则关联实体(例如employee和address是一对一关系，相互关联)将不会被保存从而导致不一致。很容易忘记在最后调用flush()方法，因为不会有任务的异常或者警告抛出。
- hibernate save()方法会立即返回id，原因很可能是调用save()的同时这个实体对象已经被写入数据库(立即执行sql语句insert into)
- **提交事务或者调用flush()方法**，save()方法才会将关联对象也写入数据库。

persist()方法：

- **persist()方法会将实体对象添加到持久化上下文中，如此被保存的实体后续改变会被记录。如果在提交事务或者会话flush()，对象的属性被重新赋值，那么这个变化也会被保存到数据库中。**
- persist()方法只能够在事务内执行，不然数据无法保存到数据库中
- 最后，persist()方法返回值是void，也就是说不会返回任何的值。

saveOrUpdate()方法：

- hibernate saveOrUpdate()方法会执行插入或者更新操作。如果该对象在数据库中已经存在则更新，不存在则插入。
- saveOrUpdate()方法可以在没有事务的情况下执行，但是如果没有手动调用flush()方法会面临关联对象不被保存的问题
- save()方法与saveOrUpdate()方法最大的不同点在于，saveOrUpdate()方法会将实体对象添加到持久化上下文中，该实体的后续改变会被跟踪。

update()方法：

- 当确定只更新实体信息时使用Hibernate update()方法。update()方法会将实体添加到持久化上下文，实体后续的改变会被跟踪并且当事务提交时这些改变会被保存到数据库中
- hibernate update()方法会跟踪实体的改变，在提交事务时保存到数据库中。

merge()方法：
- **hibernate merge()方法被用于更新数据库中的记录，然而merge()方法通过创建一个传递进来的实体对象副本并且将这个副本作为返回值返回。返回值属于持久化上下文，能够跟踪实体的改变，而传递进来的实体并不能被跟踪**。这一点是merge()方法与其他方法最大的不同。

教程结束，感谢阅读。

欢迎转载，但请注明本文链接，谢谢。

2016/4/10 星期日 17:05:41 