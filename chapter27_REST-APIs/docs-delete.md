#Delete API


从指定索引中删除JSON文档。


##请求

`DELETE /<index>/_doc/<_id>`

##说明

使用DELETE从索引中删除文档。必须指定索引名和文档ID。



###乐观并发控制

删除操作可以是有条件的，并且只有在上次对文档的修改被分配了`if_seq_no`和`if_primary_term`参数指定的序列号和主要项时才能执行。如果检测到不匹配，该操作将导致`VersionConflictException`和状态代码409。有关详细信息，请参阅[乐观并发控制]()。



###版本控制

索引的每个文档都有版本控制。删除文档时，可以指定`version`，以确保我们尝试删除的相关文档实际上已被删除，并且在此期间没有更改。对文档执行的每个写入操作（包括删除）都会导致其版本递增。已删除文档的版本号在删除后短时间内保持可用，以便控制并发操作。已删除文档的版本保持可用的时间长度由`index.gc_deletes`的索引设置控制，默认为60秒。



###路由

如果索引期间使用路由，则还需要指定路由值以删除文档。



如果将`_routing` 映射设置为`required`并且没有指定路由值，delete API将抛出`RoutingMissingException`并拒绝请求。



例如：

```json
DELETE /my-index-000001/_doc/1?routing=shard-1
```


此请求删除id为`1`的文档，但它是基于用户路由的。如果未指定正确的路由，则不会删除文档。



###自动创建索引

如果使用[外部版本控制变量]()，则删除操作会自动创建指定的索引（如果不存在）。有关手动创建索引的信息，请参见[创建索引API]()。



###分布式

delete操作被哈希到一个特定的分片id中，然后被重定向到该id组中的主分片，并复制（如果需要）到该id组中的分片副本。



###等待活动分片

在发出删除请求时，可以将`wait_for_active_shards`参数设置在开始处理删除请求之前要求的活动的最小碎片副本数。请参阅[此处]()了解更多详细信息和用法示例。



###刷新

控制此请求所做的更改何时对搜索可见。参阅 [?refresh]()。



###超时

执行删除操作时，分配用于执行删除操作的主分片可能不可用。其中的一些原因可能是主分片当前正在从存储中恢复或正在重新定位。默认情况下，删除操作将等待主碎片变为可用状态最多1分钟，否则将失败并返回错误。`timeout`参数可用于显式指定它等待的时间。下面是将其设置为5分钟的示例：



```json
DELETE /my-index-000001/_doc/1?timeout=5m
```

##路径参数

###<index>

(必需, 字符串) 目标索引的名字.

###<_id>

(必需, 字符) 文档的唯一标识符.


##查询参数

###`if_seq_no`

(可选, 整数) Only perform the operation if the document has this sequence number. See Optimistic concurrency control.

###`if_primary_term`

(可选, 整数) Only perform the operation if the document has this primary term. See Optimistic concurrency control.

###`refresh`

(可选, 枚举) If true, Elasticsearch refreshes the affected shards to make this operation visible to search, if wait_for then wait for a refresh to make this operation visible to search, if false do nothing with refreshes. Valid values: true, false, wait_for. Default: false.

###`routing`

(可选, 字符串) Target the specified primary shard.

###`master_timeout`

(可选, 时间单位) Specifies the period of time to wait for a connection to the master node. If no response is received before the timeout expires, the request fails and returns an error. Defaults to 30s.

###`timeout`

(可选, 时间单位) Specifies the period of time to wait for a response. If no response is received before the timeout expires, the request fails and returns an error. Defaults to 30s.

###`version`

(可选, 整数) Explicit version number for concurrency control. The specified version must match the current version of the document for the request to succeed.

###`version_type`

(可选, 枚举) Specific version type: internal, external, external_gte.

###`wait_for_active_shards`

(可选, 字符串) The number of shard copies that must be active before proceeding with the operation. Set to all or any positive integer up to the total number of shards in the index (number_of_replicas+1). Default: 1, the primary shard.

See Active shards.



##范例

从索引`my-index-000001`删除JSON文档`1`


```json
DELETE /my-index-000001/_doc/1
```

API 返回以下结果：


```json
{
  "_shards": {
    "total": 2,
    "failed": 0,
    "successful": 2
  },
  "_index": "my-index-000001",
  "_type": "_doc",
  "_id": "1",
  "_version": 2,
  "_primary_term": 1,
  "_seq_no": 5,
  "result": "deleted"
}
```