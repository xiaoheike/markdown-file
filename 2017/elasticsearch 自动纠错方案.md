# es suggest did you mean 资料 #
## [term suggester 功能介绍](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-suggesters.html)
`term suggester` 根据提供的文档提供搜索关键词的建议，也就是关键词自动纠错。该链接介绍如何使用 `term suggester` 语法。`term suggester` 是支持中文的，必须非常小心参数 `min_word_length`，默认值为 `4`，是指推荐词的长度大于 `4` 才会被显示，设置小一些能够开到效果（本人就被这个参数坑了，误以为 `term suggester` 不支持中文，绕了一大圈）。
### 本人使用的查询语法 ###
```json
{
	"from": 0,
	"size": 0,
	"suggest": {
		"didyoumean": {
			"text": "长安城北京城",
			"term": {
				"field": "search_text_new",
				"analyzer": "ik_smart",
				"size": 5,
				"suggest_mode": "always",
				"min_word_length": 2
			}
		}
	}
}

结果：
{
   "took": 32,
   "timed_out": false,
   "_shards": {
      "total": 5,
      "successful": 5,
      "failed": 0
   },
   "hits": {
      "total": 2267687,
      "max_score": 0,
      "hits": []
   },
   "suggest": {
      "didyoumean": [
         {
            "text": "长安城",
            "offset": 0,
            "length": 3,
            "options": [
               {
                  "text": "长安街",
                  "score": 0.6666666,
                  "freq": 2
               },
               {
                  "text": "长安",
                  "score": 0.5,
                  "freq": 256
               }
            ]
         },
         {
            "text": "北京城",
            "offset": 3,
            "length": 3,
            "options": [
               {
                  "text": "北京人",
                  "score": 0.6666666,
                  "freq": 89
               },
               {
                  "text": "北京大",
                  "score": 0.6666666,
                  "freq": 68
               }
            ]
         }
      ]
   }
}
```
## [term suggester 参数](https://www.elastic.co/guide/en/elasticsearch/reference/2.3/search-suggesters-term.html) ##
`term suggester` 用到的一些参数及说明。

## [phrase suggester](https://www.elastic.co/guide/en/elasticsearch/reference/2.3/search-suggesters-phrase.html) ##
`phrase Suggester` 也是提供关键词自动纠错功能，是 `term suggester` 的升级版。

## [completion suggester](https://www.elastic.co/guide/en/elasticsearch/reference/2.3/search-suggesters-completion.html) ##
`Completion Suggester` 前缀匹配，不具有像 `term` 以及 `phrase` 关键词的自动纠错功能，是一种自动补全功能。

## [completion suggester 中文使用示例](http://www.tcao.net/article/86.html) ##

# 测试 #
## create mapping ##
```http
PUT /index/_mapping/suggest
{
  "suggest": {
    "properties": {
      "description": {
        "type": "string",
		    "analyzer": "ik"
      },
      "did_you_mean": {
        "type": "string",
        "analyzer": "ik"
      },
      "title": {
        "type": "string",
		    "analyzer": "ik"
      }
    }
  }
}
```

## 存入词条 ##
```json
POST localhost:9200/index/suggest/1
{
  "title": "西施",
  "description": "在当时明朝的北京长安街，有一对夫妇开了一家豆腐店",
  "did_you_mean": "西施 在当时明朝的北京长安街，有一对夫妇开了一家豆腐店"
}

{
  "title": "丧事",
  "description": "中国古代的江南农村，有人去世后，一般用豆腐这类素食招待来帮忙的人",
  "did_you_mean": "丧事 中国古代的江南农村，有人去世后，一般用豆腐这类素食招待来帮忙的人"
}

{
  "title": "含义",
  "description": "可以确定的是豆腐这种独特的中华美食，千百年来在丰富中国人物质生活的同时，也在不断添加着新的文化内涵",
  "did_you_mean": "含义 可以确定的是豆腐这种独特的中华美食，千百年来在丰富中国人物质生活的同时，也在不断添加着新的文化内涵"
}
```

## 搜索 ##
尝试 `北` 能否匹配到 `北京`
```json
POST /index/_search
{
  "suggest": {
    "didYouMean": {
      "text": "北",
      "phrase": {
        "field": "did_you_mean"
      }
    }
  },
  "query": {
    "multi_match": {
      "query": "北",
      "fields": [
        "description",
        "title"
      ]
    }
  }
}
```

### 结果 ###
```json
{
  "took": 2,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "failed": 0
  },
  "hits": {
    "total": 0,
    "max_score": null,
    "hits": []
  },
  "suggest": {
    "didYouMean": [
      {
        "text": "北",
        "offset": 0,
        "length": 1,
        "options": []
      }
    ]
  }
}
```

## term 第二次 ##
```json
{
  "index2": {
    "aliases": {},
    "mappings": {
      "suggest": {
        "properties": {
          "description": {
            "type": "string",
            "analyzer": "ik"
          },
          "did_you_mean": {
            "type": "string",
            "store": true,
            "term_vector": "with_positions_offsets",
            "analyzer": "ik",
            "include_in_all": true
          },
          "title": {
            "type": "string",
            "analyzer": "ik"
          }
        }
      }
    },
    "settings": {
      "index": {
        "creation_date": "1491016117726",
        "number_of_shards": "1",
        "number_of_replicas": "1",
        "uuid": "zsfJIEzoTmO6VdU9iqWRlw",
        "version": {
          "created": "2030299"
        }
      }
    },
    "warmers": {}
  }
}
```

### 存入词条 ###
```json
POST localhost:9200/index2/suggest/1
{
  "title": "西施",
  "description": "在当时明朝的北京长安街，有一对夫妇开了一家豆腐店",
  "did_you_mean": "西施 在当时明朝的北京长安街，有一对夫妇开了一家豆腐店"
}

{
  "title": "丧事",
  "description": "中国古代的江南农村，有人去世后，一般用豆腐这类素食招待来帮忙的人",
  "did_you_mean": "丧事 中国古代的江南农村，有人去世后，一般用豆腐这类素食招待来帮忙的人"
}

{
  "title": "含义",
  "description": "可以确定的是豆腐这种独特的中华美食，千百年来在丰富中国人物质生活的同时，也在不断添加着新的文化内涵",
  "did_you_mean": "含义 可以确定的是豆腐这种独特的中华美食，千百年来在丰富中国人物质生活的同时，也在不断添加着新的文化内涵"
}
```

### 搜索 ###
```json
localhost:9200/index2/_search
{
  "suggest": {
    "didYouMean": {
      "text": "北京城长安城",
      "term": {
        "field": "did_you_mean",
		"min_word_length": 1
		}
    }
  },
  "query": {
    "multi_match": {
      "query": "北京城长安城",
      "fields": [
        "description",
        "title"
      ]
    }
  }
}

{
  "took": 10,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 0.0701967,
    "hits": [
      {
        "_index": "index2",
        "_type": "suggest",
        "_id": "1",
        "_score": 0.0701967,
        "_source": {
          "title": "西施",
          "description": "在当时明朝的北京长安街，有一对夫妇开了一家豆腐店",
          "did_you_mean": "西施 在当时明朝的北京长安街，有一对夫妇开了一家豆腐店"
        }
      }
    ]
  },
  "suggest": {
    "didYouMean": [
      {
        "text": "北京城",
        "offset": 0,
        "length": 3,
        "options": [
          {
            "text": "北京",
            "score": 0.5,
            "freq": 1
          }
        ]
      },
      {
        "text": "北京",
        "offset": 0,
        "length": 2,
        "options": []
      },
      {
        "text": "京城",
        "offset": 1,
        "length": 2,
        "options": []
      },
      {
        "text": "京",
        "offset": 1,
        "length": 1,
        "options": []
      },
      {
        "text": "城",
        "offset": 2,
        "length": 1,
        "options": []
      },
      {
        "text": "长安城",
        "offset": 3,
        "length": 3,
        "options": [
          {
            "text": "长安街",
            "score": 0.6666666,
            "freq": 1
          },
          {
            "text": "长安",
            "score": 0.5,
            "freq": 1
          }
        ]
      },
      {
        "text": "长安",
        "offset": 3,
        "length": 2,
        "options": []
      },
      {
        "text": "城",
        "offset": 5,
        "length": 1,
        "options": []
      }
    ]
  }
}
```

### search with ik_smart ###
```json
POST localhost:9200/index2/_search
{
  "suggest": {
    "didYouMean": {
      "text": "北京城长安城",
      "term": {
        "field": "did_you_mean",
		"min_word_length": 1,
		"analyzer" : "ik_smart"
		}
    }
  },
  "query": {
    "multi_match": {
      "query": "北京城长安城",
      "fields": [
        "description",
        "title"
      ]
    }
  }
}

{
  "took": 6,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 0.0701967,
    "hits": [
      {
        "_index": "index2",
        "_type": "suggest",
        "_id": "1",
        "_score": 0.0701967,
        "_source": {
          "title": "西施",
          "description": "在当时明朝的北京长安街，有一对夫妇开了一家豆腐店",
          "did_you_mean": "西施 在当时明朝的北京长安街，有一对夫妇开了一家豆腐店"
        }
      }
    ]
  },
  "suggest": {
    "didYouMean": [
      {
        "text": "北京城",
        "offset": 0,
        "length": 3,
        "options": [
          {
            "text": "北京",
            "score": 0.5,
            "freq": 1
          }
        ]
      },
      {
        "text": "长安城",
        "offset": 3,
        "length": 3,
        "options": [
          {
            "text": "长安街",
            "score": 0.6666666,
            "freq": 1
          },
          {
            "text": "长安",
            "score": 0.5,
            "freq": 1
          }
        ]
      }
    ]
  }
}
```

## es did you mean 可行方案 ##
## [complete suggest](https://www.elastic.co/guide/en/elasticsearch/reference/2.3/search-suggesters-completion.html) ##
### create mapping ###
```json
PUT localhost:9200/medcl/
{
	"mappings": {
      "song": {
        "properties": {
          "name": {
            "type": "string"
          },
          "name_full_pinyin": {
            "type": "completion",
            "analyzer": "full_pinyin_analyzer",
            "payloads": true,
            "preserve_separators": false,
            "preserve_position_increments": true,
            "max_input_length": 50
          },
          "name_prefix_pinyin": {
            "type": "completion",
            "analyzer": "prefix_pinyin_analyzer",
            "payloads": true,
            "preserve_separators": false,
            "preserve_position_increments": true,
            "max_input_length": 50
          }
        }
      }
    },
    "settings": {
      "index": {
        "analysis": {
          "filter": {
            "full_pinyin": {
              "padding_char": "",
              "type": "pinyin",
              "first_letter": "none"
            },
            "prefix_pinyin": {
              "padding_char": "",
              "type": "pinyin",
              "first_letter": "prefix"
            }
          },
          "analyzer": {
            "full_pinyin_analyzer": {
              "filter": [
                "lowercase",
                "full_pinyin"
              ],
              "tokenizer": "keyword"
            },
            "prefix_pinyin_analyzer": {
              "filter": [
                "lowercase",
                "prefix_pinyin"
              ],
              "tokenizer": "keyword"
            }
          },
          "tokenizer": {
            "prefix_pinyin": {
              "padding_char": "",
              "type": "pinyin",
              "first_letter": "prefix"
            }
          }
        }
      }
    }
}
```

### create doc ##
创建 4 个文档
```json
PUT /medcl/song/1
{
    "name" : "白衣飘飘的年代",
    "name_full_pinyin" : {
        "input": [ "白衣飘飘的年代full" ],
        "output": "白衣飘飘的年代full",
        "weight" : 34
    },
    "name_prefix_pinyin" : {
        "input": [ "白衣飘飘的年代prefix" ],
        "output": "白衣飘飘的年代prefix",
        "weight" : 34
    }
}

PUT /medcl/song/2
{
    "name" : "郎心似铁",
    "name_full_pinyin" : {
        "input": [ "郎心似铁full" ],
        "output": "郎心似铁full",
        "weight" : 34
    },
    "name_prefix_pinyin" : {
        "input": [ "郎心似铁prefix" ],
        "output": "郎心似铁prefix",
        "weight" : 34
    }
}


PUT /medcl/song/3
{
    "name" : "流浪歌手的请人",
    "name_full_pinyin" : {
        "input": [ "流浪歌手的请人full" ],
        "output": "流浪歌手的请人full",
        "weight" : 34
    },
    "name_prefix_pinyin" : {
        "input": [ "流浪歌手的请人prefix" ],
        "output": "流浪歌手的请人prefix",
        "weight" : 34
    }
}

PUT /medcl/song/4
{
    "name" : "搜索建议，自动补全搜索结结果",
    "name_full_pinyin" : {
        "input": [ "搜索建议，自动补全搜索结结果full" ],
        "output": "搜索建议，自动补全搜索结结果full",
        "weight" : 34
    },
    "name_prefix_pinyin" : {
        "input": [ "搜索建议，自动补全搜索结结果prefix" ],
        "output": "搜索建议，自动补全搜索结结果prefix",
        "weight" : 34
    }
}
```

### search ###
**搜索 `流浪歌迷`**
```json
POST localhost:9200/medcl/_suggest
{
    "goods_full_suggest" : {
        "text" : "流浪歌迷",
        "completion" : {
            "field" : "name_full_pinyin",
            "fuzzy" : {
            	"fuzziness" : 2,
            	"size" : 1,
            	"unicode_aware" : true,
            	"transpositions" : true
            }
        }
    }
}

{
  "_shards": {
    "total": 1,
    "successful": 1,
    "failed": 0
  },
  "goods_full_suggest": [
    {
      "text": "流浪歌迷",
      "offset": 0,
      "length": 4,
      "options": [
        {
          "text": "流浪歌手的请人full",
          "score": 34
        }
      ]
    }
  ]
}
```

**搜索 `流浪歌星`**
```json
POST localhost:9200/medcl/_suggest
{
    "goods_full_suggest" : {
        "text" : "流浪歌星",
        "completion" : {
            "field" : "name_full_pinyin",
            "fuzzy" : {
            	"fuzziness" : 2,
            	"size" : 1,
            	"unicode_aware" : true,
            	"transpositions" : true
            }
        }
    }
}

{
  "_shards": {
    "total": 1,
    "successful": 1,
    "failed": 0
  },
  "goods_full_suggest": [
    {
      "text": "流浪歌星",
      "offset": 0,
      "length": 4,
      "options": []
    }
  ]
}
```

**分词器分析 `流浪歌手的请人full`**
```json
POST localhost:9200/medcl/_analyze
{
  "analyzer" : "full_pinyin_analyzer",
  "text" : ["流浪歌手的请人full"]
}

{
  "tokens": [
    {
      "token": "liulanggeshoudeqingrenfull",
      "start_offset": 0,
      "end_offset": 11,
      "type": "word",
      "position": 0
    }
  ]
}
```

### 总结 ###
- complete suggest 是前缀匹配
- 中文使用，是先将中文转为拼音，在对比拼音之间的差异。`流浪歌迷`--> `liulanggemi`，`流浪歌星`-->`liulanggexing`，在与 `流浪歌手的请人full(liulanggeshoudeqingrenfull)` 做比较
- 字符串模糊匹配(fuzziness)最大长度为 2。所以 `流浪歌迷` 能够匹配 `流浪歌手的请人full(liulanggeshoudeqingrenfull)`，而 `流浪歌星` 则不能够匹配 `流浪歌手的请人full(liulanggeshoudeqingrenfull)`
