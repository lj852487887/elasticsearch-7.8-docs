#对搜索结果排序

允许您在特定字段上添加一个或多个排序。每个排序也可以被反向。排序是在字段级别上定义的，特殊的字段名，比如`_score`按分数排序，`_doc`按索引顺序排序。

假设以下索引映射：

```json
PUT /my-index-000001
{
  "mappings": {
    "properties": {
      "post_date": { "type": "date" },
      "user": {
        "type": "keyword"
      },
      "name": {
        "type": "keyword"
      },
      "age": { "type": "integer" }
    }
  }
}
```

```json
GET /my-index-000001/_search
{
  "sort" : [
    { "post_date" : {"order" : "asc"}},
    "user",
    { "name" : "desc" },
    { "age" : "desc" },
    "_score"
  ],
  "query" : {
    "term" : { "user" : "kimchy" }
  }
}
```

> `_doc`除了是最有效的排序顺序之外，没有真正的用例。因此，如果您不关心文档返回的顺序，那么您应该按文档`_doc`。这在[滚动]()时特别有用。

##排序值

返回的每个文档的排序值也作为响应的一部分返回。

##排序顺序

order 选项有以下取值：

`asc` 	| 	升序排序 	
---		|	---			
**`desc`** | 	**降序排序**

当对`_score`排序时，默认为 `desc` , 当对其他所有字段排序时，默认为 `asc`。

##排序模式选项

Elasticsearch支持按数组或多值字段排序。`mode`选项控制选择什么数组值来对它所属的文档进行排序。模式选项可以具有以下值：

```json
PUT /my-index-000001/_doc/1?refresh
{
   "product": "chocolate",
   "price": [20, 4]
}

POST /_search
{
   "query" : {
      "term" : { "product" : "chocolate" }
   },
   "sort" : [
      {"price" : {"order" : "asc", "mode" : "avg"}}
   ]
}
```




```json
PUT /index_double
{
  "mappings": {
    "properties": {
      "field": { "type": "double" }
    }
  }
}
```


```json
PUT /index_long
{
  "mappings": {
    "properties": {
      "field": { "type": "long" }
    }
  }
}
```
