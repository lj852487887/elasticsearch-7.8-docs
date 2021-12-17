##29.2.1 Mixing exact search with stemming 混合精确搜索和词干分析

在构建搜索应用程序时，词干分析通常是必须的，因为查询`skiing`时需要匹配包含`ski`或`skis`的文档。但是如果用户想专门搜索`skiing`呢？执行此操作的典型方法是使用[多字段]()，以便以两种不同的方式索引相同的内容：

```json
PUT index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "english_exact": {
          "tokenizer": "standard",
          "filter": [
            "lowercase"
          ]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "body": {
        "type": "text",
        "analyzer": "english",
        "fields": {
          "exact": {
            "type": "text",
            "analyzer": "english_exact"
          }
        }
      }
    }
  }
}

PUT index/_doc/1
{
  "body": "Ski resort"
}

PUT index/_doc/2
{
  "body": "A pair of skis"
}

POST index/_refresh
```
使用这样的设置，在`body`上搜索`ski`将会返回相同的文档：

```json
GET index/_search
{
  "query": {
    "simple_query_string": {
      "fields": [ "body" ],
      "query": "ski"
    }
  }
}
```

```json
{
  "took": 2,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped" : 0,
    "failed": 0
  },
  "hits": {
    "total" : {
        "value": 2,
        "relation": "eq"
    },
    "max_score": 0.18232156,
    "hits": [
      {
        "_index": "index",
        "_type": "_doc",
        "_id": "1",
        "_score": 0.18232156,
        "_source": {
          "body": "Ski resort"
        }
      },
      {
        "_index": "index",
        "_type": "_doc",
        "_id": "2",
        "_score": 0.18232156,
        "_source": {
          "body": "A pair of skis"
        }
      }
    ]
  }
}
```

另一方面，在`body.exact`上搜索`ski`,将只返回文档`1`，因为`body.exact`不执行词干分析。

```json
GET index/_search
{
  "query": {
    "simple_query_string": {
      "fields": [ "body.exact" ],
      "query": "ski"
    }
  }
}
```

```json
{
  "took": 1,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped" : 0,
    "failed": 0
  },
  "hits": {
    "total" : {
        "value": 1,
        "relation": "eq"
    },
    "max_score": 0.8025915,
    "hits": [
      {
        "_index": "index",
        "_type": "_doc",
        "_id": "1",
        "_score": 0.8025915,
        "_source": {
          "body": "Ski resort"
        }
      }
    ]
  }
}
```

这并不是一件容易向最终用户公开的事情，因为我们需要一种方法来确定他们是否在寻找一个精确的匹配，并相应地重定向到相应的字段。另外，如果只有部分查询需要精确匹配，而其他部分仍然需要考虑词干，该怎么办？


幸运的是，`query_string`和`simple_query_string`查询有一个功能可以解决这个问题：`quote_field_suffix`。这会告诉Elasticsearch，出现在引号之间的单词将被重定向到另一个字段，请参见以下内容：

```json
GET index/_search
{
  "query": {
    "simple_query_string": {
      "fields": [ "body" ],
      "quote_field_suffix": ".exact",
      "query": "\"ski\""
    }
  }
}
```

```json
{
  "took": 2,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped" : 0,
    "failed": 0
  },
  "hits": {
    "total" : {
        "value": 1,
        "relation": "eq"
    },
    "max_score": 0.8025915,
    "hits": [
      {
        "_index": "index",
        "_type": "_doc",
        "_id": "1",
        "_score": 0.8025915,
        "_source": {
          "body": "Ski resort"
        }
      }
    ]
  }
}
```

在上面的例子中，由于`quote_field_suffix`参数，所以在`body.exact`字段，因此只有文档`1`匹配。这允许用户根据自己的喜好将精确搜索与词干搜索混合使用。

* 如果在`quote_field_suffix`中传递的字段选项不存在，搜索将退回到使用查询字符串的默认字段。





