# [gitchennan/elasticsearch-analysis-lc-pinyin](https://github.com/gitchennan/elasticsearch-analysis-lc-pinyin) #
配置参数少，功能满足需求。

## 对应版本 ##
`elasticsearch2.3.2` 对应 `elasticsearch-analysis-lc-pinyin` 分支 `2.4.2.1` 或者 `2.2.2.1`

## 创建一个类型 ##
`elasticsearch-analysis-lc-pinyin` 的 `README` 是根据 `elasticsearch5.0` 编写的，给出的创建一个类型的语法如下
```curl
curl -XPOST http://localhost:9200/index/_mapping/brand -d'
{
  "brand": {
    "properties": {
      "name": {
        "type": "text",
        "analyzer": "lc_index",
        "search_analyzer": "lc_search",
        "term_vector": "with_positions_offsets"
      }
    }
  }
}'
```

[type=text 是 elasticsearch5.0 之后的类型](https://github.com/medcl/elasticsearch-analysis-ik/issues/276)，所以无法创建成功，稍作修改 `type=text`，使用如下语法创建一个类型
```curl
curl -XPOST http://localhost:9200/index/_mapping/brand -d'
{
  "brand": {
    "properties": {
      "name": {
        "type": "string",
        "analyzer": "lc_index",
        "search_analyzer": "lc_search",
        "term_vector": "with_positions_offsets"
      }
    }
  }
}'
```

`index` 索引结构如下
```json
{
  "index": {
    "aliases": {},
    "mappings": {
      "brand": {
        "properties": {
          "name": {
            "type": "string",
            "term_vector": "with_positions_offsets",
            "analyzer": "lc_index",
            "search_analyzer": "lc_search"
          }
        }
      }
    },
    "settings": {
      "index": {
        "creation_date": "1490152096129",
        "number_of_shards": "5",
        "number_of_replicas": "1",
        "uuid": "Lp1sSHGhQZyZ57LKO5KwRQ",
        "version": {
          "created": "2030299"
        }
      }
    },
    "warmers": {}
  }
}
```

存入几条数据
```curl
curl -XPOST http://localhost:9200/index/brand/1 -d'{"name":"百度"}'
curl -XPOST http://localhost:9200/index/brand/8 -d'{"name":"百度糯米"}'
curl -XPOST http://localhost:9200/index/brand/2 -d'{"name":"阿里巴巴"}'
curl -XPOST http://localhost:9200/index/brand/3 -d'{"name":"腾讯科技"}'
curl -XPOST http://localhost:9200/index/brand/4 -d'{"name":"网易游戏"}'
curl -XPOST http://localhost:9200/index/brand/9 -d'{"name":"大众点评"}'
curl -XPOST http://localhost:9200/index/brand/10 -d'{"name":"携程旅行网"}'
```

查出目前的所有数据
```json
http://localhost:9200/index/_search
{
  "took": 70,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
  "hits": {
    "total": 7,
    "max_score": 1,
    "hits": [
      {
        "_index": "index",
        "_type": "brand",
        "_id": "8",
        "_score": 1,
        "_source": {
          "name": "百度糯米"
        }
      },
      {
        "_index": "index",
        "_type": "brand",
        "_id": "9",
        "_score": 1,
        "_source": {
          "name": "大众点评"
        }
      },
      {
        "_index": "index",
        "_type": "brand",
        "_id": "10",
        "_score": 1,
        "_source": {
          "name": "携程旅行网"
        }
      },
      {
        "_index": "index",
        "_type": "brand",
        "_id": "2",
        "_score": 1,
        "_source": {
          "name": "阿里巴巴"
        }
      },
      {
        "_index": "index",
        "_type": "brand",
        "_id": "4",
        "_score": 1,
        "_source": {
          "name": "网易游戏"
        }
      },
      {
        "_index": "index",
        "_type": "brand",
        "_id": "1",
        "_score": 1,
        "_source": {
          "name": "百度"
        }
      },
      {
        "_index": "index",
        "_type": "brand",
        "_id": "3",
        "_score": 1,
        "_source": {
          "name": "腾讯科技"
        }
      }
    ]
  }
}
```

## 插件自带分词器 lc_index ##
原文：***lc_index : 该分词器用于索引数据时指定，将中文转换为全拼和首字，同时保留中文***
分词器分词效果
```json
curl -X POST -d '{
  "analyzer" : "lc_index",
  "text" : ["刘德华"]
}' "http://localhost:9200/lc/_analyze"

{
  "tokens": [
    {
      "token": "刘",
      "start_offset": 0,
      "end_offset": 1,
      "type": "word",
      "position": 0
    },
    {
      "token": "liu",
      "start_offset": 0,
      "end_offset": 1,
      "type": "word",
      "position": 0
    },
    {
      "token": "l",
      "start_offset": 0,
      "end_offset": 1,
      "type": "word",
      "position": 0
    },
    {
      "token": "德",
      "start_offset": 1,
      "end_offset": 2,
      "type": "word",
      "position": 1
    },
    {
      "token": "de",
      "start_offset": 1,
      "end_offset": 2,
      "type": "word",
      "position": 1
    },
    {
      "token": "d",
      "start_offset": 1,
      "end_offset": 2,
      "type": "word",
      "position": 1
    },
    {
      "token": "华",
      "start_offset": 2,
      "end_offset": 3,
      "type": "word",
      "position": 2
    },
    {
      "token": "hua",
      "start_offset": 2,
      "end_offset": 3,
      "type": "word",
      "position": 2
    },
    {
      "token": "h",
      "start_offset": 2,
      "end_offset": 3,
      "type": "word",
      "position": 2
    }
  ]
}
```

## 插件自带分词器 lc_search ##
原文：***lc_search: 该分词器用于拼音搜索时指定，按最小拼音分词个数拆分拼音，优先拆分全拼***
```json
curl -X POST -d '{
  "analyzer" : "lc_search",
  "text" : ["刘德华"]
}' "http://localhost:9200/index/_analyze"

{
  "tokens": [
    {
      "token": "刘",
      "start_offset": 0,
      "end_offset": 1,
      "type": "word",
      "position": 0
    },
    {
      "token": "德",
      "start_offset": 1,
      "end_offset": 2,
      "type": "word",
      "position": 1
    },
    {
      "token": "华",
      "start_offset": 2,
      "end_offset": 3,
      "type": "word",
      "position": 2
    }
  ]
}
```

## 拼音全拼 ##
搜索 `baidu`，结果正确
```json
curl -X POST -d '{
    "query": {
        "match": {
          "name": {
            "query": "baidu",
            "analyzer": "lc_search",
            "type": "phrase"
          }
        }
    },
    "highlight" : {
        "pre_tags" : ["<tag1>"],
        "post_tags" : ["</tag1>"],
        "fields" : {
            "name" : {}
        }
    }
}' "http://localhost:9200/index/brand/_search"

{
  "took": 4,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
  "hits": {
    "total": 2,
    "max_score": 1.4054651,
    "hits": [
      {
        "_index": "index",
        "_type": "brand",
        "_id": "8",
        "_score": 1.4054651,
        "_source": {
          "name": "百度糯米"
        },
        "highlight": {
          "name": [
            "<tag1>百度</tag1>糯米"
          ]
        }
      },
      {
        "_index": "index",
        "_type": "brand",
        "_id": "1",
        "_score": 0.38356602,
        "_source": {
          "name": "百度"
        },
        "highlight": {
          "name": [
            "<tag1>百度</tag1>"
          ]
        }
      }
    ]
  }
}
```

## 单字拼音全拼与中文混合 ##
搜索 `xie程lu行`，结果正确
```json
{
  "took": 11,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 2.459564,
    "hits": [
      {
        "_index": "index",
        "_type": "brand",
        "_id": "10",
        "_score": 2.459564,
        "_source": {
          "name": "携程旅行网"
        },
        "highlight": {
          "name": [
            "<tag1>携程旅行</tag1>网"
          ]
        }
      }
    ]
  }
}
```

## 单字拼音首字母与中文混合 ##
搜索 `携cl行`，结果正确
```json
curl -X POST -d '{
    "query": {
        "match": {
          "name": {
            "query": "携cl行",
            "analyzer": "lc_search",
            "type": "phrase"
          }
        }
    },
    "highlight" : {
        "pre_tags" : ["<tag1>"],
        "post_tags" : ["</tag1>"],
        "fields" : {
            "name" : {}
        }
    }
}' "http://localhost:9200/index/brand/_search"

{
  "took": 6,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 2.459564,
    "hits": [
      {
        "_index": "index",
        "_type": "brand",
        "_id": "10",
        "_score": 2.459564,
        "_source": {
          "name": "携程旅行网"
        },
        "highlight": {
          "name": [
            "<tag1>携程旅行</tag1>网"
          ]
        }
      }
    ]
  }
}
```

## 拼音首字母
搜索 `albb`，结果正确
```JSON
curl -X POST -d '{
    "query": {
        "match": {
          "name": {
            "query": "albb",
            "analyzer": "lc_search",
            "type": "phrase"
          }
        }
    },
    "highlight" : {
        "pre_tags" : ["<tag1>"],
        "post_tags" : ["</tag1>"],
        "fields" : {
            "name" : {}
        }
    }
}' "http://localhost:9200/index/brand/_search"

{
  "took": 4,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 2.828427,
    "hits": [
      {
        "_index": "index",
        "_type": "brand",
        "_id": "2",
        "_score": 2.828427,
        "_source": {
          "name": "阿里巴巴"
        },
        "highlight": {
          "name": [
            "<tag1>阿里巴巴</tag1>"
          ]
        }
      }
    ]
  }
}
```
## 结论 ##
`elasticsearch-analysis-lc-pinyin` 按照全拼、首字母，拼音中文混合搜索
