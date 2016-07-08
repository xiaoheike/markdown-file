## gremlin 是什么 ##
`Gremlin` 是 `Apache TinkerPop` 下的一种遍历语言。`Gremlin` 是一种功能性，数据流语言，使用户能够简洁的表达复杂的图遍历。每次 `Gremlin` 遍历都包含一系列子步骤。每一个步骤都是对数据流的原子操作。`gremlin` 类似于 `java` 一样就是一种语言，但是只能不同，`gremlin` 倾向于实现图的遍历。
## titan 是什么 ##
titan 是一个可扩展的图数据库，通过分布式集群，可以存储和查询千亿顶点和边。Titan 是事务型数据库，能够支持数千并发的用户数实时执行复杂的图遍历。
另外，titan 还支持如下特性：
- 为不断增长的数据和用户群弹性和线性扩展
- 分布式数据和分片提高性能和容错能力
- 多数据中心的高可用性和热备份
- 支持 `ACID` 和最终一致性
- 支持多种数据存储后端
        - `Apache Cassandra`
        - `Apache HBase`
        - `Oracle BerkeleyDB`
- 与大数据集成支持全局数据分析，报告和 `ETL`
        - `Apache Spark`
        - `Apache Giraph`
        - `Apache Hadoop`
- 支持地理，数字范围和全文所有：
        - `ElasticSearch`
        - `Solr`
        - `Lucene`
- 默认集成 `TinkerPop` 图技术栈：
        - `Gremlin graph query language`
        - `Gremlin graph server`
        - `Gremlin applications`
- 以 `Apache 2` 协议开源

## titan 与 gremlin 的关系 ##
我们查看 `titan` 的 `jar` 包会发现，`titan` 将许多的服务给集合起来了，比如 `gremlin`，`cassandra`，`es` 等等。也就是说 `titan` 像是一个集合体，将不同的服务聚集并使他们能够相互协作，当需要全文检索时则调用 `es` 或者 `solr`，当需要存储服务时则调用相关的数据库，比如 `cassandra` 等等。
使用 `titan` 能够让我们在没有安装其他服务的情况下能够快速开始图的相关操作，当然为了性能，最终是需要独立部署相关的应用的。 


