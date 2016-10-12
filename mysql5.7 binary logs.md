## [开启 binary logs 功能](http://stackoverflow.com/questions/11445678/binary-log-error-in-mysql) ##
在 mysql 配置文件中配置 `log-bin`，重启 mysql
my.cnf (on Linux/unix) or my.ini (on Windows) 例子:
```XML
[client]
...

[mysqld]
...
log-bin=mysql-bin  (log_bin=/var/mydb/bin-log，指定 log 的路径，以及名称前缀)
---
```
一旦重启，Mysql 会自动创建新的二进制文件。您也不妨看看下面的[变量](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_expire_logs_days)：
```XML
server-id        = 1
expire_logs_days = 4 (自动删除 log 的天数，缺省值为 0，即不自动删除)
sync_binlog      = 1
```
详细信息可以阅读 [MySQL documentation](http://dev.mysql.com/doc/refman/5.1/en/server-system-variables.html)，如果你使用主从库（使用二进制文件的主要理由），请查阅[Replication configuration checklist](http://code.openark.org/blog/mysql/replication-configuration-checklist)
## 查看 binary logs ##
登陆 MySQL 之后执行如下语句：
```SQL
SHOW BINARY LOGS
等价
SHOW MASTER LOGS
```
返回值：
```XML
mysql> show master logs;
+------------------+-----------+
| Log_name         | File_size |
+------------------+-----------+
| mysql-bin.000001 |        98 |
+------------------+-----------+
1 row in set (0.00 sec)
```
## [手动删除 binary log](https://dev.mysql.com/doc/refman/5.7/en/purge-binary-logs.html) ##
```XML
PURGE { BINARY | MASTER } LOGS
    { TO 'log_name' | BEFORE datetime_expr }
```
例子：
```XML
PURGE BINARY LOGS TO 'mysql-bin.010';
PURGE BINARY LOGS BEFORE '2008-04-02 22:46:26';
```
datetime_expr 参数格式为 `YYYY-MM-DD hh:mm:ss`。
上述语法，当从库正在同步时，也可以安全运行。你不必要关闭从库。如果你正在删除一个从库正在同步的 log，上述语句将不会做任何操作。MySQL 5.7.2 以及之后版本将会报错。然而，如果你删除了一个从库没有同步的 log，那么从库将无法与主库保持数据一致。
手动安全删除日志的步骤：
- 在每一个从库的 MySQL 上运行 `SHOW SLAVE STATUS`，检验从库没有从主库读取日志
- 使用命令 `SHOW BINARY LOGS`，查看主库上的 binary log 文件
- 找出在从库中时间最早的 log 文件，这是我们要删除的目标文件。如果所有从库都对同一个 log 与主库保持同步，那么那个日志就是我们要删除的目标文件
- 删除之前，备份 log。（这一步是可选的，但是建议你这么做）
- 删除目标文件之前的 log

可以通过设置 `expire_logs_days` 参数，自动删除 log。
在调用上述删除语句之前，log 已经被删除，比如 linux 中使用 `rm` 命令，那么该语句将会报错。
## binary log 格式 ##
binary log 会记录所有与数据修改相关的操作，查询不会被记录哦。
比如我有如下的一些操作：
```SQL
1. drop table student_information;
2. drop table student;
3. CREATE TABLE `student` (
     `id` int(11) NOT NULL AUTO_INCREMENT,
     `name` char(20) NOT NULL,
     `age` tinyint(2) NOT NULL DEFAULT '0',
     PRIMARY KEY (`id`),
     KEY `index_name` (`name`)
) ENGINE=InnoDB AUTO_INCREMENT=8 DEFAULT CHARSET=utf8;
4. insert student values(1,'zhangsan',20);
5. insert student values(1,'zhangsan',20);
6. insert student values(3,'wangwu',22);
```
通过命令将 binary log 转为 SQL 脚本：
```SQL
mysqlbinlog mysql-bin.000001 > my.sql
```
binary log 存储内容为：
```SQL
SET TIMESTAMP=1473842609/*!*/;
SET @@session.foreign_key_checks=1, @@session.sql_auto_is_null=1, @@session.unique_checks=1/*!*/;
SET @@session.sql_mode=1344274432/*!*/;
/*!\C latin1 *//*!*/;
SET @@session.character_set_client=8,@@session.collation_connection=8,@@session.collation_server=8/*!*/;
drop table student_information
/*!*/;
# at 193
#160914 16:43:35 server id 1  end_log_pos 276 	Query	thread_id=1	exec_time=0	error_code=0
SET TIMESTAMP=1473842615/*!*/;
drop table student
/*!*/;
# at 276
#160914 16:43:57 server id 1  end_log_pos 578 	Query	thread_id=1	exec_time=0	error_code=0
SET TIMESTAMP=1473842637/*!*/;
CREATE TABLE `student` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` char(20) NOT NULL,
  `age` tinyint(2) NOT NULL DEFAULT '0',
  PRIMARY KEY (`id`),
  KEY `index_name` (`name`)
) ENGINE=InnoDB AUTO_INCREMENT=8 DEFAULT CHARSET=utf8
/*!*/;
# at 578
#160914 16:44:11 server id 1  end_log_pos 648 	Query	thread_id=1	exec_time=0	error_code=0
SET TIMESTAMP=1473842651/*!*/;
BEGIN
/*!*/;
# at 648
#160914 16:44:11 server id 1  end_log_pos 751 	Query	thread_id=1	exec_time=0	error_code=0
SET TIMESTAMP=1473842651/*!*/;
insert student values(1,'zhangsan',20)
/*!*/;
# at 751
#160914 16:44:11 server id 1  end_log_pos 778 	Xid = 29
COMMIT/*!*/;
# at 778
#160914 16:44:19 server id 1  end_log_pos 848 	Query	thread_id=1	exec_time=0	error_code=0
SET TIMESTAMP=1473842659/*!*/;
BEGIN
/*!*/;
# at 848
#160914 16:44:19 server id 1  end_log_pos 947 	Query	thread_id=1	exec_time=0	error_code=0
SET TIMESTAMP=1473842659/*!*/;
insert student values(2,'lisi',21)
/*!*/;
# at 947
#160914 16:44:19 server id 1  end_log_pos 974 	Xid = 30
COMMIT/*!*/;
# at 974
#160914 16:44:25 server id 1  end_log_pos 1044 	Query	thread_id=1	exec_time=0	error_code=0
SET TIMESTAMP=1473842665/*!*/;
BEGIN
/*!*/;
# at 1044
#160914 16:44:25 server id 1  end_log_pos 1145 	Query	thread_id=1	exec_time=0	error_code=0
SET TIMESTAMP=1473842665/*!*/;
insert student values(3,'wangwu',22)
/*!*/;
# at 1145
#160914 16:44:25 server id 1  end_log_pos 1172 	Xid = 31
COMMIT/*!*/;
# at 1172
#160914 17:31:33 server id 1  end_log_pos 1242 	Query	thread_id=1	exec_time=0	error_code=0
SET TIMESTAMP=1473845493/*!*/;
BEGIN
/*!*/;
# at 1242
#160914 17:31:33 server id 1  end_log_pos 1348 	Query	thread_id=1	exec_time=0	error_code=0
SET TIMESTAMP=1473845493/*!*/;
update student set age = 100 where id = 1
/*!*/;
# at 1348
#160914 17:31:33 server id 1  end_log_pos 1375 	Xid = 33
COMMIT/*!*/;
DELIMITER ;
# End of log file
ROLLBACK /* added by mysqlbinlog */;
/*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*/;
```
从语法上可以看出，如果是 `update` 操作恢复的概率相对要麻烦很多，需要对比操作。 `SET TIMESTAMP` 是操作的时间戳，这个相当有用，这样就允许我们通过时间来确定重做的范围。`delete` 以及 `update` 都被放置在事务里边了，但是只有当整个 binary log 执行完成才算成功，任何一条语法的异常都会导致事务回滚，重做失败。
## [使用 binary log 增量恢复数据](http://dev.mysql.com/doc/refman/5.7/en/point-in-time-recovery.html) ##
直接重做 binary log 中的操作：
```SHELL
mysqlbinlog mysql-bin.000001 | mysql -u root -p
```
执行过程中发生异常就被被终止，所以在重做之前需要自己处理该文件，使得重做的动作是自己想要的。比如从上述 binary log 中有 `drop table student_information` 操作，而此时的数据库中已经不存在该表，所以重做该步骤就会抛出异常信息。`ERROR 1051 (42S02) at line 16: Unknown table 'student_information'`

逐行查看 binary log 中内容：
```SHELL
mysqlbinlog mysql-bin.000001 | more
```

转换 binary log 为 SQL 脚本： 
``` SHELL
mysqlbinlog mysql-bin.000001 > my.sql
```

重做 SQL 脚本：
```SHELL
mysql -u root -p < my.sql
```

如果你有多个 binary log 文件需要被执行，安全的方式是：将所有的 binary log 一次性执行。不安全方法的示例：
```SHELL
mysqlbinlog binlog.000001 | mysql -u root -p # DANGER!!
mysqlbinlog binlog.000002 | mysql -u root -p # DANGER!!
```
使用两个不同连接处理 binary log 可能导致问题，有可能会发生如下情况：第一个 binary log 包含语法 `CREATE TEMPOARY TEBLE` 而第二个 binary log 使用到该临时表。当第一个 binary log 执行完成将会删除临时表，那么第二个 binary log 需要使用到该临时表的语句将报错。

在一个连接中完成 binary logs 的处理，例子如下：
```SHELL
mysqlbinlog binlog.000001 binlog.000002 | mysql -u root -p
```
另外一种方法，将 binary logs 合并为一个 SQL 脚本：
```SHELL
mysqlbinlog binlog.000001 >  /tmp/statements.sql
mysqlbinlog binlog.000002 >> /tmp/statements.sql
mysql -u root -p -e "source /tmp/statements.sql"
```

When writing to a dump file while reading back from a binary log containing GTIDs (see Section 18.1.3, “Replication with Global Transaction Identifiers”), use the --skip-gtids option with mysqlbinlog, like this:
```SHELL
mysqlbinlog --skip-gtids binlog.000001 >  /tmp/dump.sql
mysqlbinlog --skip-gtids binlog.000002 >> /tmp/dump.sql
mysql -u root -p -e "source /tmp/dump.sql"
```

