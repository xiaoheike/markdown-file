## [MySQL升级之timestamp的坑](https://louishust.github.io/mysql/2014/09/05/timestamp-bug)  ##
MySQL自5.6.4版本开始，对timestamp类型进行了扩展，允许精度达到微秒级。按理说这是一个好消息，but，公司内部一个软件升级数据库后，导致了一个P1故障，我们先看下这个相关的抽象出来的代码逻辑：
```SQL
Timestamp now = new Timestamp(System.currentTimeMillis());
template.update("insert into new_table(id,times) values(?,?)",
               1, now);
int i = template.queryForInt("select count(*) from new_table "
                             "where id=? and times=?", 1, now);
System.out.println(i);
```

上面的代码很简单，生成一个时间类型的变量，然后调用了spring的JdbcTemplate去做insert，然后根据这个时间去查询。但是，就这在查询的时候，5.5与5.6产生了差别，5.5可以查出来，5.6查不出来。因为这个原因导致了一个P1故障。身为DBA的我有必要好好研究下

### 原因分析 ###
根据manual，我们可以大概的猜测到原因。

5.5不支持高精度的timestamp类型，故不论是在存储还是比较的时候，都会将now变量转化为精度 为秒的类型，所以5.5中，即使是传入了带有微妙级别的timestamp变量，最终也会按照秒级去比较。 5.6支持高精度的timestamp类型，但是要在建表的时候指定，比如timestamp(6)，表示支持到”2014-09-09 10:00:00.123456″后面6位有效数字。但是由于建表语句没有指定精度，故存进表里面的now会被截断到秒级别，查询的时候是一个秒级别的列和一个微秒级的now进行比较，故查不出来。

所以，之所以出现P1故障，是因为我们迁移到5.6之后，建表语句没有变化，timestamp依然没有指定精度，但是程序里面用于插入和比较的列都是精确到微秒级别的，这就导致了插入的值被截断，查询查不出来。

### 解决方法 ###
事后和同事商量，既然5.5，5.6的表现有点区别，虽然是和程序有点关系，但是毕竟MySQL表现不友好，我们需要制定一个不区分版本的使用规范：
- 使用timestamp列时，用于插入和比较的值最多精确到秒
- 如果要使用更高的精度的时间，请使用BIGINT代替
当然，规范是死的，人是活的，如果你真的对自己的程序以及后端MySQL的行为非常清楚的话，我们无话可说，你可以使用更高精度的timestamp类型。

### 深入代码 ###
上面只是针对问题进行了原始的分析，下面要更深入的分析代码相关的东西。我们以插入为例，代码级别比较5.5与5.6的异同之处。对于JDBC程序来说，插入其实分为两种：常量插入，绑定插入。

**常量插入**
所谓的常量插入，类似 `insert into t1 values(’2014-10-01 10:00:00.1234′);`

对于这种插入，JDBC并不会对这个字符串的时间值做任何修改，而是会直接将这个语句发到服务器端。那么对于这种插入，5.5与5.6的区别就是，对这个字符串的值转换成timestamp类型的差别。

其实不论是5.5还是5.6，从字符串转化为内部时间类型的时候，使用了一样的函数str_to_datetime，这个函数会将string类型的时间转化为内部结构体MYSQL_TIME，这里会保留完整的精度，不会有任何精度损失。5.5与5.6的差别在于转化为MYSQL_TIME之后的处理有出入。

5.5之后会调用TIME_to_timestamp进行转化，这个函数如下：
```SQL
my_time_t TIME_to_timestamp(THD *thd, const MYSQL_TIME *t, my_bool *in_dst_time_gap)
{
  my_time_t timestamp;
 
  *in_dst_time_gap= 0;
  thd->time_zone_used= 1;
 
  timestamp= thd->variables.time_zone->TIME_to_gmt_sec(t, in_dst_time_gap);
  if (timestamp)
  {
    return timestamp;
  }
 
  /* If we are here we have range error. */
  return(0);
}
```
这个函数会调用TIME_to_gmt_sec，将MYSQL_TIME存储的值只精确到秒级。

5.6 会进行如下的转化：
```SQL
type_conversion_status
Field_temporal_with_date::store_internal_with_round(MYSQL_TIME *ltime,
                                                    int *warnings)
{
  if (my_datetime_round(ltime, dec, warnings))
  {
    reset();
    return time_warning_to_type_conversion_status(*warnings);
  }
  else
    return store_internal(ltime, warnings);
}
```
这里调用了my_datetime_round函数进行round，可以看到传入一个参数dec，这个参数就是用于描述timestamp的精度，即timestamp(x)中这个x的值。

同时还需要注意的是，这个round操作可能会产生进位，比如timestamp(3)，然后insert ’2014-12-12 12:00:00.1235′，其实存储的是’2014-12-12 12:00:00.124′

**绑定插入**
实际使用中更多的是绑定插入，绑定插入和常量插入是不一样的，常量插入JDBC驱动不做任何关于插入值的变化，直接把相应的insert语句发送到server。但是绑定插入，JDBC驱动会对传入的参数做手脚，这个也是我没想到的，驱动会对传入的参数做手脚，你敢相信？

绑定参数必然会有PreparedStatement，而PreparedStatement在初始化的时候，会根据服务器的版本初始化一个变量：serverSupportsFracSecs,这个变量表示服务器是否支持高精度的timestamp。也就是说对于5.6.4之前的版本，这个变量的值是false，5.6.4及以后的版本这个值是true。

那么我们来看下JDBC对于绑定参数的变化代码：
```SQL
if (this.serverSupportsFracSecs) {
     int nanos = x.getNanos();
 
     if (nanos != 0) {
         buf.append('.');
         buf.append(TimeUtil.formatNanos(nanos, this.serverSupportsFracSecs, true));
     }
 }
```
可以看出如果server支持高精度的timestamp，JDBC绑定参数的时候才会考虑到把高精度的数值传递过去，否则JDBC会自动屏蔽掉毫秒，微秒级别。

## [MySQL 对小数部分的处理](https://dev.mysql.com/doc/refman/5.7/en/fractional-seconds.html) ##
MySQL 5.7 has fractional seconds support for TIME, DATETIME, and TIMESTAMP values, with up to microseconds (6 digits) precision:

- To define a column that includes a fractional seconds part, use the syntax type_name(fsp), where type_name is TIME, DATETIME, or TIMESTAMP, and fsp is the fractional seconds precision. For example:
```SQL
CREATE TABLE t1 (t TIME(3), dt DATETIME(6));
```
The fsp value, if given, must be in the range 0 to 6. A value of 0 signifies that there is no fractional part. If omitted, the default precision is 0. (This differs from the standard SQL default of 6, for compatibility with previous MySQL versions.)

- Inserting a TIME, DATE, or TIMESTAMP value with a fractional seconds part into a column of the same type but having fewer fractional digits results in rounding, as shown in this example:
```SQL
mysql> CREATE TABLE fractest( c1 TIME(2), c2 DATETIME(2), c3 TIMESTAMP(2) );
Query OK, 0 rows affected (0.33 sec)

mysql> INSERT INTO fractest VALUES
     > ('17:51:04.777', '2014-09-08 17:51:04.777', '2014-09-08 17:51:04.777');
Query OK, 1 row affected (0.03 sec)

mysql> SELECT * FROM fractest;
+-------------+------------------------+------------------------+
| c1          | c2                     | c3                     |
+-------------+------------------------+------------------------+
| 17:51:04.78 | 2014-09-08 17:51:04.78 | 2014-09-08 17:51:04.78 |
+-------------+------------------------+------------------------+
1 row in set (0.00 sec)
```
No warning or error is given when such rounding occurs. This behavior follows the SQL standard, and is not affected by the server's sql_mode setting.

- Functions that take temporal arguments accept values with fractional seconds. Return values from temporal functions include fractional seconds as appropriate. For example, NOW() with no argument returns the current date and time with no fractional part, but takes an optional argument from 0 to 6 to specify that the return value includes a fractional seconds part of that many digits.

- Syntax for temporal literals produces temporal values: DATE 'str', TIME 'str', and TIMESTAMP 'str', and the ODBC-syntax equivalents. The resulting value includes a trailing fractional seconds part if specified. Previously, the temporal type keyword was ignored and these constructs produced the string value. See Standard SQL and ODBC Date and Time Literals

## MySQL5.7 timestamp ##
实验，跟新一条记录，原先的时间为 `2017-07-22 16:29:38`，精度到 秒。更新该时间为 `2017-07-22 16:29:38.123`，更新之后的结果为：`2017-07-22 16:29:38`。更新该时间为 `2017-07-22 17:29:38.5`，则更新之后的时间为 `2017-07-22 17:29:39`。
也就是说按照 四舍五入。

