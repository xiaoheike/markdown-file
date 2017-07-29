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
查询语法中指定的 `field:search_text_new` 使用 `ik` 分词。

`suggest_mode` 参考文章：[http://elasticsearch.cn/article/142](http://elasticsearch.cn/article/142)，理解如下：
`term suggester` 有一个可配置参数为 `suggest_mode`，可选值为：`popular`，`missing`，`always`。`missing` 是默认配置，如果搜索的关键词在文档中已经存在，那么将不会有推荐选项，比如 `北京` 一词已经在文档中存在，那么使用 `term suggest` 语法搜索 `北京` 将不会有推荐选项；`popular` 是指会推荐相似度更高的匹配词，比如文档中存在 `北京`，`北京人`，使用 `term suggest` 搜索 `北京`，如果 `北京人` 的分数高于 `北京` 则 `北京人` 会被推荐；`always` 是指只要是相似则会在推荐选项中给出，比如 `北京`，`北京人`，`北京城`，使用 `term suggest` 语法搜索 `北京` 会将 `北京人` 和 `北京城` 都做为推荐选项。

## [term suggester 参数](https://www.elastic.co/guide/en/elasticsearch/reference/2.3/search-suggesters-term.html) ##
`term suggester` 用到的一些参数及说明。

## [phrase suggester](https://www.elastic.co/guide/en/elasticsearch/reference/2.3/search-suggesters-phrase.html) ##
`phrase Suggester` 也是提供关键词自动纠错功能，是 `term suggester` 的升级版。

## [completion suggester](https://www.elastic.co/guide/en/elasticsearch/reference/2.3/search-suggesters-completion.html) ##
`Completion Suggester` 前缀匹配，不具有像 `term` 以及 `phrase` 关键词的自动纠错功能，是一种自动补全功能。

## [completion suggester 中文使用示例](http://www.tcao.net/article/86.html) ##
