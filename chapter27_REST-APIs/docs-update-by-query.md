#Update By Query API

更新与指定查询匹配的文档。如果未指定查询，则在索引中的每个文档上执行更新，而不修改源，这对于拾取映射更改非常有用。


```json
POST my-index-000001/_update_by_query?conflicts=proceed
```

##请求

`POST /<index>/_update_by_query`


##说明

您可以使用与[Search API]()相同的语法在请求URI或请求正文中指定查询条件。



当您提交`update by query`请求时，Elasticsearch会在开始处理请求时获取索引的快照，并使用`internal`版本控制更新匹配的文档。当版本匹配时，文档将更新，并且版本号将递增。如果文档在拍摄快照和处理更新操作之间发生更改，则会导致版本冲突，操作将失败。您可以选择统计版本冲突，而不是通过将`conflicts`设置为`proceed`来停止和返回。



> 无法使用`update by query`更新版本为`0`的文档，因为内部版本控制不支持`0`作为有效的版本号。



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



