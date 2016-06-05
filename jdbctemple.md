- 使用JdbcTemplate的execute()方法执行SQL语句

```JAVA
jdbcTemplate.execute("CREATE TABLE USER (user_id integer, name varchar(100))");
```

- 如果是UPDATE或INSERT,可以用update()方法。

```JAVA
jdbcTemplate.update("INSERT INTO USER VALUES('"  
           + user.getId() + "', '"  
           + user.getName() + "', '"  
           + user.getSex() + "', '"  
           + user.getAge() + "')");
```

- 带参数的更新
 
```JAVA
jdbcTemplate.update("UPDATE USER SET name = ? WHERE user_id = ?", new Object[] {name, id});
```

```JAVA
jdbcTemplate.update("INSERT INTO USER VALUES(?, ?, ?, ?)", new Object[] {user.getId(), user.getName(), user.getSex(), user.getAge()});
```

- 使用JdbcTemplate进行查询时，使用queryForXXX()等方法

```JAVA
int count = jdbcTemplate.queryForInt("SELECT COUNT(*) FROM USER");
```

```JAVA
String name = (String) jdbcTemplate.queryForObject("SELECT name FROM USER WHERE user_id = ?", new Object[] {id}, java.lang.String.class);
```

```JAVA
List rows = jdbcTemplate.queryForList("SELECT * FROM USER"); 
```

```JAVA
List rows = jdbcTemplate.queryForList("SELECT * FROM USER");  
Iterator it = rows.iterator();  
while(it.hasNext()) {  
    Map userMap = (Map) it.next();  
    System.out.print(userMap.get("user_id") + "\t");  
    System.out.print(userMap.get("name") + "\t");  
    System.out.print(userMap.get("sex") + "\t");  
    System.out.println(userMap.get("age") + "\t");  
}  
```

JdbcTemplate将我们使用的JDBC的流程封装起来，包括了异常的捕捉、SQL的执行、查询结果的转换等等。spring大量使用Template Method模式来封装固定流程的动作，XXXTemplate等类别都是基于这种方式的实现。

除了大量使用Template Method来封装一些底层的操作细节，spring也大量使用callback方式类回调相关类别的方法以提供JDBC相关类别的功能，使传统的JDBC的使用者也能清楚了解spring所提供的相关封装类别方法的使用。

JDBC的PreparedStatement 

```JAVA
final String id = user.getId();  
final String name = user.getName();  
final String sex = user.getSex() + "";  
final int age = user.getAge();  
  
jdbcTemplate.update("INSERT INTO USER VALUES(?, ?, ?, ?)",  
                     new PreparedStatementSetter() {  
                         public void setValues(PreparedStatement ps) throws SQLException {  
                             ps.setString(1, id);  
                             ps.setString(2, name);            
                             ps.setString(3, sex);  
                             ps.setInt(4, age);  
                         }  
                     });  
```

```JAVA
final User user = new User();  
jdbcTemplate.query("SELECT * FROM USER WHERE user_id = ?",  
                    new Object[] {id},  
                    new RowCallbackHandler() {  
                        public void processRow(ResultSet rs) throws SQLException {  
                            user.setId(rs.getString("user_id"));  
                            user.setName(rs.getString("name"));  
                            user.setSex(rs.getString("sex").charAt(0));  
                            user.setAge(rs.getInt("age"));  
                        }  
                    });  
```

```JAVA
class UserRowMapper implements RowMapper {  
    public Object mapRow(ResultSet rs, int index) throws SQLException {  
        User user = new User();  
  
        user.setId(rs.getString("user_id"));  
        user.setName(rs.getString("name"));  
        user.setSex(rs.getString("sex").charAt(0));  
        user.setAge(rs.getInt("age"));  
  
        return user;  
    }  
}  
  
public List findAllByRowMapperResultReader() {  
    String sql = "SELECT * FROM USER";  
    return jdbcTemplate.query(sql, new RowMapperResultReader(new UserRowMapper()));  
} 
```

```JAVA
public User getUser(final String id) throws DataAccessException {  
    String sql = "SELECT * FROM USER WHERE user_id=?";  
    final Object[] params = new Object[] { id };  
    List list = jdbcTemplate.query(sql, params, new RowMapperResultReader(new UserRowMapper()));  
  
    return (User) list.get(0);  
} 
```
- org.springframework.jdbc.core.PreparedStatementCreator 返回预编译SQL   不能于Object[]一起用


```JAVA
public PreparedStatement createPreparedStatement(Connection con) throws SQLException {  
 return con.prepareStatement(sql);  
}  
```

```JAVA
template.update("insert into web_person values(?,?,?)",Object[]); 
```

```JAVA
template.update("insert into web_person values(?,?,?)",new PreparedStatementSetter(){ 匿名内部类 只能访问外部最终局部变量  
  
 public void setValues(PreparedStatement ps) throws SQLException {  
  ps.setInt(index++,3);  
}); 
````

```JAVA
public void setValues(PreparedStatement ps) throws SQLException {  
 ps.setInt(index++,3);  
}  
````

```JAVA
public Object mapRow(ResultSet rs, int arg1) throws SQLException {   int表当前行数  
  person.setId(rs.getInt("id"));  
}  
List template.query("select * from web_person where id=?",Object[],RowMapper);  
````

```JAVA
template.query("select * from web_person where id=?",Object[],new RowCallbackHandler(){  
 public void processRow(ResultSet rs) throws SQLException {  
  person.setId(rs.getInt("id"));  
});  
````