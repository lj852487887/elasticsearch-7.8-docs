##29.2.3 Incorporating static relevance signals into the score 将静态相关信号纳入分数

许多领域有静态信号，已知与相关性相关。例如，PageRank和url length是网页搜索的两个常用特性，用于独立于查询调整网页页面的得分。

有两个主要的查询允许将静态分数贡献与文本相关性相结合，例如用BM25计算：-[`script_score` query]() -[`rank_feature` query]()

例如，假设您有一个`pagerank`字段，希望将其与BM25分数相结合，以便最终分数等于 `score = bm25_score + pagerank / (10 + pagerank)`

使用[`script_score` query]()的请求将会像这样：

```json
GET index/_search
{
  "query": {
    "script_score": {
      "query": {
        "match": { "body": "elasticsearch" }
      },
      "script": {
        "source": "_score * saturation(doc['pagerank'].value, 10)" ①
      }
    }
  }
}
```

① `pagerank` 必须被映射为 [Numeric]()

同时，使用[`rank_feature` query]()的请求将会像这样：

```json
GET _search
{
  "query": {
    "bool": {
      "must": {
        "match": { "body": "elasticsearch" }
      },
      "should": {
        "rank_feature": {
          "field": "pagerank", ①
          "saturation": {
            "pivot": 10
          }
        }
      }
    }
  }
}
```
① `pagerank` 必须被映射为 [rank_feature]() 字段

虽然这两个选项都会返回相似的分数，但有一个权衡：[script_score]()提供了很大的灵活性，使您能够根据自己的喜好将文本相关性分数与静态信号相结合。另一方面，[`rank_feature` 查询]()只公开了几种将静态符号合并到分数中的方法。但是，它依赖于[`rank_feature`]()和[`rank_features`]()字段，这些字段以一种特殊的方式索引值，从而允许[`rank_feature` 查询]() 跳过非竞争性文档并更快地获得查询的头部匹配项。




















