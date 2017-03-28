内容来自：[http://elasticsearch.cn/book/elasticsearch_definitive_guide_2.x/combining-queries-together.html](http://elasticsearch.cn/book/elasticsearch_definitive_guide_2.x/combining-queries-together.html)
## 组合多查询 ##
现实的查询需求从来都没有那么简单；它们需要在多个字段上查询多种多样的文本，并且根据一系列的标准来过滤。为了构建类似的高级查询，你需要一种能够将多查询组合成单一查询的查询方法。
你可以用 `bool` 查询来实现你的需求。这种查询将多查询组合在一起，成为用户自己想要的布尔查询。它接收以下参数：
**must**
文档 ***必须*** 匹配这些条件才能被包含进来。

**must_not**
文档 ***必须不*** 匹配这些条件才能被包含进来。

**should**
如果满足这些语句中的任意语句，将增加 `_score`，否则，无任何影响。它们主要用于修正每个文档的相关性得分。

**filter**
***必须*** 匹配，但它 ***以不评分、过滤模式*** 来进行。这些语句对评分没有贡献，只是根据过滤标准来排除或包含文档。

由于这是我们看到的第一个包含多个查询的查询，所以有必要讨论一下相关性得分是如何组合的。每一个子查询都独自地计算文档的相关性得分。一旦他们的得分被计算出来， bool 查询就将这些得分进行合并并且返回一个代表整个布尔操作的得分。
下面的查询用于查找 `title` 字段匹配 `how to make millions` 并且不被标识为 `spam` 的文档。那些被标识为 starred 或在2014之后的文档，将比另外那些文档拥有更高的排名。如果 ***两者*** 都满足，那么它排名将更高：
```json
{
    "bool": {
        "must":     { "match": { "title": "how to make millions" }},
        "must_not": { "match": { "tag":   "spam" }},
        "should": [
            { "match": { "tag": "starred" }},
            { "range": { "date": { "gte": "2014-01-01" }}}
        ]
    }
}
```

***在 `bool` 查询中，被包裹的 `should` 查询默认情况下不需要匹配任何 `should` 子句；但是如果没有 `must` 语句，那么至少需要能够匹配其中的一条 `should` 语句；相对的如果已经存在 `must` 语句，则 `soulde` 语句只会对打分造成影响。***
像我们控制 `match` 查询的精度一样，我们也可以通过 `minimum_should_match` 参数控制多少 `should` 子句需要被匹配，这个参数可以是正整数，也可以是百分比。例如以下的查询语法就要求：结果集仅包含 `title` 字段中有 `brown` 和 `fox`, `brown` 和 `dog`, 或 `fox` 和 `dog` 的文档。如果一个文档包含上述三个条件，那么它的相关性就会比其他仅包含三者中的两个条件的文档要高。
```json
{
  "query": {
    "bool": {
      "should": [
        { "match": { "title": "brown" }},
        { "match": { "title": "fox"   }},
        { "match": { "title": "dog"   }}
      ],
      "minimum_should_match": 2 <1>
    }
  }
}
```
<1> 这也可以用百分比表示

## 增加带过滤器（filtering）的查询 ##
如果我们不想因为文档的时间而影响得分，可以用 `filter` 语句来重写前面的例子：
```json
{
    "bool": {
        "must":     { "match": { "title": "how to make millions" }},
        "must_not": { "match": { "tag":   "spam" }},
        "should": [
            { "match": { "tag": "starred" }}
        ],
        "filter": {
          "range": { "date": { "gte": "2014-01-01" }}  
        }
    }
}
```
- `range` 查询已经从 `should` 语句中移到 `filter` 语句

通过将 `range` 查询移到 `filter` 语句中，我们将它转成不评分的查询，将不再影响文档的相关性排名。由于它现在是一个不评分的查询，可以使用各种对 `filter` 查询有效的优化手段来提升性能。
所有查询都可以借鉴这种方式。将查询移到 `bool` 查询的 `filter` 语句中，这样它就自动的转成一个不评分的 `filter` 了。
如果你需要通过多个不同的标准来过滤你的文档，`bool` 查询本身也可以被用做不评分的查询。简单地将它放置到 `filter` 语句中并在内部构建布尔逻辑：
```json
{
    "bool": {
        "must":     { "match": { "title": "how to make millions" }},
        "must_not": { "match": { "tag":   "spam" }},
        "should": [
            { "match": { "tag": "starred" }}
        ],
        "filter": {
          "bool": {  
              "must": [
                  { "range": { "date": { "gte": "2014-01-01" }}},
                  { "range": { "price": { "lte": 29.99 }}}
              ],
              "must_not": [
                  { "term": { "category": "ebooks" }}
              ]
          }
        }
    }
}
```
- 我们可以在过滤标准中增加布尔逻辑，例如上述示例，将 `bool` 查询包裹在 `filter` 语句中

通过混合布尔查询，我们可以在我们的查询请求中灵活地编写 `scoring` 和 `filtering` 查询逻辑。

## constant_score 查询 ##
尽管没有 `bool` 查询使用这么频繁，`constant_score` 查询也是你工具箱里有用的查询工具。它将一个不变的常量评分应用于所有匹配的文档。它被经常用于你只需要执行一个 `filter` 而没有其它查询（例如，评分查询）的情况下。
可以使用它来取代只有 `filter` 语句的 `bool` 查询。在性能上是完全相同的，但对于提高查询简洁性和清晰度有很大帮助。
```json
{
    "constant_score":   {
        "filter": {
            "term": { "category": "ebooks" }  
        }
    }
}
```
- `term` 查询被放置在 `constant_score` 中，转成不评分的 `filter`。这种方式可以用来取代只有 `filter` 语句的 `bool` 查询。
