# titan 在存在数据的属性下创建索引 #
## 起因 ##
这是一个遗留的问题，因为一开始时苗哥并没有解决，因为功能需要先完成，所以就留到现在解决。因为后续我们需要解决性能问题，而索引往往是优化需要使用到的，当然我们可以在导入数据之前就创建索引，但是导数据的时间太长了，少则 2 天，多则 1 周。性能优化过程我们需要对一些属性做建立与删除索引的操作。
## 遇到的问题 ##
感觉建立索引也是很简单的事情，因为官网上有两章专门介绍如何建立索引，并且也已经提及在存在数据的属性下建立索引。参考官方文档：[Indexing for better Performance](http://s3.thinkaurelius.com/docs/titan/1.0.0/indexes.html) 以及 [Reindexing Process](http://s3.thinkaurelius.com/docs/titan/current/reindex.html#_illegalargumentexception_when_starting_job)
我根据文档上边进行操作，主要的命令在于：
```JAVA
import com.thinkaurelius.titan.graphdb.database.management.ManagementSystem

// Load some data from a file without any predefined schema
graph = TitanFactory.open('conf/titan-berkeleyje.properties')
g = graph.traversal()
m = graph.openManagement()
m.makePropertyKey('name').dataType(String.class).cardinality(Cardinality.LIST).make()
m.makePropertyKey('lang').dataType(String.class).cardinality(Cardinality.LIST).make()
m.makePropertyKey('age').dataType(Integer.class).cardinality(Cardinality.LIST).make()
m.commit()
graph.io(IoCore.gryo()).readGraph('data/tinkerpop-modern.gio')
graph.tx().commit()

// Run a query -- note the planner warning recommending the use of an index
g.V().has('name', 'lop')
graph.tx().rollback()

// Create an index
m = graph.openManagement()
m.buildIndex('names', Vertex.class).addKey(m.getPropertyKey('name')).buildCompositeIndex()
m.commit()
graph.tx().commit()

// Block until the SchemaStatus transitions from INSTALLED to REGISTERED
ManagementSystem.awaitGraphIndexStatus(graph, 'names').status(SchemaStatus.REGISTERED).call()

// Reindex using TitanManagement
m = graph.openManagement()
i = m.getGraphIndex('names')
m.updateIndex(i, SchemaAction.REINDEX)
m.commit()

// Enable the index
ManagementSystem.awaitGraphIndexStatus(graph, 'names').status(SchemaStatus.ENABLED).call()

// Run a query -- Titan will use the new index, no planner warning
g.V().has('name', 'lop')
graph.tx().rollback()

// Concerned that Titan could have read cache in that last query, instead of relying on the index?
// Start a new instance to rule out cache hits.  Now we're definitely using the index.
graph.close()
graph = TitanFactory.open("conf/titan-berkeleyje.properties")
g = graph.traversal()
g.V().has('name', 'lop')
```
但是按照上述的方法，但是输出的结果却是差强人意：
```JAVA
==>GraphIndexStatusReport[success=false, indexName='comp_identifier_cyj_1', targetStatus=REGISTERED, notConverged={lc_creator=INSTALLED}, converged={}, elapsed=PT1M0.109S]
```
索引创建失败了，状态卡在了 INSTALLED。
## 解决索引创建失败问题 ##
首先想到的当然是 `google`，输入关键词“titan stuck on installed”，果然有一些小伙伴遇到了相同的问题。
- [titan 0.9.0-M2 composite index stuck on INSTALLED](https://groups.google.com/forum/#!topic/aureliusgraphs/jtE_Eo9I3Sk)
- [Unable to register the index](https://groups.google.com/forum/#!topic/aureliusgraphs/Gl_iJhJVxNo)
- [[Tinkerpop 3] Trying to create indices with already existing data](https://groups.google.com/forum/#!msg/gremlin-users/T9R_NjMerAA/ex-jz6rYCQAJ)
- [Reindexing in titan 5](https://groups.google.com/forum/#!topic/aureliusgraphs/3YWZapYg3Is)
- [Composite Indexing gets stuck in "Installed"](https://groups.google.com/forum/#!topic/gremlin-users/ovXlOkXSfP0)
- [Unable to create a composite index, stuck at INSTALLED](https://stackoverflow.com/questions/35656531/unable-to-create-a-composite-index-stuck-at-installed)
- [TitanDB Index not changing state](http://stackoverflow.com/questions/34643409/titandb-index-not-changing-state)

从 `google` 上检索到的结果主要集中在：在建立索引的时候需要不能够存在任何事务，但是并没有明确提及集群的状态应该如何处理，不能够存在事务是指创建索引的该 titan 机子不能够存在集群？当然这需要后续的本地测试才能够确定。我感觉其中有一篇中提到的方法还是挺靠谱的，脚本如下：
```JAVA
// cd titan-1.0.0-hadoop1
// rm -rf ./db
// bin/titan.sh start
// bin/gremlin.sh 
 
// Start Titan against C* + ES and define a propert key called "foo"
graph = TitanFactory.open("conf/titan-cassandra-es.properties")
mgmt = graph.openManagement()
pkey = mgmt.makePropertyKey("foo").dataType(String.class).cardinality(Cardinality.SINGLE).make()
mgmt.commit()
 
// Add a vertex with the property key set to "bar"
graph.addVertex("foo","bar")
graph.tx().commit()
 
// Define a composite graph index on the property key created above
mgmt = graph.openManagement()
pkey = mgmt.getPropertyKey("foo")
mgmt.buildIndex("index", Vertex.class).addKey(pkey).buildCompositeIndex()
mgmt.commit()
 
// Wait for the index to transition from INSTALLED to REGISTERED
import com.thinkaurelius.titan.graphdb.database.management.ManagementSystem
ManagementSystem.awaitGraphIndexStatus(graph, "index").call()
// The await call() must return something like this:
// ==>GraphIndexStatusReport[success=true, indexName='index', targetStatus=REGISTERED, notConverged={}, converged={foo=REGISTERED}, elapsed=PT0.009S]
// the success property must be true (implying that notConverged is empty)
 
// Retrieve a vertex by property condition foo=bar;
// this logs a linear scan warning because we haven't reindexed yet,
// and the index doesn't know about any properties that predate it
graph.query().has("foo", "bar").vertices()
 
// Reindex
mgmt = graph.openManagement()
index = mgmt.getGraphIndex("index")
mgmt.updateIndex(index, SchemaAction.REINDEX).get()
mgmt.commit()
 
// Reopen Titan and retrieve a vertex by property condition foo=bar; 
// this hits the index and does not log a linear scan warning
graph.close()
graph = TitanFactory.open("conf/titan-cassandra-es.properties")
graph.query().has("foo", "bar").vertices()
 
// Note: the index's status has transitioned from 
// REGISTERED to ENABLED as part of the REINDEX action
mgmt = graph.openManagement()
pkey = mgmt.getPropertyKey("foo")
index = mgmt.getGraphIndex("index")
index.getIndexStatus(pkey) // ENABLED
mgmt.rollback()
```
该方法是新开了一个 titan 的实例，也就是说没有任何的事务，通过语法：`:> graph.getOpenTransactions()`，可以得知确实是没有事务的。
我觉得需要重新弄搭建一个服务，专门用于各种测试。在测试环境重新部署了一套数据，再次按照上述的脚本执行，竟然成功了：
```
==>GraphIndexStatusReport[success=true, indexName='cyj4', targetStatus=REGISTERED, notConverged={}, converged={cyj1=REGISTERED}, elapsed=PT5.533S]
```
确实在这种情况下是可行的，我赶紧按照上述脚本的格式在 debug 环境做了实验，结果还是失败了，到底哪里出了问题？debug 环境只有一台机子，按照上述的方法在创建索引的时候不会有事务影响。难道是要保证所有的机子上的 titan 都没有事务才能够行？我估计在机子搭建的环境上遗留一个事务，果然这种情况下是无法创建索引成功的，也就验证了我的想法。之后我就在夜深人静的时候在真正的环境上做了实验，确实如此，这个在有数据的属性下创建索引的问题也就成功解决了。
## 新建索引 ##
事情的关键在于保证所有的 titan 机子没有任何的事务才行。而文档中提供的方法 `graph.tx().rollback()`，并不能够将所有的事务回滚：
### composite Index ###
```XML
// 回滚所有事物，创建 composite Index
:> int size = graph.getOpenTransactions().size();for(i=0;i<size;i++) {graph.getOpenTransactions().getAt(0).rollback()};mgmt = graph.openManagement();identifier= mgmt.getPropertyKey("identifier");mgmt.buildIndex('comp_identifier_cyj_8', Vertex.class).addKey(identifier).buildCompositeIndex();mgmt.commit();
 
// INSTALLED --> REGISTERED, success 得要为 true
:> com.thinkaurelius.titan.graphdb.database.management.ManagementSystem.awaitGraphIndexStatus(graph, "comp_identifier_cyj_8").call()
 

// 索引状态从REGISTERED --> ENABLED
:> mgmt = graph.openManagement();index = mgmt.getGraphIndex("comp_identifier_cyj_8");mgmt.updateIndex(index, SchemaAction.REINDEX).get();mgmt.commit();
 
// 查看索引状态，得要是 ENABLED 才行，composite index 到ENABLED状态时间较短（几分钟），与数据量相关
:> mgmt = graph.openManagement();pro= mgmt.getPropertyKey("identifier");mgmt.getGraphIndex("comp_identifier_cyj_8").getIndexStatus(pro);
索引状态为 ENABLED，则表示该索引已经可用
```
### Mixed Index ###
```XML
// 回滚所有事物，创建 mixed Index
:> int size = graph.getOpenTransactions().size();for(i=0;i<size;i++) {graph.getOpenTransactions().getAt(0).rollback()};mgmt = graph.openManagement();primary_category = mgmt.getPropertyKey("identifier");mgmt.buildIndex('mixed_ndresource_cyj_1', Vertex.class).addKey(primary_category,Mapping.STRING.asParameter()).buildMixedIndex("search");mgmt.commit();
 
// INSTALLED --> REGISTERED,success 得要为 true
:> com.thinkaurelius.titan.graphdb.database.management.ManagementSystem.awaitGraphIndexStatus(graph, "mixed_ndresource_cyj_1").call()
// 索引状态从REGISTERED --> ENABLED
:> mgmt = graph.openManagement();index = mgmt.getGraphIndex("mixed_ndresource_cyj_1");mgmt.updateIndex(index, SchemaAction.REINDEX).get();mgmt.commit();
 
// 查看索引状态，得要是 ENABLED 才行，mixed index 到ENABLED状态需要比较长的时间（十分钟），与数据量相关
:> mgmt = graph.openManagement();pro= mgmt.getPropertyKey("identifier");mgmt.getGraphIndex("mixed_ndresource_cyj_1").getIndexStatus(pro)
```

## 索引删除问题 ##
- 要时刻注意事务的状态
- REMOVE_INDEX 并不能够移除已经存在的索引名称
### composite index ###
```JAVA
// 回滚所有事务，状态ENABLED --> DISABLED
:> int size = graph.getOpenTransactions().size();for(i=0;i<size;i++) {graph.getOpenTransactions().getAt(0).rollback()};m = graph.openManagement();nameIndex = m.getGraphIndex('comp_identifier_cyj_8');m.updateIndex(nameIndex, SchemaAction.DISABLE_INDEX).get();m.commit()
 
状态转换时间很短，日志中输出：
Set status DISABLED on shema element comp_identifier_cyj_8 with property keys []
// 查看索引状态，success得要是true
:> com.thinkaurelius.titan.graphdb.database.management.ManagementSystem.awaitGraphIndexStatus(graph, "comp_identifier_cyj_8").status(SchemaStatus.DISABLED).call()
 
// 移除索引
// 输出的日志是和创建索引时一样，是一个逆过程
:> m = graph.openManagement();i = m.getGraphIndex('comp_identifier_cyj_8');m.updateIndex(i, SchemaAction.REMOVE_INDEX).get();m.commit()
 
// 查看索引状态
// 索引处于 DISABLED 状态，可是索引是名称还是存在的，之后建立索引就无法使用这个名称了。
:> m = graph.openManagement();nameIndex = m.getGraphIndex('comp_identifier_cyj_8');
```
### Mixed Index ###
```JAVA
// 回滚所有事务，状态ENABLED --> DISABLED
:> int size = graph.getOpenTransactions().size();for(i=0;i<size;i++) {graph.getOpenTransactions().getAt(0).rollback()};m = graph.openManagement();nameIndex = m.getGraphIndex('mixed_ndresource_cyj_1');m.updateIndex(nameIndex, SchemaAction.DISABLE_INDEX).get();m.commit()
// 查看索引状态，success得要是true
:> com.thinkaurelius.titan.graphdb.database.management.ManagementSystem.awaitGraphIndexStatus(graph, "mixed_ndresource_cyj_1").status(SchemaStatus.DISABLED).call()
 
// Mixed Index 是外部索引，只能够将索引的状态置为 DISABLED，如果需要删除则通过 ES 命令：
curl -XDELETE "http://172.24.133.44:9206/titan_debug_02/mixed_ndresource_cyj_1/"
```
## 修改索引 ##
修改索引只能够往 mixed index 中新增属性，composite index 不支持新增属性，两种索引都不支持删除属性。
### composite index ###
不支持新增与删除属性
### mixed index ### 
```XML
// 回滚所有事务，新增属性
:> int size = graph.getOpenTransactions().size();for(i=0;i<size;i++) {graph.getOpenTransactions().getAt(0).rollback()};mgmt = graph.openManagement();creator = mgmt.getPropertyKey('lc_creator');lcCreator = mgmt.getGraphIndex('mixed_ndresource_cyj_3');mgmt.addIndexKey(lcCreator, creator);mgmt.commit()

// 新增属性的状态，REGISTERED
:> mgmt = graph.openManagement();pro= mgmt.getPropertyKey("lc_creator");mgmt.getGraphIndex("mixed_ndresource_cyj_1").getIndexStatus(pro)
 
// 重建索引使新增字段从 REGISTED -- > ENABLED
:> mgmt = graph.openManagement();mgmt.updateIndex(mgmt.getGraphIndex("mixed_ndresource_cyj_3"), SchemaAction.REINDEX).get();mgmt.commit()
```
## 可能使用到的命令 ##
```XML
列出所有索引
:> graph.openManagement().getGraphIndexes(Vertex.class)
索引使用到的字段
:> graph.openManagement().getGraphIndex(String name).getFieldKeys() // name 索引名称
```
