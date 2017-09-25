## pom.xml 文件配置 ##
引入 `mybatis generator`
```xml
<properties>
  <mysql.connector.version>5.1.44</mysql.connector.version>
  <mybatis.generator.version>1.3.5</mybatis.generator.version>
  <mybatis.spring.version>1.3.1</mybatis.spring.version>
  <mybatis.version>3.4.4</mybatis.version>
</properties>

<build>
  <plugins>
   <plugin>
     <groupId>org.mybatis.generator</groupId>
     <artifactId>mybatis-generator-maven-plugin</artifactId>
     <version>${mybatis.generator.version}</version>
     <configuration>
       <configurationFile>src/main/resources/generatorConfig.xml</configurationFile>
       <verbose>true</verbose>
       <overwrite>true</overwrite>
     </configuration>
     <dependencies>
       <dependency>
         <groupId>mysql</groupId>
         <artifactId>mysql-connector-java</artifactId>
         <version>${mysql.connector.version}</version>
       </dependency>
       <dependency>
         <groupId>org.mybatis.generator</groupId>
         <artifactId>mybatis-generator-core</artifactId>
         <version>${mybatis.generator.version}</version>
       </dependency>
       <dependency>
         <groupId>com.github.oceanc</groupId>
         <artifactId>mybatis3-generator-plugin</artifactId>
         <version>0.4.0</version>
       </dependency>
     </dependencies>
   </plugin>
  </plugins>
 <resources>
   <resource>
     <!--src/main/java 下的 xml 文件打包时也加入-->
     <directory>src/main/java</directory>
     <includes>
       <include>**/*.xml</include>
     </includes>
   </resource>
   <resource>
     <directory>src/main/resources</directory>
     <filtering>true</filtering>
     <includes>
       <include>**/*.*</include>
     </includes>
     <excludes>
       <!--<exclude>test/**/*.*</exclude> -->
     </excludes>
   </resource>
 </resources>
</build>
```
1. `<configurationFile>src/main/resources/generatorConfig.xml</configurationFile>` 自动生成代码的核心配置文件 `generatorConfig.xml` 的路径
2. `mysql-connector-java` 生成哪种数据库的代码，不可省略
3. `com.github.oceanc` 引入第三方的 `jar`，能够生成常用的查询语法
4. `resources` 标签配置是为了将 `mybatis 语法 xml 文件` 打包进 `war` 包，缺少 `xml` 文件代码是无法执行的
5. `org.mybatis.generator` 自动生成可执行代码的核心 `jar`，不可缺少

## [org.mybatis.generator 自带生成代码插件](http://www.mybatis.org/generator/reference/plugins.html) ##
- org.mybatis.generator.plugins.CachePlugin
二级缓存相关，需要更深入了解一下
- org.mybatis.generator.plugins.CaseInsensitiveLikePlugin
对字符串匹配生成大小写敏感的方法
- org.mybatis.generator.plugins.EqualsHashCodePlugin
重写 `model` 中 `equals` 和 `hashCode` 方法，这对比两个对象的值是否相等很在意义。这个重写可以通过 `Intellj` 自带的帮助方法解决，`ALT+Insert`-->`equals() and hashCode()`。各处统一使用相同的生成方法即可
- org.mybatis.generator.plugins.FluentBuilderMethodsPlugin
生成 `withXxx()` 方法，可以简洁的赋值 `new MyDomain().withFoo("Test").withBar(4711);`。这个功能可以通过 `Intellj` 的插件 `InnerBuilder` 实现，使用 `Builder` 模式。
- org.mybatis.generator.plugins.MapperConfigPlugin
生成 `MyBatis 3.x` 框架使用的配置文件。感觉没有什么用处，生成的配置文件也没有生成什么有用的东西
- org.mybatis.generator.plugins.RenameExampleClassPlugin
`mybatis` 默认生成的查询类是以 `Example` 结尾，往往都使用如下配置改成 `Criteria` 结尾
```xml
<plugin type="org.mybatis.generator.plugins.RenameExampleClassPlugin">
    <property name="searchString" value="Example$" />
    <property name="replaceString" value="Criteria" />
</plugin>
```
- org.mybatis.generator.plugins.RowBoundsPlugin
分页
- org.mybatis.generator.plugins.SerializablePlugin
继承序列化
- org.mybatis.generator.plugins.SqlMapConfigPlugin
生成 `iBATIS 2.x` 框架使用的配置文件。感觉没有什么用处，生成的配置文件也没有生成什么有用的东西
```xml
<plugin type = "org.mybatis.generator.plugins.SqlMapConfigPlugin">
    <property name="fileName" value="test.xml"/>
    <property name="targetPackage" value="com.nd.mybatis"/>
    <property name="targetProject" value="src/main/java"/>
</plugin>
```
- org.mybatis.generator.plugins.ToStringPlugin
重写 `model` 中 `toString` 方法。这个可以通过 `Intellj` 完成，`ALT+insert`--> `toString()`
- org.mybatis.generator.plugins.VirtualPrimaryKeyPlugin
如果表没有主键，`mybatis` 有部分方法不会生成，配置几个虚拟的主键，即使在数据库中并不是主键也可以配置。配置方案
```xml
<table tableName="foo">
    <property name="virtualKeyColumns" value="ID1, ID2" />
</table>
```

## [com.github.oceanc 支持生成代码插件](https://github.com/oceanc/mybatis3-generator-plugins) ##
**受sql dialect的限制，多数plugin目前仅支持 Mysql 5.x**
### Download
在 Maven Central 上最新的发布版本是:
```xml
<dependency>
    <groupId>com.github.oceanc</groupId>
    <artifactId>mybatis3-generator-plugin</artifactId>
    <version>0.4.0</version>
</dependency>
```

mybatis3-generator-plugins目前提供了如下可用插件：
- com.github.oceanc.mybatis3.generator.plugin.BatchInsertPlugin
- com.github.oceanc.mybatis3.generator.plugin.JacksonAnnotationPlugin
- com.github.oceanc.mybatis3.generator.plugin.JacksonToJsonPlugin
- com.github.oceanc.mybatis3.generator.plugin.LombokAnnotationPlugin
- com.github.oceanc.mybatis3.generator.plugin.MinMaxPlugin
- com.github.oceanc.mybatis3.generator.plugin.OptimisticLockAutoIncreasePlugin
- com.github.oceanc.mybatis3.generator.plugin.PaginationPlugin
- com.github.oceanc.mybatis3.generator.plugin.SliceTablePlugin
- com.github.oceanc.mybatis3.generator.plugin.SumSelectivePlugin
- com.github.oceanc.mybatis3.generator.plugin.UpdateSqlTextOfUpdateSelectivePlugin
- com.github.oceanc.mybatis3.generator.plugin.WhereSqlTextPlugin

### 使用
在[MyBatis GeneratorXML Configuration File](http://www.mybatis.org/generator/configreference/xmlconfig.html)中添加你需要用到的`<plugin>`元素：

```xml
<context id="MysqlTables" targetRuntime="MyBatis3">
  <plugin type = "com.github.oceanc.mybatis3.generator.plugin.BatchInsertPlugin" />
  <plugin type = "com.github.oceanc.mybatis3.generator.plugin.JacksonAnnotationPlugin" />
  <plugin type = "com.github.oceanc.mybatis3.generator.plugin.JacksonToJsonPlugin" />
  <plugin type = "com.github.oceanc.mybatis3.generator.plugin.LombokAnnotationPlugin" />
  <plugin type = "com.github.oceanc.mybatis3.generator.plugin.MinMaxPlugin" />
  <plugin type = "com.github.oceanc.mybatis3.generator.plugin.OptimisticLockAutoIncreasePlugin" />
  <plugin type = "com.github.oceanc.mybatis3.generator.plugin.PaginationPlugin" />
  <plugin type = "com.github.oceanc.mybatis3.generator.plugin.SliceTablePlugin" />
  <plugin type = "com.github.oceanc.mybatis3.generator.plugin.SumSelectivePlugin" />
  <plugin type = "com.github.oceanc.mybatis3.generator.plugin.UpdateSqlTextOfUpdateSelectivePlugin" />
  <plugin type = "com.github.oceanc.mybatis3.generator.plugin.WhereSqlTextPlugin" />
</context>
```

为了举例，假设我们创建一张简单的账户表，并命名为`Account`。DDL如下：
```sql
CREATE TABLE `Account` (
  `id`            bigint(16)    NOT NULL,
  `create_time`   timestamp     NOT NULL,
  `name`          varchar(64)   DEFAULT NULL,
  `age`           tinyint(3)    DEFAULT NULL,
  `version`       int(11)       NOT NULL,
  PRIMARY KEY (`id`)
)ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin;
```

**如果使用了 `SliceTablePlugin`，为了其产生的作用始终有效，推荐将其放置在自定义 `<plugin>` 声明的第一条，如上例所示。这么做的原因是，其他 `plugin（BatchInsertPlugin、MinMaxPlugin、和SumSelectivePlugin）` 会通过检测 `SliceTablePlugin` 是否应用，来适配其效果。**

### SliceTablePlugin ###
当数据库单表数据量过大时，可以通过水平拆分把单表分解为多表。例如，若 `Account` 表中的预期数据量过大时（也许5、6亿），可以把一张 `Account` 表分解为多张 `Account` 表。**SliceTablePlugin并不会自动创建和执行拆分后的DDL，因此必须手工创建DDL并按下述约定修改表名。**

 `SliceTablePlugin` 目前支持两种分表命名方式：
* 对指定列取模。拆分后的表名为 `原表名_N`。如：Account_0、Account_1、Account_3 ......
* 对指定的时间类型列，按单自然月拆分。拆分后的表名为 `原表名_yyyyMM`。如：Account_201501、Account_201502 ...... Account_201512

#### Xml config ####
1. 利用取模  
假如通过 `Account` 表的 `id` 字段做分表，计划拆分为 `97` 张表。在[MyBatis GeneratorXML Configuration File](http://www.mybatis.org/generator/configreference/xmlconfig.html) 的 `<table>` 元素中添加两个 `<property>`
```xml
<table tableName="Account_0" domainObjectName="Account">
    <property name="sliceMod" value="97"/>
    <property name="sliceColumn" value="id"/>
</table>
```
* `sliceMod`指按取模方式分表，`97`是取模的模数
* `sliceColumn`指取模所使用的列，`id`是具体列名

2. 利用自然月  
假如通过 `Account` 表的 `create_time` 字段做拆分:
```xml
<table tableName="Account_0" domainObjectName="Account">
    <property name="sliceMonth" value="1"/>
    <property name="sliceColumn" value="create_time"/>
</table>
```
* `sliceMonth` 指按自然月分表，`1` 指按单个自然月。**Note：目前只支持按单月分表，此处的值 1 无实际意义**
* `sliceColumn` 指时间类型的列，`create_time` 是具体列名

#### Java sample ####
- insert
```java
AccountMapper mapper = ...
Account record = new Account();
record.setAge(33);
record.setId(101);
record.setCreateTime(new Date());
mapper.insert(record);
// or mapper.insertSelective(record)
```
* 通过取模分表时，必须调用 `setId` 并传入合适的参数
* 通过自然月分表时，必须调用 `setCreateTime` 并传入合适的参数

- read
```java
AccountMapper mapper = ...
AccountExample example = new AccountExample();
example.partitionFactorId(id).createCriteria().andAgeEqualTo(33);
// or example.partitionFactorCreateTime(new Date()).createCriteria().andAgeEqualTo(33);
List<Account> as = mapper.selectByExample(example);
```
* 通过取模分表时，`partitionFactorId` 方法表示分表因子是 `Id` 字段，必须调用该方法并传入合适的参数
* 通过自然月分表时，`partitionFactorCreateTime` 方法表示分表因子是 `createTime` 字段，必须调用该方法并传入合适的参数

- update
```java
AccountMapper mapper = ...
Account record = new Account();
record.setAge(33);
AccountExample example = new AccountExample();
example.partitionFactorId(id).createCriteria().andAgeEqualTo(33);
// or example.partitionFactorCreateTime(new Date()).createCriteria().andAgeEqualTo(33);
mapper.updateByExampleSelective(record, example);
```
上例的用法和read操作一样，在example对象上必须调用 `partitionFactorId` 或 `partitionFactorCreateTime` 方法。  
除此之外，还可以用如下方式进行update：
```java
AccountMapper mapper = ...
Account record = new Account();
record.setCreateTime(new Date());
record.setId(101);
// or record.setCreateTime(new Date());
AccountExample example = new AccountExample();
example.createCriteria().andAgeEqualTo(33);
mapper.updateByExampleSelective(record, example);
```
由于在record对象调用了`setId`或`setCreateTime`，就无须在example对象指定分表因子。

- delete
```java
AccountMapper mapper = ...
AccountExample example = new AccountExample();
example.partitionFactorId(id).createCriteria().andAgeEqualTo(33);
// or example.partitionFactorCreateTime(new Date()).createCriteria().andVersionEqualTo(0);
mapper.deleteByExample(example);
```
* 通过取模分表时，`partitionFactorId` 方法表示分表因子是Id字段，必须调用该方法并传入合适的参数
* 通过自然月分表时，`partitionFactorCreateTime` 方法表示分表因子是createTime字段，必须调用该方法并传入合适的参数

- other
当无法获得分表因子的值时、或者确定所操作的表名时，可以通过:
* `record.setTableNameSuffix(...)` 取代`record.setId(...)` 或 `record.setId(...)`
* `example.setTableNameSuffix(...) `取代 `example.partitionFactorId(...)`

**WARNING**：由于 `setTableNameSuffix` 的参数是 `String` 类型，在 `Mybatis3` 的 `mapper xml` 中生成 `${}` 变量，这种变量不会做 `sql` 转义，而直接嵌入到 `sql` 语句中。如果以用户输入作为 `setTableNameSuffix` 的参数，会导致潜在的 `SQL Injection` 攻击，需谨慎使用。

### BatchInsertPlugin ###
该 `plugin` 是为了增加批量 `insert` 的功能。特别适用于初始化数据、或迁移数据。

#### Java sample ####
```java
AccountMapper mapper = ...
List<Account> as = new ArrayList<>();
as.add(...)
mapper.batchInsert(as);
```
如果使用了 `SliceTablePlugin`，则需要对 `List` 中每一个 `Account` 实例设置分表因子：
```java
AccountMapper mapper = ...
List<Account> as = new ArrayList<>();
for (Account account : accounts) {
    account.setId(...);
    // or account.setCreateTime(...);
}
mapper.batchInsert(as);
```
* 通过取模分表时，必须调用`setId`并传入合适的参数
* 通过自然月分表时，必须调用`setCreateTime`并传入合适的参数

### JacksonAnnotationPlugin ###
该 `plugin` 是为生成的 `model` 类添加[jackson](https://github.com/FasterXML/jackson)的 `@JsonProperty`、`@JsonIgnore`、 和 `@JsonFormat` 注解。若使用此插件，需要额外依赖`jackson 2.5 +`。(这个插件没有什么必要啊，手工添加更加方便)

#### Xml config ####
在[MyBatis GeneratorXML Configuration File](http://www.mybatis.org/generator/configreference/xmlconfig.html)的 `<table>` 元素中添加四个 `<property>`（可选的）：

```xml
<table tableName="Account_0" domainObjectName="Account">
    <property name="jacksonColumns" value="name,age"/>
    <property name="jacksonProperties" value="nickName,realAge"/>
    <property name="jacksonFormats" value="create_time@yyyy-MM-dd HH:mm:ss"/>
    <property name="jacksonIgnores" value="id,version"/>
</table>
```
* `jacksonColumns` 指需要添加 `@JsonProperty` 注解的列，由 `,` 分割的列名组成。该 `<property>` 必须和 `jacksonProperties` 成对出现
* `jacksonProperties` 指 `@JsonProperty` 注解所需的参数值，由 `,` 分割，这些值的数量和顺序必须和 `jacksonColumns` 的值一一对应。该 `<property>` 必须和 `jacksonColumns` 成对出现
* `jacksonFormats` 指需要添加 `@JsonFormat` 注解的列，值由 `,` 分割的键值对组成，键值对由 `@` 分割，键为列名，值为 `@JsonFormat` 所需参数
* `jacksonIgnores` 指需要添加 `@JsonIgnore` 注解的列，值由 `,` 分割的列名组成

#### Java sample ####
```java
public class Account implements Serializable {
    @JsonIgnore
    private Long id;
    @JsonProperty("nickName")
    private String name;
    @JsonProperty("realAge")
    private Integer age;
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    private Date createTime;
    @JsonIgnore
    private Long version;
}
```

#### JacksonToJsonPlugin ####
该 `plugin` 是为生成的 `model` 类添加 `toJson` 方法。`toJson` 方法的实现依赖于[jackson](https://github.com/FasterXML/jackson)。若使用此插件，需要额外依赖`jackson 2.5 +`。（这个插件没有必要使用，并且会有性能的问题，因为生成的 `toJson` 方法每次被调用都重新 `new ObjectMapper`。`ObjectMapper` 是线程安全的，完全可以做一个全局的变量）

#### Java sample
```java
public class Account implements Serializable {
    public String toJson() throws IOException {
        ... ...
    }
}
```

### LombokAnnotationPlugin ###
该 `plugin` 是为生成的 `model` 类添加[lombok](https://projectlombok.org/)注解，避免了 `java bean` 中繁琐的 `setter` 和 `getter`。生成的代码看起来也更干净、紧凑。若使用此插件，需要额外依赖 `lombok`。（这个插件可用，可不用，因为手工添加几个注解也是简单的）
```xml
<dependency>
  <groupId>org.projectlombok</groupId>
  <artifactId>lombok</artifactId>
  <version>1.16.8</version>
</dependency>
```

#### Java sample ####
```java
@Data
public class Account implements Serializable {
    private Long id;
    ... ...
}
```

### MinMaxPlugin ###
#### Xml config ####
在[MyBatis GeneratorXML Configuration File](http://www.mybatis.org/generator/configreference/xmlconfig.html) 的 `<table>` 元素中添加两个 `<property>`（可选的）。（这个支持不好，生成的方法名称不友好，以逗号分隔没有起到作用）
```xml
<table tableName="Account_0" domainObjectName="Account">
    <property name="minColumns" value="id,version"/>
    <property name="maxColumns" value="id,version"/>
</table>
```
* `minColumns` 是可进行min操作的列，值由`,`分割的列名组成
* `maxColumns` 是可进行max操作的列，值由`,`分割的列名组成

#### Java sample ####
```java
AccountMapper mapper = ...
AccountExample example = new AccountExample();
example.createCriteria().andAgeEqualTo(33);
long min = mapper.minIdByExample(example);
long max = mapper.maxIdByExample(example);
```
如果使用了 `SliceTablePlugin`，别忘了对分表因子赋值：`example.partitionFactorCreateTime(...)`

#### OptimisticLockAutoIncreasePlugin ####
在处理并发写操作时，如果数据竞争较低，通常会采用乐观锁，避免并发问题的同时获得较好的性能。实现乐观锁的一般方式是在数据中添加版本信息，就像 `Account` 表中的 `version` 列。当写操作成功时，版本信息也相应递增。该 `plugin` 就是解决 `version` 自动递增问题的，可以避免手动对 `version+1`。(从生成的文件中没有发现 `version` 增加的地方，不知道在哪里处理的，也没有做实验验证是否可行)

#### Xml config ####
在[MyBatis GeneratorXML Configuration File](http://www.mybatis.org/generator/configreference/xmlconfig.html)的 `<table>` 元素中添加一个 `<property>`
```xml
<table tableName="Account_0" domainObjectName="Account">
    <property name="optimisticLockColumn" value="version"/>
</table>
```
* `optimisticLockColumn` 指具有版本信息语义的列，`version` 是具体的列名

#### Java sample ####
```java
AccountMapper mapper = ...
Account record = new Account();
record.setName("tom");
// record.setVersion(1) 无须手工对version进行赋值
AccountExample example = new AccountExample();
example.createCriteria().andAgeEqualTo(33);
mapper.updateByExampleSelective(record, example);
```
如果使用了 `SliceTablePlugin`，别忘了对分表因子赋值：`example.partitionFactorId(...)`

### PaginationPlugin ###
这个与 `org.mybatis.generator.plugins.RowBoundsPlugin` 的功能是类似的
#### Java sample ####
```java
AccountMapper mapper = ...
AccountExample example = new AccountExample();
example.page(0, 2).createCriteria().andAgeEqualTo(33);
List<Account> as = mapper.selectByExample(example);
```
如果使用了 `SliceTablePlugin`，别忘了对分表因子赋值：`example.partitionFactorCreateTime(...)`

### SumSelectivePlugin ###
#### Java sample ####
```java
AccountMapper mapper = ...
AccountExample example = new AccountExample();
example.sumAge().createCriteria().andVersionEqualTo(0);
long sum = mapper.sumByExample(example);
```
如果使用了 `SliceTablePlugin` ，别忘了对分表因子赋值：`example.partitionFactorCreateTime(...)`

### UpdateSqlTextOfUpdateSelectivePlugin ###
#### Java sample ####
```java
AccountMapper mapper = ...
Account record = new Account();
record.setUpdateSql("version = version + 1")
record.setAge(33);
AccountExample example = new AccountExample();
example.createCriteria().andAgeEqualTo(22);
mapper.updateByExampleSelective(record.setAge, example);
```
如果使用了 `SliceTablePlugin`，别忘了对分表因子赋值：`example.partitionFactorCreateTime(...)`  
**WARNING**：由于 `setUpdateSql` 的参数是 `String` 类型，在 `Mybatis3` 的 `mapper xml` 中生成 `${}` 变量，这种变量不会做 `sql` 转义，而直接嵌入到 `sql` 语句中。如果以用户输入作为 `setUpdateSql` 的参数，会导致潜在的 `SQL Injection` 攻击，需谨慎使用。

### WhereSqlTextPlugin ###
#### Java sample ####
```java
AccountMapper mapper = ...
int v = 1;
AccountExample example = new AccountExample();
example.createCriteria().andAgeEqualTo(33).addConditionSql("version =" + v + " + 1");
List<Account> as = mapper.selectByExample(example);
```
如果使用了 `SliceTablePlugin`，别忘了对分表因子赋值：`example.partitionFactorCreateTime(...)`  
**WARNING**：由于 `setUpdateSql` 的参数是 `String` 类型，在 `Mybatis3` 的 `mapper xml` 中生成 `${}` 变量，这种变量不会做 `sql` 转义，而直接嵌入到 `sql` 语句中。如果以用户输入作为 `setUpdateSql` 的参数，会导致潜在的 `SQL Injection` 攻击，需谨慎使用。

本篇博客主要来自：[https://github.com/oceanc/mybatis3-generator-plugins](https://github.com/oceanc/mybatis3-generator-plugins)，其他部分内容来自官网：[http://www.mybatis.org/generator/reference/plugins.html](http://www.mybatis.org/generator/reference/plugins.html)