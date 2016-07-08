## 请求 URL ##
```
172.24.133.44:8080/v0.6/assets/actions/titan?coverage=Org/nd/&coverage=User/194152/&coverage=Debug/qa/&include=CG,CR,EDU,LC&limit=(0,20)&relation=chapters/cb6e99c0-e7cc-40b8-9f68-ffec60ed445f/&words
```
## 基础测试测试 ##
titan-cassandra-es.properties 文件配置，性能相关的是默认配置：
```
gremlin.graph=com.thinkaurelius.titan.core.TitanFactory
storage.backend=cassandrathrift

storage.hostname=debug.pre.cassandra.sdp
storage.username=prepub_cassandra_lcms
storage.password=Rs4rphSx5XqO
storage.cassandra.keyspace=prepub_cassandra_lcms

cache.db-cache = true
cache.db-cache-clean-wait = 20

cache.db-cache-time = 180000

cache.db-cache-size = 0.2

index.search.backend=elasticsearch
index.search.elasticsearch.cluster-name = es1.5.1-titan
index.search.index-name = titan_integration

index.search.hostname=172.24.133.44:9306

index.search.elasticsearch.client-only=true
index.search.elasticsearch.ext.script.disable_dynamic=false
```

Jmeter 200 并发 15分钟的测试结果：
![titan 使用默认配置并且es缓存没有打开](http://i.imgur.com/CHLXMDS.png)

## 打开 es 缓存 ##
开启 es 缓存可以使用命令的方式，命令格式如下：
```
#默认未开启，启用缓存：
curl -XPUT localhost:9200/titan_integration/_settings -d'
{ "index.cache.query.enable": true }'
#修改缓存大小：
indices.cache.query.size: 10%
```

JMeter 200 并发 15分钟的测试结果：
![titan 使用默认配置并且es 缓存没有打开](http://i.imgur.com/ovqpamj.png)

吞吐量提升将近 1 倍

## 修改 es 缓存大小 ##
配置文件 config/elasticsearch.yml 添加如下参数：
```
#修改缓存大小：
indices.cache.query.size: 10%
```

JMeter 200 并发 15分钟的测试结果：
![titan 使用默认配置并且es 缓存没有打开大小调整为10%](http://i.imgur.com/iiMfVwQ.png)

从吞吐量上来看，略有提升。
## 关闭 titan 缓存 ##
由于 titan 的缓存开启之后的效果也不是很明显，所以大家觉得是 cache 没有起作用，我是觉得缓存已经起作用了，但是还是需要以事实说话，我做了如下测试，关闭缓存 --》 压测 --》 结果。如果关闭缓存性能和之前没有差别或者性能反而更好了，那说明缓存确实是有问题的。
所以我关闭缓存，修改了 conf/titan-cassandra-es.properties 文件的如下配置：
```
# 关闭缓存
cache.db-cache = false 
```

JMeter 200 并发 15分钟的测试结果：
![titan 缓存关闭es 缓存开启](http://i.imgur.com/zdz7c41.png)
实际上压测不到10s 就已经有大量的错误出现了，说明 titan 的缓存是起作用的，但是不知道为什么缓存之后的查询还需要 300-400ms。按理缓存之后立即再次查询，经过缓存绝对是 10ms 以内，不知道为什么 titan 的缓存这么弱。

## 修改 cache.tx-cache-size 参数 ##
验证顶点缓存是否起作用。官方文档确实写的比较含糊，无法具体理解这个缓存有什么作用。cache.tx-cache-size 的默认值为 20000，以下内容都需要修改 titan-cassandra-es.properties 文件。
### cache.tx-cache-size=40000 ###
JMeter 200 并发 15分钟的测试结果：
![titan 顶点事务缓存值为40000](http://i.imgur.com/oECZVG0.jpg)

### cache.tx-cache-size=200000 ###
JMeter 200 并发 15分钟的测试结果:
![titan 顶点事务缓存值为200000](http://i.imgur.com/nHIZ0AK.jpg)

### cache.tx-cache-size=2000 ###
JMeter 200 并发 15分钟的测试结果:
![titan 顶点事务缓存值为2000](http://i.imgur.com/UbEjcjC.jpg)

### cache.tx-cache-size=20000 ###
JMeter 200 并发 15分钟的测试结果:
![titan 顶点事务缓存值为20000](http://i.imgur.com/s665LYV.jpg)

### cache.tx-cache-size 不定义，使用默认设置 ###
JMeter 200 并发 15分钟的测试结果:
![titan 顶点事务缓存不定义使用默认设置](http://i.imgur.com/XUfYaHU.jpg)
### 结论 ###
经过上边的测试可以顶点事务缓存大小的修改是没有作用的，因为这个顶点缓存只能够在同一个事务中被使用到，使用的场景应该在遍历顶点的时候。
正确的使用方法，是在同一个事务中，同一个查询，感觉局限性也比较大，因为两次的查询必须是一模一样的才能够使用顶点缓存。单个查询语句使用 titan 缓存之后需要 300-400ms 返回结果。我一条语句在一个事务中执行两次，第二次查询的时间如下所示：
![titan 顶点事务的使用方式](http://i.imgur.com/2HgIFTS.jpg)

