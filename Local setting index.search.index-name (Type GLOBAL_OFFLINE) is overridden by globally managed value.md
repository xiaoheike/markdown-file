## 异常信息 ##
```XML
         \,,,/^M
         (o o)^M
-----oOOo-(3)-oOOo-----^M
225  2016-06-23 17:26:57 [main] INFO  org.apache.tinkerpop.gremlin.server.GremlinServer  - Configuring Gremlin Server from conf/gremlin-server/gremlin-server.yaml
331  2016-06-23 17:26:57 [main] INFO  org.apache.tinkerpop.gremlin.server.util.MetricManager  - Configured Metrics ConsoleReporter configured with report interval=180000ms
334  2016-06-23 17:26:57 [main] INFO  org.apache.tinkerpop.gremlin.server.util.MetricManager  - Configured Metrics CsvReporter configured with report interval=180000ms to fileName=/tmp/gremlin-server-metrics.csv
339  2016-06-23 17:26:57 [main] INFO  org.apache.tinkerpop.gremlin.server.util.MetricManager  - Configured Metrics JmxReporter configured with domain= and agentId=
342  2016-06-23 17:26:57 [main] INFO  org.apache.tinkerpop.gremlin.server.util.MetricManager  - Configured Metrics Slf4jReporter configured with interval=180000ms and loggerName=org.apache.tinkerpop.gremlin.server.Settings$Slf4jReporterMetrics
1537 2016-06-23 17:26:58 [main] INFO  com.thinkaurelius.titan.core.util.ReflectiveConfigOptionLoader  - Loaded and initialized config classes: 12 OK out of 12 attempts in PT0.1S
1738 2016-06-23 17:26:58 [main] INFO  org.reflections.Reflections  - Reflections took 135 ms to scan 2 urls, producing 0 keys and 0 values
1848 2016-06-23 17:26:58 [main] WARN  com.thinkaurelius.titan.graphdb.configuration.GraphDatabaseConfiguration  - Local setting index.search.index-name=titan_debug (Type: GLOBAL_OFFLINE) is overridden by globally managed value (testfor44titan).  Use the ManagementSystem interface instead of the local configuration to control this setting.
1850 2016-06-23 17:26:58 [main] WARN  com.thinkaurelius.titan.graphdb.configuration.GraphDatabaseConfiguration  - Local setting index.search.elasticsearch.cluster-name=es1.5.1-titan-debug (Type: GLOBAL_OFFLINE) is overridden by globally managed value (es1.5.1-titan).  Use the ManagementSystem interface instead of the local configuration to control this setting.
1857 2016-06-23 17:26:58 [main] INFO  com.thinkaurelius.titan.diskstorage.cassandra.thrift.CassandraThriftStoreManager  - Closed Thrift connection pooler.
1866 2016-06-23 17:26:58 [main] INFO  com.thinkaurelius.titan.graphdb.configuration.GraphDatabaseConfiguration  - Generated unique-instance-id=7f00000139594-other-cl-031911
2001 2016-06-23 17:26:58 [main] INFO  com.thinkaurelius.titan.diskstorage.Backend  - Configuring index [search]
2160 2016-06-23 17:26:59 [main] INFO  org.elasticsearch.plugins  - [Korath the Pursuer] loaded [], sites []
3208 2016-06-23 17:27:00 [main] INFO  com.thinkaurelius.titan.diskstorage.es.ElasticSearchIndex  - Configured remote host: 172.24.133.99 : 9300
3411 2016-06-23 17:27:00 [main] WARN  org.elasticsearch.client.transport  - [Korath the Pursuer] node null not part of the cluster Cluster [es1.5.1-titan], ignoring...
3416 2016-06-23 17:27:00 [main] WARN  org.apache.tinkerpop.gremlin.server.GremlinServer  - Graph [graph] configured at [conf/titan-cassandra-es.properties] could not be instantiated and will not be available in Gremlin Server.  GraphFactory message: GraphFactory could not instantiate this Graph implementation [class com.thinkaurelius.titan.core.TitanFactory]
java.lang.RuntimeException: GraphFactory could not instantiate this Graph implementation [class com.thinkaurelius.titan.core.TitanFactory]
        at org.apache.tinkerpop.gremlin.structure.util.GraphFactory.open(GraphFactory.java:82)
        at org.apache.tinkerpop.gremlin.structure.util.GraphFactory.open(GraphFactory.java:70)
        at org.apache.tinkerpop.gremlin.structure.util.GraphFactory.open(GraphFactory.java:104)
        at org.apache.tinkerpop.gremlin.server.GraphManager.lambda$new$27(GraphManager.java:50)
        at java.util.LinkedHashMap$LinkedEntrySet.forEach(LinkedHashMap.java:671)
        at org.apache.tinkerpop.gremlin.server.GraphManager.<init>(GraphManager.java:48)
        at org.apache.tinkerpop.gremlin.server.util.ServerGremlinExecutor.<init>(ServerGremlinExecutor.java:94)
        at org.apache.tinkerpop.gremlin.server.GremlinServer.<init>(GremlinServer.java:88)
        at org.apache.tinkerpop.gremlin.server.GremlinServer.main(GremlinServer.java:290)
Caused by: java.lang.reflect.InvocationTargetException
        at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
        at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.lang.reflect.Method.invoke(Method.java:498)
        at org.apache.tinkerpop.gremlin.structure.util.GraphFactory.open(GraphFactory.java:78)
        ... 8 more
Caused by: java.lang.IllegalArgumentException: Could not instantiate implementation: com.thinkaurelius.titan.diskstorage.es.ElasticSearchIndex
        at com.thinkaurelius.titan.util.system.ConfigurationUtil.instantiate(ConfigurationUtil.java:55)
        at com.thinkaurelius.titan.diskstorage.Backend.getImplementationClass(Backend.java:473)
        at com.thinkaurelius.titan.diskstorage.Backend.getIndexes(Backend.java:460)
        at com.thinkaurelius.titan.diskstorage.Backend.<init>(Backend.java:147)
        at com.thinkaurelius.titan.graphdb.configuration.GraphDatabaseConfiguration.getBackend(GraphDatabaseConfiguration.java:1805)
at com.thinkaurelius.titan.graphdb.database.StandardTitanGraph.<init>(StandardTitanGraph.java:123)
        at com.thinkaurelius.titan.core.TitanFactory.open(TitanFactory.java:94)
        at com.thinkaurelius.titan.core.TitanFactory.open(TitanFactory.java:74)
        ... 13 more
Caused by: java.lang.reflect.InvocationTargetException
        at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
        at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62)
        at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
        at java.lang.reflect.Constructor.newInstance(Constructor.java:423)
        at com.thinkaurelius.titan.util.system.ConfigurationUtil.instantiate(ConfigurationUtil.java:44)
        ... 20 more
Caused by: org.elasticsearch.client.transport.NoNodeAvailableException: None of the configured nodes are available: []
        at org.elasticsearch.client.transport.TransportClientNodesService.ensureNodesAreAvailable(TransportClientNodesService.java:279)
        at org.elasticsearch.client.transport.TransportClientNodesService.execute(TransportClientNodesService.java:198)
        at org.elasticsearch.client.transport.support.InternalTransportClusterAdminClient.execute(InternalTransportClusterAdminClient.java:86)
        at org.elasticsearch.client.support.AbstractClusterAdminClient.health(AbstractClusterAdminClient.java:127)
        at org.elasticsearch.action.admin.cluster.health.ClusterHealthRequestBuilder.doExecute(ClusterHealthRequestBuilder.java:92)
        at org.elasticsearch.action.ActionRequestBuilder.execute(ActionRequestBuilder.java:91)
        at org.elasticsearch.action.ActionRequestBuilder.execute(ActionRequestBuilder.java:65)
        at com.thinkaurelius.titan.diskstorage.es.ElasticSearchIndex.<init>(ElasticSearchIndex.java:201)
        ... 25 more
3419 2016-06-23 17:27:00 [main] INFO  org.apache.tinkerpop.gremlin.server.util.ServerGremlinExecutor  - Initialized Gremlin thread pool.  Threads in pool named with pattern gremlin-*
3704 2016-06-23 17:27:00 [main] INFO  org.apache.tinkerpop.gremlin.groovy.engine.ScriptEngines  - Loaded nashorn ScriptEngine
4019 2016-06-23 17:27:00 [main] INFO  org.apache.tinkerpop.gremlin.groovy.engine.ScriptEngines  - Loaded gremlin-groovy ScriptEngine
4555 2016-06-23 17:27:01 [main] WARN  org.apache.tinkerpop.gremlin.groovy.engine.GremlinExecutor  - Could not initialize gremlin-groovy ScriptEngine with scripts/empty-sample.groovy as script could not be evaluated - javax.script.ScriptException: groovy.lang.MissingPropertyException: No such property: graph for class: Script1
```
真正的错误信息为：
```XML
1848 2016-06-23 17:26:58 [main] WARN  com.thinkaurelius.titan.graphdb.configuration.GraphDatabaseConfiguration  - Local setting index.search.index-name=titan_debug (Type: GLOBAL_OFFLINE) is overridden by globally managed value (testfor44titan).  Use the ManagementSystem interface instead of the local configuration to control this setting.
```
从上边的信息来看，是索引的集群名称不对，elasticsearch 应用 config/ 目录下 elasticsearch.yaml 集群名称设置为：
```XML
cluster.name: es1.5.1-titan-debug
```
titan-1.0.0-hadoop1 应用 conf/ 目录下 titan-cassandra-es.properties 文件 es 集群配置如下：
```XML
index.search.elasticsearch.cluster-name = es1.5.1-titan-debug
index.search.index-name = titan_debug
```
索引信息的配置是正确的，但是无法理解 testfor44titan 是怎么冒出来的。
## 原因 ##
titan 使用的存储数据库为 cassandra，而 cassandra 中的数据并没有清理。titan 连接cassandra 会默认生成如下表：
```XML
edgestore;
edgestore_lock_;
graphindex;
graphindex_lock_;
system_properties;
system_properties_lock_;
systemlog;
titan_ids;
txlog;
```
猜测 titan 应该是将相关的es 索引信息存放在某个表中了，每次启动连接是都会对索引信息进行校验。
## 解决 ##
删除 cassandra 中的所有相关表即可。
```XML
drop table edgestore;
drop table edgestore_lock_;
drop table graphindex;
drop table graphindex_lock_;
drop table system_properties;
drop table system_properties_lock_;
drop table systemlog;
drop table titan_ids;
drop table txlog;
```
之后通过命令启动 gremlin-server.sh 即可：
```XML
nohup sh bin/gremlin-server.sh conf/gremlin-server/gremlin-server.yaml run &
```
