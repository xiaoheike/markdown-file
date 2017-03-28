内容来自：[http://elasticsearch.cn/book/elasticsearch_definitive_guide_2.x/_most_important_queries.html](http://elasticsearch.cn/book/elasticsearch_definitive_guide_2.x/_most_important_queries.html)
## 最重要的查询 ##
虽然 `Elastidsearch` 自带了很多的查询，但经常用到的也就那么几个。我们将在 [深入搜索](http://elasticsearch.cn/book/elasticsearch_definitive_guide_2.x/search-in-depth.html) 章节详细讨论那些查询的细节，接下来我们对最重要的几个查询进行简单介绍。

## match_all 查询 ##
`match_all` 查询简单的 ***匹配*** 所有文档。在没有指定查询方式时，它是默认的查询：
```json
{ "match_all": {}}
```

它经常与 `filter` 结合使用，因为 `filter` 是不计算 `_score` 的--例如，检索收件箱里的所有邮件。所有邮件被认为具有相同的相关性，所以都将获得分值为 `1` 的 `_score`。

## match 查询 ##
无论你在任何字段上进行的是 ***全文搜索*** 还是 ***精确查询***，`match` 查询是你可用的标准查询。

如果你在一个全文字段上使用 `match` 查询，在执行查询前，它将用正确的 ***分析器*** 去分析查询字符串：
```json
{ "match": { "tweet": "QUICK" }}
```
`Elasticsearch` 通过下面的步骤执行 `match` 查询：
- 检查 `field` 类型
`tweet` 字段是一个字符串(`analyzed`)，所以该查询字符串也需要被分析(`analyzed`)。假设如果我们将 `tweet` 的 `analyzer` 指定为 `ik` 分析器，那么 `match` 查询将会调用 `ik` 分析器对该字符串做处理。

- 分析查询字符串
查询词 `QUICK!` 经过标准分析器的分析后变成单词 `quick`。因为我们只有一个查询词，因此 `match` 查询可以以一种低级别 `term` 查询的方式执行（`term` 是精确匹配，不会做分词等等的处理）。

- 找到匹配的文档
`term` 查询在倒排索引中搜索 `quick`，并且返回包含该词的文档。

- 为每个文档打分
`term` 查询综合考虑词频（每篇文档 `tweet` 字段包含 `quick` 的次数）、逆文档频率（在全部文档中 `title` 字段包含 `quick` 的次数）、包含 `quick` 的字段长度（长度越短越相关）来计算每篇文档的相关性得分 `_score`。

如果在一个精确值的字段上使用它， 例如数字、日期、布尔或者一个 `not_analyzed` 字符串字段，那么它将会精确匹配给定的值：
```json
{ "match": { "age":    26           }}
{ "match": { "date":   "2014-09-01" }}
{ "match": { "public": true         }}
{ "match": { "tag":    "full_text"  }}
```
所以说 `match` 查询非常强大，能够实现 ***全文检索（分析器处理）与精确检索（无分析器处理）***

**Tip**
对于精确值的查询，你可能需要使用 `filter` 语句来取代 `query`，因为 `filter` 将会被缓存。接下来，我们将看到一些关于 `filter` 的例子。

## multi_match 查询 ##
`multi_match` 查询可以在多个字段上执行相同的 `match` 查询：
```json
{
    "multi_match": {
        "query":    "full text search",
        "fields":   [ "title", "body" ]
    }
}
```

## range 查询 ##
`range` 查询找出那些落在指定区间内的数字或者时间：
```json
{
    "range": {
        "age": {
            "gte":  20,
            "lt":   30
        }
    }
}
```

`range` 查询允许的操作符如下：
- gt 大于
- gte 大于等于
- lt 小于
- lte 小于等于

## term 查询 ##
`term` 查询被用于 ***精确值匹配***，这些精确值可能是数字、时间、布尔或者那些 ***not_analyzed*** 的字符串：
```json
{ "term": { "age":    26           }}
{ "term": { "date":   "2014-09-01" }}
{ "term": { "public": true         }}
{ "term": { "tag":    "full_text"  }}
```

`term` 查询对于输入的文本 ***不分析***，所以它将给定的值进行精确查询。

## terms 查询 ##
`terms` 查询和 `term` 查询一样，但它允许你指定多值进行匹配。如果这个字段包含了指定值中的***任何一个值***，那么这个文档满足条件：
```json
{ "terms": { "tag": [ "search", "full_text", "nosql" ] }}
```
和 `term` 查询一样，`terms` 查询对于输入的文本 ***不分析***。它查询那些精确匹配的值（包括在大小写、重音、空格等方面的差异）。

## exists 查询和 missing 查询 ##
`exists` 查询和 `missing` 查询被用于查找那些指定字段中有值 (`exists`) 或无值 (`missing`) 的文档。这与 `SQL` 中的 `IS_NULL (missing)` 和 `NOT IS_NULL (exists)` 在本质上具有共性：
```json
{
    "exists":   {
        "field":    "title"
    }
}
```
这些查询经常用于某个字段有值的情况和某个字段缺值的情况。
