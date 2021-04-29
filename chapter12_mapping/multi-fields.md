# `fields`

为不同的目的以不同的方式索引同一字段通常很有用。这就是多字段的目的。例如，`string`字段可以映射为全文搜索的`text`字段，也可以映射为排序或聚合的`keyword`字段：

```json
PUT my-index-000001
{
  "mappings": {
    "properties": {
      "city": {
        "type": "text",
        "fields": {
          "raw": {  ①
            "type":  "keyword" 
          }
        }
      }
    }
  }
}

PUT my-index-000001/_doc/1
{
  "city": "New York"
}

PUT my-index-000001/_doc/2
{
  "city": "York"
}

GET my-index-000001/_search
{
  "query": {
    "match": {
      "city": "york" ②
    }
  },
  "sort": {
    "city.raw": "asc" ③
  },
  "aggs": {
    "Cities": {
      "terms": {
        "field": "city.raw" ③
      }
    }
  }
}
```

① `city.raw`字段是`city`字段的`keyword`版本。

② `city`字段可用于全文本搜索。

③ `city.raw`字段可用于排序和聚合







