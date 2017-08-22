## [elasticsearch-analysis-pinyin 2.x 分支](https://github.com/medcl/elasticsearch-analysis-pinyin/tree/2.x) ##
`github` 项目 `elasticsearch-analysis-pinyin 2.x 分支` 是为 `elasticsearch 2.x` 服务，经过测试 `elasticsearch 2.3.2` 也可以使用该插件。
## 官方文档中的说明 ##
- remove_duplicated_term when this option enabled, duplicated term will be removed to save index, eg: de的>de, default: false, NOTE: position related query maybe influenced
- keep_first_letter when this option enabled, eg: 刘德华>ldh, default: true
- keep_separate_first_letter when this option enabled, will keep first letters separately, eg: 刘德华>l,d,h, default: false, NOTE: query result maybe too fuzziness due to term too frequency
- limit_first_letter_length set max length of the first_letter result, default: 16 (首字母组成的拼音的最大长度，超过部分会被舍弃)
- keep_full_pinyin when this option enabled, eg: 刘德华> [liu,de,hua], default: true
- keep_joined_full_pinyin when this option enabled, eg: 刘德华> [liudehua], default: false
- keep_none_chinese keep non chinese letter or number in result, default: true
- keep_none_chinese_together keep non chinese letter together, default: true, eg: DJ音乐家 -> DJ,yin,yue,jia, when set to false, eg: DJ音乐家 -> D,J,yin,yue,jia, NOTE: keep_none_chinese should be enabled first
- keep_none_chinese_in_first_letter keep non Chinese letters in first letter, eg: 刘德华AT2016->ldhat2016, default: true
- none_chinese_pinyin_tokenize break non chinese letters into separate pinyin term if they are pinyin, default: true, eg: liudehuaalibaba13zhuanghan -> liu,de,hua,a,li,ba,ba,13,zhuang,han, NOTE: keep_none_chinese and keep_none_chinese_together should be enabled first
- keep_original when this option enabled, will keep original input as well, default: false
- lowercase lowercase non Chinese letters, default: true
- trim_whitespace default: true

## 基准配置 ##
基准配置参数
```json
"keep_joined_full_pinyin": "false",
"lowercase": "true",
"keep_original": "false",
"keep_none_chinese_together": "true",
"remove_duplicated_term": "false",
"keep_first_letter": "true",
"keep_separate_first_letter": "false",
"trim_whitespace": "true",
"keep_none_chinese": "true",
"limit_first_letter_length": "16",
"keep_full_pinyin": "true"
```

创建索引与分词器
```json
curl -X POST -d '{
	"mappings": {
		"folk": {
			"properties": {
			   "text": {
				  "type": "string",
				  "analyzer": "pinyin_analyzer"
			   }
			}
		}
	},
	"settings": {
			"index" : {
	        "analysis" : {
	            "analyzer" : {
	                "pinyin_analyzer" : {
	                    "tokenizer" : "my_pinyin"
	                    }
	            },
	            "tokenizer" : {
	                "my_pinyin" : {
	                    "type" : "pinyin",
          						"remove_duplicated_term" : false,
          						"keep_joined_full_pinyin" : false,
	                    "keep_separate_first_letter" : false,
          						"keep_first_letter" : true,
          						"limit_first_letter_length" : 16,
	                    "keep_full_pinyin" : true,
	                    "keep_original" : true,
	                    "keep_none_chinese" : true,
						          "keep_none_chinese_together" : true,
	                    "lowercase" : true,
						          "trim_whitespace" : true
	                }
	            }
	        }
	    }
	}
}' "http://localhost:9200/medcl20"
```

生成索引结构
```json
curl -X GET "http://localhost:9200/medcl20"
{
  "medcl20": {
    "aliases": {},
    "mappings": {
      "folk": {
        "properties": {
          "text": {
            "type": "string",
            "analyzer": "pinyin_analyzer"
          }
        }
      }
    },
    "settings": {
      "index": {
        "creation_date": "1490170676090",
        "analysis": {
          "analyzer": {
            "pinyin_analyzer": {
              "tokenizer": "my_pinyin"
            }
          },
          "tokenizer": {
            "my_pinyin": {
              "keep_joined_full_pinyin": "false",
              "lowercase": "true",
              "keep_original": "true",
              "keep_none_chinese_together": "true",
              "remove_duplicated_term": "false",
              "keep_first_letter": "true",
              "keep_separate_first_letter": "false",
              "trim_whitespace": "true",
              "type": "pinyin",
              "keep_none_chinese": "true",
              "limit_first_letter_length": "16",
              "keep_full_pinyin": "true"
            }
          }
        },
        "number_of_shards": "5",
        "number_of_replicas": "1",
        "uuid": "31Y9PizQQ2KQn_Fl6bpPNw",
        "version": {
          "created": "2030299"
        }
      }
    },
    "warmers": {}
  }
}
```

分词器分词效果
```json
curl -X POST -d '{
  "analyzer" : "pinyin_analyzer",
  "text" : ["刘德华"]
}' "http://localhost:9200/medcl20/_analyze"

{
  "tokens": [
    {
      "token": "liu",
      "start_offset": 0,
      "end_offset": 1,
      "type": "word",
      "position": 0
    },
    {
      "token": "de",
      "start_offset": 1,
      "end_offset": 2,
      "type": "word",
      "position": 1
    },
    {
      "token": "hua",
      "start_offset": 2,
      "end_offset": 3,
      "type": "word",
      "position": 2
    },
    {
      "token": "刘德华",
      "start_offset": 0,
      "end_offset": 3,
      "type": "word",
      "position": 3
    },
    {
      "token": "ldh",
      "start_offset": 0,
      "end_offset": 3,
      "type": "word",
      "position": 4
    }
  ]
}
```

### keep_original ###
#### keep_original = true ####
```json
curl -X POST -d '{
  "analyzer" : "pinyin_analyzer",
  "text" : ["刘德华"]
}' "http://localhost:9200/medcl20/_analyze"

{
  "tokens": [
    {
      "token": "liu",
      "start_offset": 0,
      "end_offset": 1,
      "type": "word",
      "position": 0
    },
    {
      "token": "de",
      "start_offset": 1,
      "end_offset": 2,
      "type": "word",
      "position": 1
    },
    {
      "token": "hua",
      "start_offset": 2,
      "end_offset": 3,
      "type": "word",
      "position": 2
    },
    {
      "token": "刘德华",
      "start_offset": 0,
      "end_offset": 3,
      "type": "word",
      "position": 3
    },
    {
      "token": "ldh",
      "start_offset": 0,
      "end_offset": 3,
      "type": "word",
      "position": 4
    }
  ]
}
```

#### keep_original = false ####
```json
curl -X POST -d '{
  "analyzer" : "pinyin_analyzer",
  "text" : ["刘德华"]
}' "http://localhost:9200/medcl20/_analyze"

{
  "tokens": [
    {
      "token": "liu",
      "start_offset": 0,
      "end_offset": 1,
      "type": "word",
      "position": 0
    },
    {
      "token": "de",
      "start_offset": 1,
      "end_offset": 2,
      "type": "word",
      "position": 1
    },
    {
      "token": "hua",
      "start_offset": 2,
      "end_offset": 3,
      "type": "word",
      "position": 2
    },
    {
      "token": "ldh",
      "start_offset": 0,
      "end_offset": 3,
      "type": "word",
      "position": 3
    }
  ]
}
```

#### keep_original 功能 ####
`keep_original=true` 将保留原字符串，比如存入索引的数据为 `刘德华` 那么 `刘德华` 将也会被保存到索引中。`keep_original=false` 则不保存原字符串到索引

### trim_whitespace ###
#### trim_whitespace=true ####
```json
curl -X POST -d '{
  "analyzer" : "pinyin_analyzer",
  "text" : ["   最爱   刘德华   的帅气帅气的   "]
}' "http://localhost:9200/medcl20/_analyze"

{
  "tokens": [
    {
      "token": "zui",
      "start_offset": 3,
      "end_offset": 4,
      "type": "word",
      "position": 0
    },
    {
      "token": "ai",
      "start_offset": 4,
      "end_offset": 5,
      "type": "word",
      "position": 1
    },
    {
      "token": "liu",
      "start_offset": 8,
      "end_offset": 9,
      "type": "word",
      "position": 2
    },
    {
      "token": "de",
      "start_offset": 9,
      "end_offset": 10,
      "type": "word",
      "position": 3
    },
    {
      "token": "hua",
      "start_offset": 10,
      "end_offset": 11,
      "type": "word",
      "position": 4
    },
    {
      "token": "de",
      "start_offset": 14,
      "end_offset": 15,
      "type": "word",
      "position": 5
    },
    {
      "token": "shuai",
      "start_offset": 15,
      "end_offset": 16,
      "type": "word",
      "position": 6
    },
    {
      "token": "qi",
      "start_offset": 16,
      "end_offset": 17,
      "type": "word",
      "position": 7
    },
    {
      "token": "shuai",
      "start_offset": 17,
      "end_offset": 18,
      "type": "word",
      "position": 8
    },
    {
      "token": "qi",
      "start_offset": 18,
      "end_offset": 19,
      "type": "word",
      "position": 9
    },
    {
      "token": "de",
      "start_offset": 19,
      "end_offset": 20,
      "type": "word",
      "position": 10
    },
    {
      "token": "最爱   刘德华   的帅气帅气的",
      "start_offset": 0,
      "end_offset": 23,
      "type": "word",
      "position": 11
    },
    {
      "token": "zaldhdsqsqd",
      "start_offset": 0,
      "end_offset": 11,
      "type": "word",
      "position": 12
    }
  ]
}
```

#### trim_whitespace=false ####
```json
curl -X POST -d '{
  "analyzer" : "pinyin_analyzer",
  "text" : ["   最爱   刘德华   的帅气帅气的   "]
}' "http://localhost:9200/medcl20/_analyze"

{
  "tokens": [
    {
      "token": "zui",
      "start_offset": 3,
      "end_offset": 4,
      "type": "word",
      "position": 0
    },
    {
      "token": "ai",
      "start_offset": 4,
      "end_offset": 5,
      "type": "word",
      "position": 1
    },
    {
      "token": "liu",
      "start_offset": 8,
      "end_offset": 9,
      "type": "word",
      "position": 2
    },
    {
      "token": "de",
      "start_offset": 9,
      "end_offset": 10,
      "type": "word",
      "position": 3
    },
    {
      "token": "hua",
      "start_offset": 10,
      "end_offset": 11,
      "type": "word",
      "position": 4
    },
    {
      "token": "de",
      "start_offset": 14,
      "end_offset": 15,
      "type": "word",
      "position": 5
    },
    {
      "token": "shuai",
      "start_offset": 15,
      "end_offset": 16,
      "type": "word",
      "position": 6
    },
    {
      "token": "qi",
      "start_offset": 16,
      "end_offset": 17,
      "type": "word",
      "position": 7
    },
    {
      "token": "shuai",
      "start_offset": 17,
      "end_offset": 18,
      "type": "word",
      "position": 8
    },
    {
      "token": "qi",
      "start_offset": 18,
      "end_offset": 19,
      "type": "word",
      "position": 9
    },
    {
      "token": "de",
      "start_offset": 19,
      "end_offset": 20,
      "type": "word",
      "position": 10
    },
    {
      "token": "   最爱   刘德华   的帅气帅气的   ",
      "start_offset": 0,
      "end_offset": 23,
      "type": "word",
      "position": 11
    },
    {
      "token": "zaldhdsqsqd",
      "start_offset": 0,
      "end_offset": 11,
      "type": "word",
      "position": 12
    }
  ]
}
```

#### trim_whitespace 功能 ####
**去除字符串首尾空格字符，不去除字符串中间的空格。这个参数只有当 `keep_original=true` 时才能够看到效果。** 例如当字符串为：`   最爱   刘德华   的帅气帅气的   `，`trim_whitespace=true` 则原字符串将被保存为 `最爱   刘德华   的帅气帅气的`，如果 `trim_whitespace=false` 则原字符串将被保存为 `   最爱   刘德华   的帅气帅气的   `。如果 `keep_original=false`，那么原字符串没有被保存，也将看不到效果。

### keep_joined_full_pinyin ###
#### keep_joined_full_pinyin = false ####
```json
curl -X POST -d '{
  "analyzer" : "pinyin_analyzer",
  "text" : ["刘德华"]
}' "http://localhost:9200/medcl21/_analyze"

{
  "tokens": [
    {
      "token": "liu",
      "start_offset": 0,
      "end_offset": 1,
      "type": "word",
      "position": 0
    },
    {
      "token": "de",
      "start_offset": 1,
      "end_offset": 2,
      "type": "word",
      "position": 1
    },
    {
      "token": "hua",
      "start_offset": 2,
      "end_offset": 3,
      "type": "word",
      "position": 2
    },
    {
      "token": "刘德华",
      "start_offset": 0,
      "end_offset": 3,
      "type": "word",
      "position": 3
    },
    {
      "token": "ldh",
      "start_offset": 0,
      "end_offset": 3,
      "type": "word",
      "position": 4
    }
  ]
}
```

#### keep_joined_full_pinyin = true ####
```json
curl -X POST -d '{
  "analyzer" : "pinyin_analyzer",
  "text" : ["刘德华"]
}' "http://localhost:9200/medcl22/_analyze"

{
  "tokens": [
    {
      "token": "liu",
      "start_offset": 0,
      "end_offset": 1,
      "type": "word",
      "position": 0
    },
    {
      "token": "de",
      "start_offset": 1,
      "end_offset": 2,
      "type": "word",
      "position": 1
    },
    {
      "token": "hua",
      "start_offset": 2,
      "end_offset": 3,
      "type": "word",
      "position": 2
    },
    {
      "token": "刘德华",
      "start_offset": 0,
      "end_offset": 3,
      "type": "word",
      "position": 3
    },
    {
      "token": "liudehua",
      "start_offset": 0,
      "end_offset": 8,
      "type": "word",
      "position": 4
    },
    {
      "token": "ldh",
      "start_offset": 0,
      "end_offset": 3,
      "type": "word",
      "position": 5
    }
  ]
}
```

#### keep_joined_full_pinyin 功能 ####
**`keep_joined_full_pinyin=true` 将保存字符串拼音全拼，false 则不保存**。例如，当 `kepp_joined_full_pinyin=true` 时，文本 `刘德华` 的拼音全拼 `liudehua` 将会被保留；当 `keep_joined_full_pinyin=false` 则 全拼`liudehua`

### remove_duplicated_term ###
#### remove_duplicated_term = false ####
```json
curl -X POST -d '{
  "analyzer" : "pinyin_analyzer",
  "text" : ["刘德华刘德华帅帅帅，帅帅帅"]
}' "http://localhost:9200/medcl20/_analyze"

{
  "tokens": [
    {
      "token": "liu",
      "start_offset": 0,
      "end_offset": 1,
      "type": "word",
      "position": 0
    },
    {
      "token": "de",
      "start_offset": 1,
      "end_offset": 2,
      "type": "word",
      "position": 1
    },
    {
      "token": "hua",
      "start_offset": 2,
      "end_offset": 3,
      "type": "word",
      "position": 2
    },
    {
      "token": "liu",
      "start_offset": 3,
      "end_offset": 4,
      "type": "word",
      "position": 3
    },
    {
      "token": "de",
      "start_offset": 4,
      "end_offset": 5,
      "type": "word",
      "position": 4
    },
    {
      "token": "hua",
      "start_offset": 5,
      "end_offset": 6,
      "type": "word",
      "position": 5
    },
    {
      "token": "shuai",
      "start_offset": 6,
      "end_offset": 7,
      "type": "word",
      "position": 6
    },
    {
      "token": "shuai",
      "start_offset": 7,
      "end_offset": 8,
      "type": "word",
      "position": 7
    },
    {
      "token": "shuai",
      "start_offset": 8,
      "end_offset": 9,
      "type": "word",
      "position": 8
    },
    {
      "token": "shuai",
      "start_offset": 10,
      "end_offset": 11,
      "type": "word",
      "position": 9
    },
    {
      "token": "shuai",
      "start_offset": 11,
      "end_offset": 12,
      "type": "word",
      "position": 10
    },
    {
      "token": "shuai",
      "start_offset": 12,
      "end_offset": 13,
      "type": "word",
      "position": 11
    },
    {
      "token": "刘德华刘德华帅帅帅，帅帅帅",
      "start_offset": 0,
      "end_offset": 13,
      "type": "word",
      "position": 12
    },
    {
      "token": "ldhldhssssss",
      "start_offset": 0,
      "end_offset": 12,
      "type": "word",
      "position": 13
    }
  ]
}
```

#### remove_duplicated_term = true ####
```JSON
curl -X POST -d '{
  "analyzer" : "pinyin_analyzer",
  "text" : ["刘德华刘德华帅帅帅，帅帅帅"]
}' "http://localhost:9200/medcl26/_analyze"

{
  "tokens": [
    {
      "token": "liu",
      "start_offset": 0,
      "end_offset": 1,
      "type": "word",
      "position": 0
    },
    {
      "token": "de",
      "start_offset": 1,
      "end_offset": 2,
      "type": "word",
      "position": 1
    },
    {
      "token": "hua",
      "start_offset": 2,
      "end_offset": 3,
      "type": "word",
      "position": 2
    },
    {
      "token": "shuai",
      "start_offset": 6,
      "end_offset": 7,
      "type": "word",
      "position": 3
    },
    {
      "token": "刘德华刘德华帅帅帅，帅帅帅",
      "start_offset": 0,
      "end_offset": 13,
      "type": "word",
      "position": 4
    },
    {
      "token": "ldhldhssssss",
      "start_offset": 0,
      "end_offset": 12,
      "type": "word",
      "position": 5
    }
  ]
}
```
#### remove_duplicated_term 功能 ####
`remove_duplicated_term=true` 则会将文本中相同的拼音只保存一份，比如 `刘德华刘德华` 只会保留一份拼音 `liu`，`de`，`hua`；相对的 `remove_duplicated_term=false` 则会保留两份 `liu`，`de`，`hua`。**注意：`remove_duplicated_term` 并不会影响文本首字母的文本，`刘德华刘德华` 生成的首字母拼音始终都为 `ldhldh`**

#### remove_duplicated_term = true 并且 keep_joined_full_pinyin = true ####
```JSON
curl -X POST -d '{
  "analyzer" : "pinyin_analyzer",
  "text" : ["刘德华刘德华帅帅帅，帅帅帅"]
}' "http://localhost:9200/medcl27/_analyze"

{
  "tokens": [
    {
      "token": "liu",
      "start_offset": 0,
      "end_offset": 1,
      "type": "word",
      "position": 0
    },
    {
      "token": "de",
      "start_offset": 1,
      "end_offset": 2,
      "type": "word",
      "position": 1
    },
    {
      "token": "hua",
      "start_offset": 2,
      "end_offset": 3,
      "type": "word",
      "position": 2
    },
    {
      "token": "shuai",
      "start_offset": 6,
      "end_offset": 7,
      "type": "word",
      "position": 3
    },
    {
      "token": "刘德华刘德华帅帅帅，帅帅帅",
      "start_offset": 0,
      "end_offset": 13,
      "type": "word",
      "position": 4
    },
    {
      "token": "liudehualiudehuashuaishuaishuaishuaishuaishuai",
      "start_offset": 0,
      "end_offset": 46,
      "type": "word",
      "position": 5
    },
    {
      "token": "ldhldhssssss",
      "start_offset": 0,
      "end_offset": 12,
      "type": "word",
      "position": 6
    }
  ]
}
```

#### remove_duplicated_term 功能 ####
`remove_duplicated_term = true` 会过滤相同的拼音，但是不影响全拼，`刘德华刘德华` 生成的字符串全拼为 `liudehualiudehua`

### keep_none_chinese ###
#### keep_none_chinese = true ####
```java
POST /medcl20/_analyze HTTP/1.1
Host: localhost:9200

{
  "analyzer" : "pinyin_analyzer",
  "text" : ["刘*20*德b华DJ"]
}

{
  "tokens": [
    {
      "token": "liu",
      "start_offset": 0,
      "end_offset": 1,
      "type": "word",
      "position": 0
    },
    {
      "token": "20",
      "start_offset": 3,
      "end_offset": 5,
      "type": "word",
      "position": 1
    },
    {
      "token": "de",
      "start_offset": 5,
      "end_offset": 6,
      "type": "word",
      "position": 2
    },
    {
      "token": "b",
      "start_offset": 6,
      "end_offset": 7,
      "type": "word",
      "position": 3
    },
    {
      "token": "hua",
      "start_offset": 7,
      "end_offset": 8,
      "type": "word",
      "position": 4
    },
    {
      "token": "d",
      "start_offset": 7,
      "end_offset": 9,
      "type": "word",
      "position": 5
    },
    {
      "token": "j",
      "start_offset": 7,
      "end_offset": 9,
      "type": "word",
      "position": 6
    },
    {
      "token": "刘*20*德b华dj",
      "start_offset": 0,
      "end_offset": 10,
      "type": "word",
      "position": 7
    },
    {
      "token": "l20dbhdj",
      "start_offset": 0,
      "end_offset": 8,
      "type": "word",
      "position": 8
    }
  ]
}
```
#### keep_none_chinese = false ####
```java
POST /medcl28/_analyze HTTP/1.1
Host: localhost:9200

{
  "analyzer" : "pinyin_analyzer",
  "text" : ["刘*20*德b华DJ"]
}

{
  "tokens": [
    {
      "token": "liu",
      "start_offset": 0,
      "end_offset": 1,
      "type": "word",
      "position": 0
    },
    {
      "token": "de",
      "start_offset": 5,
      "end_offset": 6,
      "type": "word",
      "position": 1
    },
    {
      "token": "hua",
      "start_offset": 7,
      "end_offset": 8,
      "type": "word",
      "position": 2
    },
    {
      "token": "刘*20*德b华dj",
      "start_offset": 0,
      "end_offset": 10,
      "type": "word",
      "position": 3
    },
    {
      "token": "l20dbhdj",
      "start_offset": 0,
      "end_offset": 8,
      "type": "word",
      "position": 4
    }
  ]
}
```

### keep_none_chinese 功能 ##
`keep_none_chinese = true` 则非中文字母以及数字将会被保留，但是要确定所有的特别字符都是无法被保留下来的。例如，文本 `刘*20*德b华dj` 中的数字 `20`，字母 `b` 与 `dj` 将会被保留，而特殊字符 `*` 是不会保留的；当 `keep_none_chinese=false` 则非中文字母以及数字将不会被保留，上述文本中的数字 `20`，字母 `b` 与 `dj` 将不会被保留。**注意：参数 `keep_none_chinese` 是不会影响首字母以及所有字符组成全拼的拼音，上述文本生成的首字母拼音为 `l20dbhdj`，所有字符组成的全拼为：`liu20debhuadj`，特别字符始终是被过滤去除的。**
