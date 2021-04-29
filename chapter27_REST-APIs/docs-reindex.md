#Reindex API

将文档从源复制到目标。

源和目标可以是任何预先存在的索引、索引别名或[数据流]()。但是，源和目标必须不同。例如，不能将数据流重新索引到自身中。

> 重新索引要求为源中的所有文档启用[_source]()。
> 
> 在调用`_reindex`之前，应根据需要配置目标。重新索引不会从源或其关联模板复制设置。
> 
> 必须提前配置映射、分片、计数、副本等。


```json
POST _reindex
{
  "source": {
    "index": "my-index-000001"
  },
  "dest": {
    "index": "my-new-index-000001"
  }
}
```

##请求

`POST /_reindex`

##先决条件

* 如果启用了Elasticsearch安全功能，则您必须具有以下安全权限：
	* 源数据流、索引或索引别名的`read`[索引权限]()。
	* 目标数据流、索引或索引别名的`write`索引权限。
	* 若要使用 reindex API 请求自动创建数据流或索引，必须对目标数据流、索引或索引别名具有`auto_configure`, `create_index`, 或 `manage`索引权限。
	* 如果从远程集群重新索引，则`source.remote.user`必须具有源数据流、索引或索引别名的`monitor`[集群权限]()和`read`索引权限。
* 如果从远程集群重新建立索引，则必须在`elasticsearch.yml`配置文件中的`reindex.remote.whitelist`选项显式配置远程主机. 请参见[从远程重新索引]()。
* 自动创建数据流需要启用数据流的匹配索引模板。请参见[设置数据流]()。

##说明

从源索引中提取[文档源]()，并将文档索引到目标索引中。您可以将所有文档复制到目标索引，或重新索引文档的子集。

就像[`_update_by_query`]()一样，`_reindex`获取源的快照，但其目标必须**不同**，这样就不太可能发生版本冲突。`dest`元素可以像index API一样配置，以控制乐观并发控制。省略`version_type`或将其设置为`internal`会导致Elasticsearch盲目地将文档转储到目标中，覆盖所有恰好具有相同ID的文档。

将`version_type`设置为`external`会导致Elasticsearch保留源中的`version`，创建丢失的任何文档，并更新目标中版本比源中版本旧的任何文档。

将`op_type`设置为`create`会导致`_reindex`只在目标中创建缺少的文档。所有现有文档都将导致版本冲突。



> 因为数据流是[只附加(append-only)]()的，所以对目标数据流的任何重新索引请求都必须具有`op_type`类型“create”。重新索引只能向目标数据流添加新文档。它无法更新目标数据流中的现有文档。



默认情况下，版本冲突会中止`_reindex`过程。若要在存在冲突时继续重新索引，请将“`conflicts`”请求主体参数设置为`proceed`。在本例中，响应包含遇到的版本冲突的计数。请注意，其他错误类型的处理不受“`conflicts`”参数的影响。

###异步运行reindex

如果请求包含`wait_for_completion=false`，Elasticsearch将执行一些飞行前检查，启动请求，并返回一个[`task`]()，您可以使用该任务取消或获取该任务的状态。Elasticsearch在`.tasks/_doc/${taskId}`以文档形式创建此任务的记录。完成任务后，应删除任务文档，以便Elasticsearch可以回收空间。

###从多个源reindex

如果有许多源需要重新索引，通常最好一次重新索引一个源，而不是使用glob模式来选取多个源。这样，如果存在任何错误，可以通过删除部分完成的源并重新开始来恢复。它还使并行化过程相当简单：拆分源列表以重新索引，并行运行每个列表。

一次性bash脚本似乎可以很好地实现这一点：

```json
for index in i1 i2 i3 i4 i5; do
  curl -HContent-Type:application/json -XPOST localhost:9200/_reindex?pretty -d'{
    "source": {
      "index": "'$index'"
    },
    "dest": {
      "index": "'$index'-reindexed"
    }
  }'
done
```



