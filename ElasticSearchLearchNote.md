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
DELETE：删除该文档
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



