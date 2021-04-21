#13.2.1 Boolean Query 布尔查询

一种查询，它匹配与其他查询的布尔组合相匹配的文档。bool查询对应到Lucene中的`BooleanQuery`。它是使用一个或多个布尔子句构建的，每个子句都有一个类型化的引用。相关类型为：

标记 	| 	描述 	
---		|	---			
`must `| 	子句（query）必须出现在匹配的文档中，并且将对得分起作用		
`filter `| 子句（query）必须出现在匹配的文档中。但是不同于`must` ，查询的分数将被忽略。Filter子句是在[Filter上下文]()中执行的，这意味着评分被忽略，子句被考虑用于缓存。			
`should `	| 	子句（query）应该出现在匹配的文档中。		
`must_not `	| 子句（query）不能出现在匹配的文档中。子句是在[Filter上下文]()中执行的，这意味着评分被忽略，子句被考虑用于缓存。因为评分被忽略，所以返回所有文档为 `0` 分。				

`bool`查询采用了一种“*匹配越多越好*”的方法，因此每个匹配`must`或`should`子句的分数将被添加到一起，以提供每个文档的最终`_score`。

```json
POST /_search
{
  "query": {
    "bool" : {
      "must" : {
        "term" : { "user.id" : "kimchy" }
      },
      "filter": {
        "term" : { "tags" : "production" }
      },
      "must_not" : {
        "range" : {
          "age" : { "gte" : 10, "lte" : 20 }
        }
      },
      "should" : [
        { "term" : { "tags" : "env1" } },
        { "term" : { "tags" : "deployed" } }
      ],
      "minimum_should_match" : 1,
      "boost" : 1.0
    }
  }
}
```

###使用最小匹配 `minimum_should_match`

您可以使用`minimum_should_match`参数指定返回的文档*必须*匹配的`should`子句的数目或百分比。


如果`bool`查询至少包含一个`should`子句，并且没有`must`或`filter`子句，则默认值为`1`。否则，默认值为`0`。



有关其他有效值，请参阅[`minimum_should_match`参数]()。

###使用`bool.filter`算分

在`filter`元素下指定的查询对评分没有影响 - 分数返回为`0`。分数只受指定的查询影响。例如，以下三个查询都返回`status`字段包含术语`active`的所有文档。

第一个查询为所有文档分配`0`分，因为没有指定评分查询：

```json
GET _search
{
  "query": {
    "bool": {
      "filter": {
        "term": {
          "status": "active"
        }
      }
    }
  }
}
```
这个`bool`查询有一个`match_all`查询，它为所有文档分配`1.0`的分数。


```json
GET _search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "term": {
          "status": "active"
        }
      }
    }
  }
}
```
这个`constant_score`查询的行为方式与上面的第二个示例完全相同。`constant_score`查询为所有与筛选器匹配的文档分配`1.0`的分数。

```json
GET _search
{
  "query": {
    "constant_score": {
      "filter": {
        "term": {
          "status": "active"
        }
      }
    }
  }
}
```

###命名查询

每个查询在其顶级定义中接受一个`_name`。您可以使用命名查询来跟踪哪些查询与返回的文档匹配。如果使用命名查询，则响应将为每个命中包含`matched_queries`属性。


```json
GET /_search
{
  "query": {
    "bool": {
      "should": [
        { "match": { "name.first": { "query": "shay", "_name": "first" } } },
        { "match": { "name.last": { "query": "banon", "_name": "last" } } }
      ],
      "filter": {
        "terms": {
          "name.last": [ "banon", "kimchy" ],
          "_name": "test"
        }
      }
    }
  }
}
```



