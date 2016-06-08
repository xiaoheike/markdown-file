## 基本信息 ##
默认端口：9200
启动方式：下载ElasticSearch.rar 解压运行windows 下 bin/elasticsearch.bat，linux下运行bin/elasticsearch.sh
## ElasticSearch与关系型数据库的对应关系 ##
关系数据库    --> 数据库 --> 表   --> 行   --> 列(Columns)
Elasticsearch --> 索引   --> 类型 --> 文档 --> 字段(Fields)
## 节点总数 ##
`curl -XGET 'http://localhost:9200/_count?pretty'`
pretty：以JSON格式展示返回值
```JSON
{
	"count": 0,
	"_shards": {
		"total": 0,
		"successful": 0,
		"failed": 0
	}
}
```
## 创建一条文档记录 ##
`curl -XPUT http://localhost:9200/test/employee/1`
body
```JSON
{
	"first_name": "Jane",
	"last_name": "Smith",
	"age": 37,
	"about": "I like to collect rock albums",
	"interests": ["music"]
}
```
1：id值，相当于mysql数据库中表的主键
test：索引，相当于mysql的数据库
employee：类型，相当于mysql的表
注意：如果多次POST该请求，如果body内容修改，则ElasticSearch文档也会被更新，并不会产生额外的一条文档记录。
## 检索一条记录 ##
`curl -XGET http://localhost:9200/test/employee/1`
检索test索引下类型为employee，id值为1的文档记录
返回值
```JSON
{
  "_index": "test",
  "_type": "employee",
  "_id": "1",
  "_version": 1,
  "found": true,
  "_source": {
    "first_name": "John",
    "last_name": "Smith",
    "age": 25,
    "about": "I love to go rock climbing",
    "interests": [
      "sports",
      "music"
    ]
  }
}
```
source：中包含完整的文档记录。

PUT：更新或者创建文档记录
HEAD：查询该文档是否存在
DELETE：删除该文档，删除一个文档不会立即生效，它只是被标记成已删除，elasticsearch将会在添加更多索引的时候才会在后台进行删除内容的清理
## 判断指定id文档是否存在 ##
`curl -i -XHEAD http://localhost:9200/test/employee/8888`
不存在该id为8888返回值
```XML
HTTP/1.1 404 Not Found
Content-Type: text/plain; charset=UTF-8
Content-Length: 0
```
存在该id为8888返回值
```XML
HTTP/1.1 200 OK
Content-Type: text/plain; charset=UTF-8
Content-Length: 0
```
## 搜索前10条记录 ##
`curl -XGET http://localhost:9200/test/employee/_search`
使用上述命令默认会返回前10条记录
返回值
```JSON
{
  "took": 131,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
  "hits": {
    "total": 3,
    "max_score": 1,
    "hits": [
      {
        "_index": "test",
        "_type": "employee",
        "_id": "2",
        "_score": 1,
        "_source": {
          "first_name": "Douglas",
          "last_name": "Fir",
          "age": 35,
          "about": "I like to build cabinets",
          "interests": [
            "forestry"
          ]
        }
      },
      {
        "_index": "test",
        "_type": "employee",
        "_id": "1",
        "_score": 1,
        "_source": {
          "first_name": "John",
          "last_name": "Smith",
          "age": 25,
          "about": "I love to go rock climbing",
          "interests": [
            "sports",
            "music"
          ]
        }
      },
      {
        "_index": "test",
        "_type": "employee",
        "_id": "3",
        "_score": 1,
        "_source": {
          "first_name": "Jane",
          "last_name": "Smith",
          "age": 367,
          "about": "I like to collect rock albums",
          "interests": [
            "music"
          ]
        }
      }
    ]
  }
}
```
hits：命中的文档记录
## 分页检索 ##
GET /_search?size=5
## 局部更新 ##
POST /website/blog/1/_update
{
"doc" : {
"tags" : [ "testing" ],
"views": 0
}
}
## 检索语法 ##
### 检索关键词返回所有字段 ###
`curl -XGET http://localhost:9200/test/employee/_search?q=last_name:Smith`
返回值
```JSON
{
  "took": 7,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
  "hits": {
    "total": 2,
    "max_score": 0.014065012,
    "hits": [
      {
        "_index": "test",
        "_type": "employee",
        "_id": "3",
        "_score": 0.014065012,
        "_source": {
          "first_name": "Jane",
          "last_name": "Smith",
          "age": 367,
          "about": "I like to collect rock albums",
          "interests": [
            "music"
          ]
        }
      },
      {
        "_index": "test",
        "_type": "employee",
        "_id": "1",
        "_score": 0.01125201,
        "_source": {
          "first_name": "John",
          "last_name": "Smith",
          "age": 25,
          "about": "I love to go rock climbing",
          "interests": [
            "sports",
            "music"
          ]
        }
      }
    ]
  }
}
```
last_name中包含Smith的文档都会被检索出来
### 根据id检索，返回指定字段 ###
`curl -GET http://localhost:9200/test/employee/1?_source=last_name,age`
指定返回last_name以及age字段，返回结果
```JSON
{
	"_index": "test",
	"_type": "employee",
	"_id": "1",
	"_version": 1,
	"found": true,
	"_source": {
		"last_name": "Smith",
		"age": 25
	}
}
```
## 批量操作 ##
试着去批量索引越来越多的文档。当性能开始下降的时候，就说明你的数据量太大了。一般比较好初始数量级是1000到5000个文档，或者你的文档很大，你就可以试着减小队列。 有的时候看看批量请求的物理大小是很有帮助的。1000个1KB的文档和1000个1MB的文档的差距将会是天差地别的。比较好的初始批量容量是5-15MB。
## 集群健康 ##
`curl -GET http://localhost:9200/_cluster/health`
```JSON
{
  "cluster_name": "elasticsearch",
  "status": "yellow",
  "timed_out": false,
  "number_of_nodes": 1,
  "number_of_data_nodes": 1,
  "active_primary_shards": 5,
  "active_shards": 5,
  "relocating_shards": 0,
  "initializing_shards": 0,
  "unassigned_shards": 5,
  "delayed_unassigned_shards": 0,
  "number_of_pending_tasks": 0,
  "number_of_in_flight_fetch": 0,
  "task_max_waiting_in_queue_millis": 0,
  "active_shards_percent_as_number": 50
}
```
status：集群状态，green代表所有主分片和从分片都可用；yellow代表所有主分片可用，但存在不可用的从分片；red代表存在不可用的主要分片
## 集群名词 ##
索引：只是一个逻辑命名空间(logical namespace)，它指向一个或多个分片(shards)
分片(shard)：工作单元(worker unit)底层的一员，它只负责保存索引中所有数据的一小片
## 创建一个索引 ##
`curl -PUT http://localhost:9200/blogs`
Body
```JSON
{
    "settings" : 
    {
        "number_of_shards" : 3,
        "number_of_replicas" : 1
    }
}
```
创建3个分片，每个分片1个从分片
需要注意索引的名称不能是小写，不能以下划线开头，不能包含逗号
返回值
```JSON
{
  "acknowledged": true
}
```
## 主分片从分片关系 ##
分片分为主分片(primary shard)以及从分片(replica shard) 两种。在你的索引中，每一个文档都属于一个主分片，所以具体有多少主分片取决于你的索引能存储多少数据
虽然理论上主分片对存储多少数据是没有限制的。分片的最大数量完全取决于你的实际状况：硬件的配置、文档的大小和复杂度、如何索引和查询你的文档，以及你期望的响应时间
分片用来分配集群中的数据。把分片想象成一个数据的容器。数据被存储在分片中，然后分片又被分配在集群的节点上。当你的集群扩展或者缩小时，elasticsearch会自动的在节点之间迁移分配分片，以便集群保持均衡
从分片只是主分片的一个副本，它用于提供数据的冗余副本，在硬件故障时提供数据保护，同时服务于搜索和检索这种只读请求
索引中的主分片的数量在索引创建后就固定下来了，但是从分片的数量可以随时改变
个人理解：从分片的功能不仅仅是备份作用，从分片对性能影响很大。比如创建1个blog索引，其中包含3个主分片，3个从分片，那么可以理解为一共有6个分片，可以构建6个物理机的集群，每个物理机可以实现一个分片的读写操作，这种情况分配超过6个物理机并不能加速性能。如果包含6个从分片，相当于每个主分片有两个分片，可以构建9个物理机的集群。
3个主分片，3个从分片情况下，如果分配6台物理机，每台物理机都拥有所有的数据，但是只有一个分片在运行，并且一个主分片的运行与计算被分配到两个物理机上，我是觉得，elasticsearch是根据哈希一致性或者id访问的物理机。例如检索关键词“学习”，假设“学习”这个关键词只存在于主分片1中，而主分片1可以同时在A，B两台机子上同时计算，最后将计算的结果合并，返回给客户端。
## 文档路由 ##
当你索引一个文档，它被保存在单个的主分片上，Elasticsearch如何知道文档属于哪个分片
呢？ 当我们创建一个新文档，它如何知道应该存储在分片1还是分片2上呢？
这个过程不能是随机的，因为我们将来需要取回该文档。 事实上，它是由一个非常简单的公
式来决定的:
分片 = hash(routing) % 主分片数量
routing 值可以是任何的字符串， 默认是文档的 id ，但也可以设置成一个自定义的值。routing 字符串被传递到一个哈希函数以生成一个数字，然后除以索引的主分片的数量 得到余数 remainder. 余数将总是在 0 到 主分片数量 - 1 之间, 它告诉了我们用以存放 一个特定文档的分片编号。
这解释了为什么主分片的数量只能在索引创建时设置、而且不能修改。 如果主分片的数量一旦在日后进行了修改，所有之前的路由值都会无效，文档再也无法被找到。

## 文档唯一性 ##
一篇文档可以通过**_index**，**_type**，**_id**去顶它的唯一性。我们可以自己提供一个_id，或者使用index API帮我们生成一个。
自定义id命令：
`curl -PUT localhost:9200/website/blog/123`
123：自定义id
需要使用PUT请求
自增id命令：
`curl -POST localhost:9200/website/blog`
需要使用POST请求
自增id是由22个字母组成，安全universally unique identifiers简称UUIDs。
## 文档元数据 ##
一个文档至少包含三个元数据_index：文档存储的地方；_type：文档代表的对象种类；_id：文档的唯一编号。
## 处理冲突 ##
当你使用 索引 API来更新一个文档时，我们先看到了原始文档，然后修改它，最后一次性地将整个新文档进行再次索引处理。Elasticsearch会根据请求发出的顺序来选择出最新的一个文档进行保存。但是，如果在你修改文档的同时其他人也发出了指令，那么他们的修改将会丢失。
很长时间以来，这其实都不是什么大问题。或许我们的主要数据还是存储在一个关系数据库中，而我们只是将为了可以搜索，才将这些数据拷贝到Elasticsearch中。或许发生多个人同时修改一个文件的概率很小，又或者这些偶然的数据丢失并不会影响到我们的正常使用。但是有些时候如果我们丢失了数据就会出大问题。想象一下，如果我们使用Elasticsearch来存储一个网店的商品数量。每当我们卖出一件，我们就会将这个数量减少一个。
ElasticSearch是分布式的，当文档被创建、更新或者删除时，新版本的文档就会被复制到集群中的其他节点上。Elasticsearch既是同步的又是异步的，也就是说复制的请求被平行发送出去，然后可能会混乱的到达目的地。
### 悲观并发控制（PCC）###
这一点在关系数据库中被广泛使用。假设这种情况很容易发生，我们就可以阻止对这一资源的访问。典型的例子就是当我们在读取一个数据前先锁定这一行，然后确保只有读取到数据的这个线程可以修改这一行数据。
### 乐观并发控制（OCC）###
Elasticsearch所使用的。假设这种情况并不会经常发生，也不会去阻止某一数据的访问。然而，如果基础数据在我们读取和写入的间隔中发生了变化，更新就会失败。这时候就由程序来决定如何处理这个冲突。例如，它可以重新读取新数据来进行更新，又或者它可以将这一情况直接反馈给用户。

