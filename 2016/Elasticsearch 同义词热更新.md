## 问题描述 ##
* 索引配置信息
```JAVA
{
   "test": {
      "aliases": {},
      "mappings": {
         "test": {
            "properties": {
               "text_1": {
                  "type": "string",
                  "analyzer": "ik"
               }
            }
         }
      },
      "settings": {
         "index": {
            "creation_date": "1482891562524",
            "analysis": {
               "filter": {
                  "remote_synonym": {
                     "type": "dynamic_synonym",
                     "synonyms_path": "http://IP:PORT/waf_file/files/sw",
                     "interval": "30"
                  }
               },
               "analyzer": {
                  "synonym": {
                     "filter": [
                        "remote_synonym"
                     ],
                     "tokenizer": "ik"
                  }
               }
            },
            "number_of_shards": "5",
            "number_of_replicas": "1",
            "uuid": "NMZ4fUryRXyoZ057lQrhDA",
            "version": {
               "created": "2030299"
            }
         }
      },
      "warmers": {}
   }
}
```

* 创建一条数据
```JAVA
PUT /test/test/1?pretty=1
{
   "text_1" : "水的密度很大"
}
```

* 使用如下语法，进行数次查询
```
GET /test/_search
{
    "query": {
        "query_string": {
           "default_field": "text_1",
           "analyzer": "ik", 
           "query": "i"
        }
    }
}
```

* 在同义词文件中新增同义词：`密度, density`

* 查询语法
```JAVA
GET /test/_search
{
    "query": {
        "query_string": {
           "default_field": "text_1",
           "query": "density"
        }
    }
}
```

* 可以查到文档
```JAVA
{
   "took": 1,
   "timed_out": false,
   "_shards": {
      "total": 5,
      "successful": 5,
      "failed": 0
   },
   "hits": {
      "total": 1,
      "max_score": 0.16609077,
      "hits": [
         {
            "_index": "test",
            "_type": "test",
            "_id": "15",
            "_score": 0.16609077,
            "_source": {
               "text_1": "水的密度很大"
            }
         }
      ]
   }
}
```
* 多次请求有很大概率无法检索到文档
```JAVA
{
   "took": 1,
   "timed_out": false,
   "_shards": {
      "total": 5,
      "successful": 5,
      "failed": 0
   },
   "hits": {
      "total": 0,
      "max_score": null,
      "hits": []
   }
}
```
### 做过如下尝试 ###
* shard = 1, replia = 1，不会出现上述问题
* shard =5, replia = 1，单机两个 `ES` 组成集群，问题依旧存在
* 重启`ES`，不会出现上述问题

## 解决过程 ##
主要是因为对 `Elasticsearch` 内部的运行原理完全不懂，所以只能够是瞎子摸象似的猜测。无法理解为什么多次查询的某一次会没有命中。也想不通为什么 shard = 1 时可以成功，挺莫名其妙的。

### 我觉得有必要将测试自动化 ###
我使用 `JEST API` 编写了测试用例，能够大大减少人工操作的效率以及能够让测试结果更稳定和明显。

### 同义词文件并没有被加载到 SynonymMap ###
这个概率很小，但是我觉得需要验证一下，这样之后的步骤才好进行。所以我先将程序中将读取的文本输出，确实同义词文本是被读取到内存中了。关于 `SynonymMap` 是否加载，没有直观的方法，只能够多输出一些相关的日志。很容易就证明是 `SynonymMap` 是正常的。

### 查询的顺序不对 ###
我想要排序是否是无论如何这个代码都不能够工作，还是说，只有这种情况比较特殊，可以将正常情况和这个情况做一些对比。我调整了实验的步骤为：
1. 创建文档
2. 新增同义词（等同义词完全被加载到内存之后在继续后边的操作）
3. 多次查询
依据这个步骤是很容易成功的，所以呢，这个代码在特定的顺序是可以正常运行的。

### 怀疑 DynamicSynonymFilter 类中的算法有问题 ###
该类中使用了一些算法，没有时间去深究，我在想会不会是算法的 `bug` 在某一次查询中没有正常的工作。我想这个代码应该是有出处的，说不定有更新。在网络上几经波折，总是是找到了，在 `lucene-analyzers-common-5.5.0.jar` 的 `package org.apache.lucene.analysis.synonym` 中的 `SynonymFilter` 类，经过对比，两者只有少许的编码习惯的差别，所以应该是没有问题的。

### 找出各个方法的调用步骤 ###
给所有的方法入口添加输出日志，并根据加上线程名，这样方便在 `Linux` 上做文本处理。对输出的日志做对比，看看哪些地方不同。经过处理，找到差异点的日志日下：
```JAVA
线程名：[T#26]247 一共输出了 27 行日志，并且可以看到同义词 被
[2017-01-17 14:19:31,153][ERROR][dynamic-synonym          ] elasticsearch[Frog-Man][search][T#26]247 DynamicSynonymFilter reset order:11815
[2017-01-17 14:19:31,153][ERROR][dynamic-synonym          ] elasticsearch[Frog-Man][search][T#26]247 DynamicSynonymFilter reset PendingInput.outputs:
[2017-01-17 14:19:31,153][ERROR][dynamic-synonym          ] elasticsearch[Frog-Man][search][T#26]247 PendingInput reset order:11821, term:
[2017-01-17 14:19:31,154][ERROR][dynamic-synonym          ] elasticsearch[Frog-Man][search][T#26]247 DynamicSynonymFilter reset PendingInput.outputs:
[2017-01-17 14:19:31,154][ERROR][dynamic-synonym          ] elasticsearch[Frog-Man][search][T#26]247 PendingInput reset order:11822, term:
[2017-01-17 14:19:31,154][ERROR][dynamic-synonym          ] elasticsearch[Frog-Man][search][T#26]247 DynamicSynonymFilter reset PendingOutputs.outputs:[被]
[2017-01-17 14:19:31,154][ERROR][dynamic-synonym          ] elasticsearch[Frog-Man][search][T#26]247 PendingOutputs reset order:11828
[2017-01-17 14:19:31,154][ERROR][dynamic-synonym          ] elasticsearch[Frog-Man][search][T#26]247 DynamicSynonymFilter reset PendingOutputs.outputs:[null]
[2017-01-17 14:19:31,154][ERROR][dynamic-synonym          ] elasticsearch[Frog-Man][search][T#26]247 PendingOutputs reset order:11833
[2017-01-17 14:19:31,154][ERROR][dynamic-synonym          ] elasticsearch[Frog-Man][search][T#26]247 DynamicSynonymFilter incrementToken order:11834
[2017-01-17 14:19:31,154][ERROR][dynamic-synonym          ] elasticsearch[Frog-Man][search][T#26]247 DynamicSynonymFilter parse order:11835
[2017-01-17 14:19:31,154][ERROR][dynamic-synonym          ] elasticsearch[Frog-Man][search][T#26]247 DynamicSynonymFilter parse scratchArc:[2 e], fst.outputs:ByteSequenceOutputs
[2017-01-17 14:19:31,154][ERROR][dynamic-synonym          ] elasticsearch[Frog-Man][search][T#26]247 DynamicSynonymFilter parse scratchArc:[2 e], fst.outputs:ByteSequenceOutputs
[2017-01-17 14:19:31,154][ERROR][dynamic-synonym          ] elasticsearch[Frog-Man][search][T#26]247 DynamicSynonymFilter count:0 rollBufferSize:2 rollIncr order:11841
[2017-01-17 14:19:31,154][ERROR][dynamic-synonym          ] elasticsearch[Frog-Man][search][T#26]247 DynamicSynonymFilter addOutput order:11849
[2017-01-17 14:19:31,154][ERROR][dynamic-synonym          ] elasticsearch[Frog-Man][search][T#26]247 DynamicSynonymFilter count:0 rollBufferSize:2 rollIncr order:11865
[2017-01-17 14:19:31,154][ERROR][dynamic-synonym          ] elasticsearch[Frog-Man][search][T#26]247 DynamicSynonymFilter count:0 rollBufferSize:2 rollIncr order:11866
[2017-01-17 14:19:31,154][ERROR][dynamic-synonym          ] elasticsearch[Frog-Man][search][T#26]247 DynamicSynonymFilter parse end
[2017-01-17 14:19:31,155][ERROR][dynamic-synonym          ] elasticsearch[Frog-Man][search][T#26]247 PendingInput reset order:11872, term:
[2017-01-17 14:19:31,155][ERROR][dynamic-synonym          ] elasticsearch[Frog-Man][search][T#26]247 DynamicSynonymFilter incrementToken order:11873
[2017-01-17 14:19:31,155][ERROR][dynamic-synonym          ] elasticsearch[Frog-Man][search][T#26]247 PendingInput reset order:11875, term:
[2017-01-17 14:19:31,155][ERROR][dynamic-synonym          ] elasticsearch[Frog-Man][search][T#26]247 PendingOutputs reset order:11876
[2017-01-17 14:19:31,155][ERROR][dynamic-synonym          ] elasticsearch[Frog-Man][search][T#26]247 PendingOutputs pullNext order:11877 pullNext:被
[2017-01-17 14:19:31,155][ERROR][dynamic-synonym          ] elasticsearch[Frog-Man][search][T#26]247 DynamicSynonymFilter count:0 rollBufferSize:2 rollIncr order:11878
[2017-01-17 14:19:31,155][ERROR][dynamic-synonym          ] elasticsearch[Frog-Man][search][T#26]247 DynamicSynonymFilter incrementToken order:11879
[2017-01-17 14:19:31,155][ERROR][dynamic-synonym          ] elasticsearch[Frog-Man][search][T#26]247 DynamicSynonymFilter parse order:11880
[2017-01-17 14:19:31,155][ERROR][dynamic-synonym          ] elasticsearch[Frog-Man][search][T#26]247 DynamicSynonymFilter parse end
```
相比较的日志：
```JAVA
线程名：[T#2]208 输出日志 18 行，并且看不到关键词 被
[2017-01-17 14:19:31,153][ERROR][dynamic-synonym          ] elasticsearch[Frog-Man][search][T#2]208 DynamicSynonymFilter reset order:11818
[2017-01-17 14:19:31,153][ERROR][dynamic-synonym          ] elasticsearch[Frog-Man][search][T#2]208 DynamicSynonymFilter reset PendingInput.outputs:
[2017-01-17 14:19:31,154][ERROR][dynamic-synonym          ] elasticsearch[Frog-Man][search][T#2]208 PendingInput reset order:11830, term:
[2017-01-17 14:19:31,154][ERROR][dynamic-synonym          ] elasticsearch[Frog-Man][search][T#2]208 DynamicSynonymFilter reset PendingInput.outputs:
[2017-01-17 14:19:31,154][ERROR][dynamic-synonym          ] elasticsearch[Frog-Man][search][T#2]208 PendingInput reset order:11831, term:
[2017-01-17 14:19:31,154][ERROR][dynamic-synonym          ] elasticsearch[Frog-Man][search][T#2]208 DynamicSynonymFilter reset PendingOutputs.outputs:[null]
[2017-01-17 14:19:31,154][ERROR][dynamic-synonym          ] elasticsearch[Frog-Man][search][T#2]208 PendingOutputs reset order:11842
[2017-01-17 14:19:31,154][ERROR][dynamic-synonym          ] elasticsearch[Frog-Man][search][T#2]208 DynamicSynonymFilter reset PendingOutputs.outputs:[null]
[2017-01-17 14:19:31,154][ERROR][dynamic-synonym          ] elasticsearch[Frog-Man][search][T#2]208 PendingOutputs reset order:11843
[2017-01-17 14:19:31,154][ERROR][dynamic-synonym          ] elasticsearch[Frog-Man][search][T#2]208 DynamicSynonymFilter incrementToken order:11844
[2017-01-17 14:19:31,154][ERROR][dynamic-synonym          ] elasticsearch[Frog-Man][search][T#2]208 DynamicSynonymFilter parse order:11845
[2017-01-17 14:19:31,154][ERROR][dynamic-synonym          ] elasticsearch[Frog-Man][search][T#2]208 DynamicSynonymFilter count:0 rollBufferSize:2 rollIncr order:11846
[2017-01-17 14:19:31,154][ERROR][dynamic-synonym          ] elasticsearch[Frog-Man][search][T#2]208 DynamicSynonymFilter parse end
[2017-01-17 14:19:31,154][ERROR][dynamic-synonym          ] elasticsearch[Frog-Man][search][T#2]208 PendingInput reset order:11868, term:
[2017-01-17 14:19:31,154][ERROR][dynamic-synonym          ] elasticsearch[Frog-Man][search][T#2]208 DynamicSynonymFilter count:0 rollBufferSize:2 rollIncr order:11869
[2017-01-17 14:19:31,154][ERROR][dynamic-synonym          ] elasticsearch[Frog-Man][search][T#2]208 DynamicSynonymFilter incrementToken order:11870
[2017-01-17 14:19:31,155][ERROR][dynamic-synonym          ] elasticsearch[Frog-Man][search][T#2]208 DynamicSynonymFilter parse order:11884
[2017-01-17 14:19:31,155][ERROR][dynamic-synonym          ] elasticsearch[Frog-Man][search][T#2]208 DynamicSynonymFilter parse end
```
从上述日志可以证明，确实是查询的某一个线程处理过程不对导致。

### 有部分引用的 SynonymMap 对象没有更新 ###
类 `DynamicSynonymTokenFilterFactory` 中定义成员变量 `private Map<DynamicSynonymFilter, Integer> dynamicSynonymFilters = new WeakHashMap<DynamicSynonymFilter, Integer>();`。一开始我看到 `WeakHashMap` 就觉得很奇特，为什么使用这个 `Map`？我想应该是担心内存泄露，特意使用这个 `Map` 吧。
经过几轮的猜测与推理，我觉得出现问题的代码可能是下面这个方法：
```JAVA
@Override
public TokenStream create(TokenStream tokenStream) {
	DynamicSynonymFilter dynamicSynonymFilter = new DynamicSynonymFilter(
			tokenStream, synonymMap, ignoreCase);
	dynamicSynonymFilters.put(dynamicSynonymFilter, 1);

	// fst is null means no synonyms
	return synonymMap.fst == null ? tokenStream : dynamicSynonymFilter;
}
```
因为对这个方法是如何被 `Elasticsearch` 调用并不了解，但是通过添加输出日志，我们知道了，这个类是并发被访问的，这么说来 `WeakHashMap` 并不支持并发，所以应该是有问题的。因此修改成员变量的类型为：`private Map<DynamicSynonymFilter, Integer> dynamicSynonymFilters = new ConcurrentHashMap<DynamicSynonymFilter, Integer>();`。但是非常奇怪，没有奇效，本着小心为上的想法，我将 `DynamicSynonymFilters` 对象的大小输出来了，发现大小和进入 `create()` 方法的顺序并不一致。
我通过添加多个日志，查看了 `key` 是否冲突之类，但是还是没有发现问题的根源，我不知道为什么，我转而使用 `SynchronizedList` 对象，并将对应部分的代码做了修改，再次运行测试用例，成功解决。困扰了一周的问题总算是解决了。

## 总结 ##
最重要是对 `Elasticsearch` 的运行机制不了解，如果了解，单单两个类，代码量也很小，相信是能够很快找出问题的。还有就是没能够搭建出 `Elasticsearch` 的 `debug` 环境，如果能够手动调试，我想问题也能够很快的解决。在使用被人的东西时，得要先做全面的测试，不要到生产，或者其他的时候再去处理，那样反而会得不偿失，越早发现错误，才能够更快的解决。






