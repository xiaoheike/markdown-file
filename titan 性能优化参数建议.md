## titan 性能优化难处 ##
titan 使用的用户比较少，google 都找不到很详细的信息，能够得到的比较完整的说明在官方文档 [12. Configuration Reference](http://s3.thinkaurelius.com/docs/titan/1.0.0/titan-config-ref.html)。文档也比较简陋，有提供文字说明，但是并没有提供实例，以及配置之后的作用等信息。
我是觉得titan 真心是有点简陋，没有他人的经验支撑，官方文档也是各种简陋，连基本的 `get started` 教程都没有。单单是默认配置启动都很困难。所以性能提升也是困难的事情，得要先搞明白哪些参数都是什么意思
最重要的两个配置文件：`gremlin-server.yaml` 和 `titan-cassandra-es.properties`。通过 `google` 检索 `titan performance` 或者 `gremlin-server.yaml parameter meaning` 等等关键词都没有查到有效的信息。

## 配置titan-cassandra-es.properties ##
这个文件包含了 titan 启动之后加载的所有配置，这里边的配置在[12. Configuration Reference](http://s3.thinkaurelius.com/docs/titan/1.0.0/titan-config-ref.html) 中有涉及到大多数，但是该文章写得感觉不是很好，并没有准确的实例，我们也只能够猜测每个参数可能的作用。
但是有参考文件比没有强太多了，所以我阅读整篇文档，找出了其中可能影响的因素并翻译，大致如下几个分类。
### Elasticsearch ###
```XML
index.search.backend：索引后端用于扩展并且优化Titan 查询功能。这个设置是可选的。Titan 可以使用多个不同的索引后端。因此这个选项可以多次出现，只要“index”与“backend”之间的X 是不相同的。与存储后端类似，这个应该被设置为Titan 内置的标准后端简写（简写：lucene，elasticsearch，es，solr）或者完事的包和一个自定义/第三方 IndexProvider 实现的类名。缺省值为elasticsearch。
index.search.hostname：主机地址或者一系列以逗号分隔的索引后端服务地址。这个仅在对一些索引后端有用，例如elasticsearch 和solr。
index.search.elasticsearch.client-only：es node.client 选项通过这个布尔值设定，并且es node.data 选项被设置为与这个值相反。值为true 将创建一个没有数据的瘦客户端。值为false则创建一个正常的能够保存数据的节点。
index.search.elasticsearch.cluster-name：es 集群的名字。这个名字应该要匹配es 配置文件中的“cluster.name”。默认值为elasticsearch。
index.search.elasticsearch.health-request-timeout：当Titan 初始化es 后端，Titan 将等待一段时间间隔使得es 集群至少达到yellow 状态。这个字符串格式为自然数后边跟着一个小写“s”，例如 3s 或者 60s。缺省时间30s。
index.search.elasticsearch.ignore-cluster-name：是否跳过连接点集群名称的验证。这个选项仅在接口配置跟踪上使用（参见手册中关于es 跟踪配置）。缺省值为true
index.search.elasticsearch.interface：
index.search.elasticsearch.load-default-node-settings：
index.search.elasticsearch.local-mode：
index.search.elasticsearch.sniff：是否开户嗅探功能。这个选项仅仅应用于TransprotClient。允许这个选项使得TransprotClient尝试发现在启动时初始主机列表以外的其他集群顶点。缺省值为true。
index.search.elasticsearch.ttl-interval：es 内置过期文档删除器的运行间隔时间。这个字符串将会成为es indices.ttl.interval  的设置，有相应的数据格式，例如5s或者60s。缺省值为5s
index.search.elasticsearch.create.sleep：第一次成功创建一个索引所需的时间，单位是毫秒。这仅在Titan第一次启动时有用，如果索引已经创建则这个配置不起作用。缺省值200ms。
```
从上边的配置来看，es 在 titan 层面的配置是有限的，似乎都是一些必要的信息和性能优化没有多大关系。我们使用共享平台的 es，因为它们的实例是整个公司共用的，所以配置是不能够任由我们修改的。
### cassandra ###
```XML
storage.backend：titan主要持久化供应商。这个参数是必须。它应该被设置为 Titan 内置的标准后端简写名字（简写：berkeleyje，cassandrathrift，cassandra，astyanas，embeddedcassandra，hbase，inmemory）或者完整的包和一个自定义/第三方StoreManager实现的类名。
storage.batch-loading：是否启用批量加载到存储后端。缺省值为false
storage.buffer-size：批量缓存大小，这个变动是持久的。（Size of the batch in which mutations are persisted）缺省值为1024.
storage.conf-file：存储后端的配置文件路径，这个存储后端需要或者支持一个单一的分隔文件配置。没有缺省值。
storage.connection-timeout：连接一个远程数据库实例默认的超时时间，单位是毫秒。缺省值为10000ms。
storage.directory：存储后端的存储目录，需要是一个本地目录。没有缺省值。
storage.hostname：主机地址或者一系列以逗号分隔的存储后端服务地址。这个仅仅对一些存储后端有用，例如cassandra和hbase。storage.page-size：Titan break requests that may return many results from distributed storage backends into a series of requests for small chunks/pages of results, where each chunk contains up to this many elements. 只可意会，大致是说如果请求返回的块或者分布的大小超过这个值将会被中断。缺省值为100。
storage.parallel-backend-ops：是否Titan 应该尝试对存储并行操作。缺省值为true。
storage.password：存储后端密码。没有缺省值。
storage.port：连接存储后端的端口。没有缺省值。
storage.read-only：只读数据库。缺省值为false。
storage.read-time：等待后端读取操作成功完成的最大时间，单位为毫秒。如果一个后端读操作短暂性失败，Titan 将以指数形式避让并重试操作，直到等待时间被消耗完。（Titan will backoff exponentially and retry the operation until the wait time has been exhausted.）缺省值10000ms。
storage.setup-wait：当Titan 服务启动时，等待存储后端成为可使用状态的时间，单位为毫秒。缺省值为60000ms
storage.transactions：如果存储后端支持事务，能够开启事务功能。缺省值为true。
storage.username：存储后端用户名。没有缺省值。
storage.write-time：等待后端写操作成功完成的最大时间，单位为毫秒。如果一个后端写操作短暂性失败，Titan 将以指数形式避让并重试操作，直到等待时间被消耗完。（Titan will backoff exponentially and retry the operation until the wait time has been exhausted.）。缺省值为100000ms。
storage.cassandra.atomic-batch-mutate：true为使用cassandra 原子批量变动，false则不使用原子批量。
storage.cassandra.compression：当存储数据时，存储后端是否应该使用压缩。缺省值为true。
storage.cassandra.compression-block-size：压缩块大小，单位为KB。缺省值为64KB。
storage.cassandra.compression-type：压缩类型。缺省值为LZ4Compressor。
storage.cassandra.frame-size-mb：thrift 帧大小，单位为MB。缺省值为15MB。
storage.cassandra.keyspace：Titan 的keyspace 名字。如果不存在将被创建。缺省值为titan。
storage.cassandra.read-consistency-level：cassandra 读一致性等级。缺省值为QUORUM。
storage.cassandra.replication-factor：数据副本的数量（包括最最初的副本）这个仅仅对后端本身支持数据复制才有效。缺省值为1。
storage.cassandra.replication-strategy-class：Titan keysapce复制策略。缺省值为org.apache.cassandra.locator.SimpleStrategy。
storage.cassandra.replication-strategy-options：	
Replication strategy options, e.g. factor or replicas per datacenter. This list is interpreted as a map. It must have an even number of elements in [key,val,key,val,…] form. A replication_factor set here takes precedence over one set with storage.cassandra.replication-factor。没有缺省值。
storage.cassandra.write-consistency-level：cassandra 写一致性等级。缺省值为QUORUM。
```
cassandra 有一些可以优化的性能参数，比如事务，压缩等等。
### Cache ###
```XML
cache.db-cache：是否开启数据库级别的缓存，这个设置被所有的事务共享。启用这个选择将加速遍历通过将经常受访问的节点放置到内存中，但是这也同时增加了读到脏数据的可能性。禁用这个选项将迫使事务每次在读/写操作之前都要从存储介质中获取图元素。缺省值为false。
cache.db-cache-clean-wait：在清空之后数据项还能够被保存多长时间，单位是毫秒。这个选项仅仅在分布式存储后端有用，因为存储后端能够保证将数据写入后端而不必要让数据马上呈现。缺省值为50ms。
cache.db-cache-size：Titan 数据库级别缓存大小。值在0-1之间被认为是VM 堆栈的百分比，当大于1时被认为是绝对的字节大小。
cache.db-cache-time：数据库级别缓存默认过期时间，单位为毫秒。当它们达到这个时间，数据项被清理，即使缓存中还有空闲空间。设为0为禁止超时（缓存数据项将一直存活直到内存压力触发数据清理）。缺省值为10000ms
cache.tx-cache-size：最近使用顶点的事务级缓存最大大小。缺省值为20000。
cache.tx-dirty-size：未提交的脏顶点的事务级缓存的初始大小。这是写操作很费性能，性能敏感的事务性工作负载性能优化点。如果设置，它应该大致匹配每笔交易修改的平均顶点。
cache.tx-cache-size：最近使用节点的事务级缓存最大值  缺省值为20000
```
缓存的配置项比较少，titan 的默认配置似乎已经很大，能够做的修改并不多。
### query ###
```XML
query.batch：存储后端遍历查询是否应该批量查询。如果查询到后端有明显的延迟，这个将可能引起显著的性能改善。（This can lead to significant performance improvement if there is a non-trivial latency to the backend.）缺省值为false。这个可以设置为true试试？
query.fast-property：是否提前获得顶点的所有属性。这个操作能够抵消立马检索相同顶点的其他属性的开销。这个操作代价昂贵，如果一个顶点有非常多的属性。缺省值为false。咱们的顶点属性如果不多就开启吧，多就算了。
query.force-index：通过索引无法检索到数据是否应该抛出异常。这样做限制了Titan 的图查询功能，但是确保大图能够避开慢查询。建议在生产环境使用这个配置。（Recommended for production use of Titan.）缺省值为false。
query.ignore-unknown-index-key：是否忽略用户索引查询中出现的未定义类型。缺省值为false。
query.smart-limit：是否查询优化器应该尝试猜测为查询设置一个聪明的限制，确保命中最大结果集。（Whether the query optimizer should try to guess a smart limit for the query to ensure responsiveness in light of possibly large result sets.）如果启用该选项，那些将逐步加载。缺省值为true。我想知道那些是指什么？
```
我是觉得查询的配置将是瓶颈所在。
## gremlin-server.yaml ##
这个文件里边又需要的默认配置，但是并不是知道什么作用，也无法通过 google 查找到相关文章，几经波折，五一中发现类：[org.apache.tinkerpop.gremlin.server.Settins](http://tinkerpop.apache.org/javadocs/3.0.2-SNAPSHOT/full/org/apache/tinkerpop/gremlin/server/Settings.html)中的字段和这个配置文件是对应的，所以我猜测 Settings 这个类和这个配置文件是对应关系。于是我整理除了配置。
```XML
host：gremlin-server绑定的ip
port：gremlin-server绑定的端口
threadPoolWorker：worker线程池的大小。缺省值为1并且不应该超过cpu内核数目的两倍。一个处于非阻塞模式的 worker 线程为一个或者多个信道执行非阻塞读和写。我们应该设置为24即可，或者更小一些。
gremlinPool：gremlin 线程池的大小。这个线程池处理 gremlin 脚本地执行和其他相关“长期运行”的进程。这个设置应该足够大，确保非阻塞worker 线程能够在有限的队列中处理请求。缺省值为8。这个值的设置取决于gremlin server 处理脚本的时间。如果脚本执行速度快，几十毫秒，则这个值为2*threadPoolWorker。慢百毫秒，则这个值为4*threadPoolWorker.
scriptEvaluationTimeout：等待脚本被完全执行完成的时间，单位是毫秒。缺省值为30000ms。这个应该使用默认值即可。
serializedResponseTimeout：等待脚本序列化结果的时间，单位是毫秒。这个值代表请求的总序列化时间。缺省值为30000ms。这个应该使用默认值即可，或者稍微增大一些。
channelizer：gremlin-server 内部使用的通信协议。有如下几个选择：
1.	AbstractChannelizer
2.	HttpChannelizer
3.	NioChannelizer
4.	WebSocketChannelizer
并不了解其他几个协议有什么作用，暂时使用WebsocketChannelizer 即可。
graphs：类型为Map<String,String>。图的名字为key，图的配置文件为value。
plugins：gremlin-server 启用插件列表。
scriptEngines：脚本引擎。大概是引用外部服务时设置，这个暂时不需要考虑。
serializers：序列化配置。有如下几个选择：
1.	AbstractGraphSONMessageSerializerV1d0
2.	AbstractGryoMessageSerializerV1d0
3.	AbstractMessageSerializer
4.	GraphSONMessageSerializerGremlinV1d0
5.	GraphSONMessageSerializerV1d0
6.	GryoLiteMessageSerializerV1d0
7.	GryoMessageSerializerV1d0
序列化这个可能需要再深入一些，因为我们经常碰到序列化的异常。
processors：自定义接口 OpProcessor 的实现类。有如下几个选择：
1.	AbstractEvalOpProcessor
2.	AbstractOpProcessor
3.	ControlOpProcessor
4.	SessionOpProcessor
5.	StandardOpProcessor
6.	TraversalOpProcessor
这个很陌生，暂时使用默认设置即可。
metrics：设置gremlin-server 的指标
1.	consoleReporter：写到控制台的指标报告
2.	csvReporter：写到csv 文件的指标报告
3.	jmxReporter：写到JMX的指标报告
4.	slf4jReporter：写到SL4J 输出的指标报告
5.	gangliaReporter：写到Granglia 的指标报告
6.	graphiteReporter：写到Graphite 的指标报告
这几个应该都是相同的内容，只是写入不同的文件，建议只保留 slf4jReporter 并延长输出的时间。默认时间间隔为180s。
threadPoolBoss：boss 线程池的大小。缺省值为1并且应该保持在1。这个线程绑定一个端口并监控该端口，接收到来的连接。一旦连接被成功接收，这个线程将会把它交给worker线程处理。这个设置为1 即可。
maxInitialLineLength：一个请求初始化行的最大长度（例如：“Get / HTTP/1.0”），基本上控制了提交URI的最大长度。这个设置和 Netty 的 HttpRequestDecoder 有关系。不懂初始化行是什么意思？这个值的配置还需要考虑。
maxHeaderSize：所有header 的最大长度。这个设置和 Netty 的 HttpRequestDecoder 有关系。这个应该使用默认配置就可以了吧？
maxChunkSize：内容或者每个块的最大长度。如果内容长度超过这个值，编码请求的传输编码将被转换为”chunked“，并且内容将会被分为多个 HttpContents。如果请求该HTTP请求的传输编码已经是”chunked“，并且该块的大小已经超过最大长度，那么每一个块将会被切分为更小的块。这个值需要恰当的配置。
maxContentLength：一个消息所聚合的内容的最大长度。与maxChunkSize 协同合作，如果分块被合成一个单一消息时。如果一个请求超过这个值将会返回413 – 请求实体太大状态码。一个响应超过这个请求将会触发一个内部异常。这个值需要恰当的配置。
maxAccumulationBufferComponents：能够聚合为一个消息的组件的最大数目。不懂这个有什么用。
resultIterationBatchSize：Number of items in a particular resultset to iterate and serialize prior to pushing the data down the wire to the client.
writeBufferHighWaterMark：如果网络发送缓冲区的字节数目超过这个值，则信道不再是可写，不会接收额外的写入数据直到缓冲区被清空或者 writeBufferLowWaterMark 达到（原文If the number of bytes in the network send buffer exceeds this value then the channel is no longer writeable, accepting no additional writes until buffer is drained and the writeBufferLowWaterMark is met.）
writeBufferLowWaterMark：如果网络发送缓冲区的字节数目超过这个值，则信道不再是可写，不会接收额外的写入数据直到缓冲区被清空或者达到这个值。
ssl：设置ssl。这个我们可以先使用默认配置，disabled ssl。
```
这个配置文件相对很集中，需要修改的也不多。
## titan 建议配置 ##
### gremlin-server.yaml ###
```XML
host: 192.168.122.129
port: 8182
threadPoolWorker: 24
gremlinPool: 96
scriptEvaluationTimeout: 30000
serializedResponseTimeout: 30000
channelizer: org.apache.tinkerpop.gremlin.server.channel.WebSocketChannelizer
graphs: {
  graph: conf/titan-cassandra-es.properties}
plugins:
  - aurelius.titan
scriptEngines: {
  gremlin-groovy: {
    imports: [java.lang.Math],
    staticImports: [java.lang.Math.PI],
    scripts: [scripts/empty-sample.groovy]},
  nashorn: {
      imports: [java.lang.Math],
      staticImports: [java.lang.Math.PI]}}
serializers:
  - { className: org.apache.tinkerpop.gremlin.driver.ser.GryoMessageSerializerV1d0, config: { useMapperFromGraph: graph }}
  - { className: org.apache.tinkerpop.gremlin.driver.ser.GryoMessageSerializerV1d0, config: { bufferSize: 819200, serializeResultToString: true }}
  - { className: org.apache.tinkerpop.gremlin.driver.ser.GraphSONMessageSerializerGremlinV1d0, config: { useMapperFromGraph: graph }}
  - { className: org.apache.tinkerpop.gremlin.driver.ser.GraphSONMessageSerializerV1d0, config: { useMapperFromGraph: graph }}
processors:
  - { className: org.apache.tinkerpop.gremlin.server.op.session.SessionOpProcessor, config: { sessionTimeout: 28800000 }}
metrics: {
  csvReporter: {enabled: false, interval: 180000, fileName: /tmp/gremlin-server-metrics.csv},
  slf4jReporter: {enabled: true, interval: 180000}}
threadPoolBoss: 1
maxInitialLineLength: 4096
maxHeaderSize: 8192
maxChunkSize: 8192
maxContentLength: 65536
maxAccumulationBufferComponents: 1024
resultIterationBatchSize: 64
writeBufferLowWaterMark: 32768
writeBufferHighWaterMark: 65536
ssl: {
  enabled: false}

host：主机ip
port：8182
threadPoolWorker：24。官方建议不要超过2*cpu核数。136机子有2个cpu，每个cpu 6个核心，所以最大值可以为24。
gremlinPool：96。官方建议：这个值需要根据咱们提供脚本类型处理的快与慢，快是指在小几百毫秒内能够被处理完成，慢是批更长的时间。对于执行很快的脚本建议值为2*threadPoolWorker，相反则为4*threadPoolWorker。
scriptEvaluationTimeout：30000。单指脚本评估超时时间，并不包括其他的操作，比如遍历以及序列化。
serializedResponseTimeout：30000。序列化结果的时间。
```
### titan-cassandra-es.properties ###
```XML
# graph 实现类，必须配置
gremlin.graph=com.thinkaurelius.titan.core.TitanFactory
# 后端存储数据库，使用Cassandra
storage.backend=cassandra
# Cassandra主机ip
storage.hostname=127.0.0.1
# Cassandra批量载入
storage.batch-loading = true
# Cassandra批量载入大小
storage.buffer-size = 2048
# titan 连接Cassandra超时时间
storage.connection-timeout = 30000
# 读Cassandra数据超时时间
storage.read-time = 30000
# 开启Cassandra事务
storage.transactions = true
# Cassandra写入超时时间
storage.write-time = 20000
# Cassandra 读写事务一致性等级
storage.cassandra.read-consistency-level = QUORUM
storage.cassandra.write-consistency-level = QUORUM
# Cassandra 副本数量，包括它自身，所以最小值应该为 1
# storage.cassandra.replication-factor = 3
# Cassandra 用户名
storage.username=dev_cassandra_lcms_test
# Cassandra 密码
storage.password=6674N59V8986
# Cassandra keyspace，相当于数据库
storage.cassandra.keyspace=dev_cassandra_lcms_test

# 开启缓存
cache.db-cache = true
# 这个参数只在分布式环境下有用。
cache.db-cache-clean-wait = 50
# 缓冲强制清理时间
cache.db-cache-time = 180000
# 缓存占用jvm 堆大小。值大于1 为绝对值，小于1 为百分比
cache.db-cache-size = 0.5
# 事务缓存大小
cache.tx-cache-size = 20000
# 未提交事务缓存大小
cache.tx-dirty-size = 5000

# 索引采用es
index.search.backend=elasticsearch
index.search.hostname=127.0.0.1
# elasticsearch 当客户端使用， 瘦客户端，不持有数据
index.search.elasticsearch.client-only=true
# es 集群名称
index.search.elasticsearch.cluster-name = es1.5.1-titan
# es 索引
index.search.index-name = titan_dev

# 存储后端遍历查询是否应该批量查询
query.batch = true
# 查询一个顶点立即获得它的所有属性，false 不开启
query.fast-property = false
# 不忽略用户索引查询中出现的未定义类型
query.ignore-unknown-index-key = false
# 查询优化器应该尝试猜测为查询设置一个聪明的限制，确保命中最大结果集
query.smart-limit = true
```
## JVM 优化　##
JVM 已经很成熟了，网络上有不少资料，所以处理相对简单，站在别人的肩膀上。总结之后给出了建议的配置参数。
```XML
-Xmx16384m –Xms16384m -Xss384k  -XX:NewRatio=4 -XX:SurvivorRatio=8 -XX:MaxTenuringThreshold=7 -XX:MaxPermSize=96m -XX:+UseParallelGC  -XX:ParallelGCThreads=24  -XX:+UseParallelOldGC   -XX:MaxGCPauseMillis=100 -XX:+UseAdaptiveSizePolicy

-Xmx16384m：设置JVM内存为16384M。这个值可以根据实际内存修改

-Xms16384m：设置JVM内存为16384m。此值可以设置与-Xmx相同，以避免每次垃圾回收完成后JVM重新分配内存。这个值可以根据实际内存修改

-Xss384k：设置每个线程的堆栈大小。经过再虚拟机上测试titan，根据titan的报错，这个参数至少得要设置为228k。
 
-XX:NewRatio=4:设置年轻代（包括Eden和两个Survivor区）与年老代的比值（除去持久代）。设置为4，则年轻代与年老代所占比值为1：4，年轻代占整个堆栈的1/5

-XX:SurvivorRatio=8：设置年轻代中Eden区与Survivor区的大小比值。设置为8，则两个Survivor区与一个Eden区的比值为2:8，一个Survivor区占整个年轻代的1/10

-XX:MaxTenuringThreshold=7：表示一个对象如果在救助空间（Survivor区）移动7次还没有被回收就放入年老代。如果将此值设置为一个较大值，则年轻代对象会在Survivor区进行多次复制，这样可以增加对象在年轻代存活时间，增加对象在年轻代即被回收的概率。默认值是15

-XX:MaxPermSize=96m：永久代大小

-XX:+UseParallelGC：选择垃圾收集器为并行收集器。此配置仅对年轻代有效。即上述配置下，年轻代使用并发收集，而年老代仍旧使用串行收集。

-XX:ParallelGCThreads=24：配置并行收集器的线程数，即：同时多少个线程一起进行垃圾回收。此值最好配置与处理器核数相等。

-XX:+UseParallelOldGC：配置年老代垃圾收集方式为并行收集。JDK6.0支持对年老代并行收集。

-XX:MaxGCPauseMillis=100:设置每次年轻代垃圾回收的最长时间，如果无法满足此时间，JVM会自动调整年轻代大小，以满足此值

-XX:+UseAdaptiveSizePolicy：设置此选项后，并行收集器会自动选择年轻代区大小和相应的Survivor区比例，以达到目标系统规定的最低相应时间或者收集频率等，此值建议使用并行收集器时，一直打开。
```


