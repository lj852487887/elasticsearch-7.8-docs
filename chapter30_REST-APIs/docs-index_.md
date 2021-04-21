#Index API

> 请参见[移除映射类型]()。

将JSON文档添加到指定的数据流或索引并使其可搜索。如果目标是索引并且文档已经存在，则请求会更新文档并增加其版本。

> 不能使用Index API将现有文档的更新请求发送到数据流。请参见[通过查询更新数据流中的文档]()以及[更新或删除备份索引中的文档]()。


##请求

`PUT /<target>/_doc/<_id>`

`POST /<target>/_doc/`

`PUT /<target>/_create/<_id>`

`POST /<target>/_create/<_id>`

> 不能使用`PUT /<target>/_doc/<_id>`请求格式向数据流添加新文档。要指定文档ID，请改用`PUT /<target>/_create/<_id>`格式。请参见[将文档添加到数据流]()。


##先决条件

* 如果启用了Elasticsearch安全功能，则必须对目标数据流、索引或索引别名具有以下[索引权限]()。

	* 要使用`PUT/<target>/_doc/<_id>`请求格式添加或覆盖文档，必须具有`create, index`或`write`索引权限。
	* 要使用`POST /<target>/_doc/, PUT /<target>/_create/<_id>`或`POST /<target>/_create/<_id>`请求格式添加文档，您必须具有`create_doc、create、index`或`write` 索引权限。
	* 若要使用Index API请求自动创建数据流或索引，必须具有`auto_configure, create_index`或`manage`索引权限。


* 自动创建数据流需要启用数据流的匹配索引模板。请参见[设置数据流]()。


##路径参数

###`<target>`

（必需，字符串）目标数据流或索引的名称。


如果目标不存在，并且匹配到[具有`data_stream`定义的索引模板]()的名称或通配符（`*`）模式，则此请求将创建数据流。请参见[设置数据流]()。


如果目标不存在并且与数据流模板不匹配，则此请求将创建索引。


您可以使用[resolve index]() API检查现有目标。

###`<_id>`

（可选，字符串）文档的唯一标识符。

以下请求格式需要此参数：

* `PUT /<target>/_doc/<_id>`
* `PUT /<target>/_create/<_id>`
* `POST /<target>/_create/<_id>`

要自动生成文档ID，请使用`POST /<target>/_doc/` 请求格式并忽略此参数。

##请求参数

###`if_seq_no`

（可选，整数）仅当文档具有此序列号时才执行该操作。请参阅[乐观并发控制]()。

###`if_primary_term`

（可选，整数）仅当文档具有此主项时才执行该操作。请参阅[乐观并发控制]()。

###`op_type`

（可选，枚举）设置为`create`，以便仅在文档不存在时对其进行索引（如果不存在则放置）。如果指定`_id`的文档已经存在，索引操作将失败。与使用`<index>/_create`端点相同。有效值：`index`, `create`。如果指定了文档id，则默认为`index`。否则，它默认为`create`。



> 如果请求以数据流为目标，则需要 `op_type` 的`create`。请参见[将文档添加到数据流]()。


###`pipeline`

（可选，字符串）用于预处理传入文档的管道的ID。

###`refresh`

（可选，枚举）如果为`true`，Elasticsearch刷新受影响的分片以使此操作对搜索可见，如果为`wait_for`，则需等待刷新以使此操作对搜索可见，如果为`false`，则不对刷新执行任何操作。有效值：`true, false, wait_for`。默认值：`false`。

###`routing`

（可选，字符串）指定主分片为目标。

###`timeout`

（可选，[时间单位]()）等待以下操作的请求时间：

* [自动创建索引]()
* [动态映射]()更新
* [等待激活的分片]()

默认为`1m`（一分钟）。这保证了Elasticsearch在失败之前至少等待过超时。实际等待时间可能更长，特别是在发生多个等待时。



###`version`

（可选，整数）并发控制的显式版本号。指定的版本必须与文档的当前版本匹配，请求才能成功。

###`version_type`

（可选，枚举）特定的版本类型：`internal, external, external_gte`。

###`wait_for_active_shards`

（可选，字符串）在继续操作之前必须处于激活状态的分片副本数。设置为`all`或最大值为索引中碎片总数（`number_of_replicas+1`）的任意正整数。默认值：`1`，即主分片。



请参阅[激活的分片]()。


###`require_alias`

（可选，布尔值）如果为`true`，则目标必须是[索引别名]()。默认为`false`。


##范例

在`my-index-000001`索引中插入一个JSON文档，索引`_id`为1：

```json
PUT my-index-000001/_doc/1
{
  "@timestamp": "2099-11-15T13:12:00",
  "message": "GET /search HTTP/1.1 200 1070000",
  "user": {
    "id": "kimchy"
  }
}
```

API 返回如下结果：

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
  "_version": 1,
  "_seq_no": 0,
  "_primary_term": 1,
  "result": "created"
}
```


如果不存在具有该ID的文档，请使用`_create`资源将文档索引到`my-index-000001`索引中：


```json
PUT my-index-000001/_create/1
{
  "@timestamp": "2099-11-15T13:12:00",
  "message": "GET /search HTTP/1.1 200 1070000",
  "user": {
    "id": "kimchy"
  }
}
```

如果不存在具有该ID的文档，请将`op_type`参数设置为`create`，以便将文档索引到`my-index-000001`索引中：


```json
PUT my-index-000001/_doc/1?op_type=create
{
  "@timestamp": "2099-11-15T13:12:00",
  "message": "GET /search HTTP/1.1 200 1070000",
  "user": {
    "id": "kimchy"
  }
}
```