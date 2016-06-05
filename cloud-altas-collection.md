## 表结构 ##
t_bussiness_type:事件类型，比如“登陆”，“异常”，“自定义事件”等等
t_bussiness_type_table_field:与t_bussiness_type为多对一关系，表中字段用于维护某些关系，具体并不知道是什么。
t_bussiness_type_table:与t_bussiness_type为多对一关系，其中‘table_name’字段指明一个事件类型需要操作哪些表。比如Login事件，需要往t_collection_action和t_app_usage表写入数据。
t_app_usage:用户的每次LOGIN操作都会写入这张表
t_collection_action:用户的每次LOGIN操作都会写入这张表
t_app_usage与t_collection_action是一对一关系，前者的identifier字段与后者的buss_id一一对应。
调用collection工程接口会传入t_bussiness_type，通过它可以获得另外两个表的数据，得到要将数据写入的表。
从程序上看，这三个表之间是存在外键关系，但是我在数据库中没有看出，z道是先创建数据库中的表之后在运行程序导致的？
t_sqool_ins_fields:存放某些表的字段，存放方式，表名+字段名，所有字段名称在ins_fields中体现。
## 写入flume ##
`FlumeAdapterServiceImpl`类中方法`saveData()`是核心，许多逻辑都在该代码中实现。
`BaseAdapterService`中方法`dataGroupBybussinesType`将传入json格式参数按照`bussinessType`分组，输出内容：
```JAVA
{LOGIN=[com.nd.esp.cloudaltas.business.collect.domainmodel.request.CollectInfoRequestBean@1d317535]}
```
`MysqlTableStructMangerImpl`类中方法`getTableStructBean`获得表的结构，表名以及其他内容，输出内容如下：
```JAVA
{"tableName":"T_COLLECT_ACTION","metaData":{"bussiness_type":"java.lang.String","insert_time":"java.sql.Timestamp","buss_id":"java.lang.String","ext_properties":"java.lang.String","app_key":"java.lang.String","identifier":"java.lang.String"},"fieldNames":["bussiness_type","insert_time","buss_id","ext_properties","app_key","identifier"]}
```
`SqoolInsFieldsServiceImpl`类中方法`getFields`获得对应表的所有字段名称，实际上应该是读取t_sqool_ins_fields表中的内容，输出内容：
```JAVA
[identifier, app_key, buss_id, bussiness_type, ext_properties, insert_time]
```
`AdapterCommonServiceImpl`的`buildDynamicBeans()`方法返回结果：
```JAVA
{T_COLLECT_ACTION={"tableName":"T_COLLECT_ACTION","metaData":{},"rowSize":1,"fieldNames":[]}, T_APP_USAGE={"tableName":"T_APP_USAGE","metaData":{},"rowSize":1,"fieldNames":[]}}
```
返回的值中已经包含post进入的数据，例如：`[{createTime=2016-04-07T16:50:50.336+0800, session_id=ae45821d-5413-4921-84d2-ef22b07ef954, bussinessType=login, accountId=123456789, function_id=21, appVer=2016-04-07 16:33:33, account_id=123456789, bussiness_type=login, buss_id=b9d814ec-3b63-43fc-86ad-ba0e5e589c41, ext_properties=null, device_id=cyjcd08c754dd4c, app_key=sso, extProperties=null, sessionId=ae45821d-5413-4921-84d2-ef22b07ef954, create_time=2016-04-07T16:50:50.336+0800, functionId=21, identifier=b9d814ec-3b63-43fc-86ad-ba0e5e589c41, deviceId=cyjcd08c754dd4c, app_ver=2016-04-07 16:33:33}]` ，里边包含两套完全一样的数据，只是key值的格式不一致。这应该是东哥做的不够优雅的地方吧。
有了上边的数据就可以根据t_sqool_ins_fields表，取出需要的数据，拼接成字符串，将数据写入flume，通过`org.apache.flume.api.RpcClient`类将数据写入flume，之后flume会将数据写入hdfs。

`CollectInfoRequestBean`的`toMap()`方法转换数据库字段格式，例如post请求传入"{create_time:2016-04-28}"，则会被转换为"{createTime:2016-04-28}"，这是因为waf限制传入格式，由于需要，所以手工修改。
## 写入mysql ##
写入mysql的步骤与写入flume的步骤类似，但是相对简单，使用JdbcTemplate实现数据插入数据库。
