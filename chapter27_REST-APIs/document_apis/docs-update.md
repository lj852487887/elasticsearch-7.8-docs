#Update API

使用指定脚本更新一个文档

##请求

`POST /<index>/_update/<_id>`

##描述

Enables you to script document updates. The script can update, delete, or skip modifying the document. The update API also supports passing a partial document, which is merged into the existing document. To fully replace an existing document, use the index API.

This operation:

Gets the document (collocated with the shard) from the index.
Runs the specified script.
Indexes the result.
The document must still be reindexed, but using update removes some network roundtrips and reduces chances of version conflicts between the GET and the index operation.

The _source field must be enabled to use update. In addition to _source, you can access the following variables through the ctx map: _index, _type, _id, _version, _routing, and _now (the current timestamp).

##路径参数

<index>

(Required, string) Name of the target index. By default, the index is created automatically if it doesn’t exist. For more information, see Create indices automatically.

<_id>

(Required, string) Unique identifier for the document to be updated.
Query parameters
if_seq_no
(Optional, integer) Only perform the operation if the document has this sequence number. See Optimistic concurrency control.
if_primary_term
(Optional, integer) Only perform the operation if the document has this primary term. See Optimistic concurrency control.
lang
(Optional, string) The script language. Default: painless.
refresh
(Optional, enum) If true, Elasticsearch refreshes the affected shards to make this operation visible to search, if wait_for then wait for a refresh to make this operation visible to search, if false do nothing with refreshes. Valid values: true, false, wait_for. Default: false.
retry_on_conflict
(Optional, integer) Specify how many times should the operation be retried when a conflict occurs. Default: 0.
routing
(Optional, string) Target the specified primary shard.
_source
(Optional, list) Set to false to disable source retrieval (default: true). You can also specify a comma-separated list of the fields you want to retrieve.
_source_excludes
(Optional, list) Specify the source fields you want to exclude.
_source_includes
(Optional, list) Specify the source fields you want to retrieve.
master_timeout
(Optional, time units) Specifies the period of time to wait for a connection to the master node. If no response is received before the timeout expires, the request fails and returns an error. Defaults to 30s.
timeout
(Optional, time units) Specifies the period of time to wait for a response. If no response is received before the timeout expires, the request fails and returns an error. Defaults to 30s.
wait_for_active_shards
(Optional, string) The number of shard copies that must be active before proceeding with the operation. Set to all or any positive integer up to the total number of shards in the index (number_of_replicas+1). Default: 1, the primary shard.

See Active shards.

##范例

首先，我们索引一个简单的文档：

```json
PUT test/_doc/1
{
  "counter" : 1,
  "tags" : ["red"]
}
```

 
To increment the counter, you can submit an update request with the following script:

```json
POST test/_update/1
{
  "script" : {
    "source": "ctx._source.counter += params.count",
    "lang": "painless",
    "params" : {
      "count" : 4
    }
  }
}
```
 
Similarly, you could use and update script to add a tag to the list of tags (this is just a list, so the tag is added even it exists):

POST test/_update/1
{
  "script": {
    "source": "ctx._source.tags.add(params.tag)",
    "lang": "painless",
    "params": {
      "tag": "blue"
    }
  }
}
Copy as curl
View in Console
 
You could also remove a tag from the list of tags. The Painless function to remove a tag takes the array index of the element you want to remove. To avoid a possible runtime error, you first need to make sure the tag exists. If the list contains duplicates of the tag, this script just removes one occurrence.

```json
POST test/_update/1
{
  "script": {
    "source": "if (ctx._source.tags.contains(params.tag)) { ctx._source.tags.remove(ctx._source.tags.indexOf(params.tag)) }",
    "lang": "painless",
    "params": {
      "tag": "blue"
    }
  }
}
```
 
You can also add and remove fields from a document. For example, this script adds the field new_field:

```json
POST test/_update/1
{
  "script" : "ctx._source.new_field = 'value_of_new_field'"
}
```
 
Conversely, this script removes the field new_field:

```json
POST test/_update/1
{
  "script" : "ctx._source.remove('new_field')"
}
```
 
Instead of updating the document, you can also change the operation that is executed from within the script. For example, this request deletes the doc if the tags field contains green, otherwise it does nothing (noop):

POST test/_update/1
{
  "script": {
    "source": "if (ctx._source.tags.contains(params.tag)) { ctx.op = 'delete' } else { ctx.op = 'none' }",
    "lang": "painless",
    "params": {
      "tag": "green"
    }
  }
}
Copy as curl
View in Console
 
Update part of a document
The following partial update adds a new field to the existing document:

POST test/_update/1
{
  "doc": {
    "name": "new_name"
  }
}
Copy as curl
View in Console
 
If both doc and script are specified, then doc is ignored. If you specify a scripted update, include the fields you want to update in the script.

Detect noop updates
By default updates that don’t change anything detect that they don’t change anything and return "result": "noop":

POST test/_update/1
{
  "doc": {
    "name": "new_name"
  }
}
Copy as curl
View in Console
 
If the value of name is already new_name, the update request is ignored and the result element in the response returns noop:

{
   "_shards": {
        "total": 0,
        "successful": 0,
        "failed": 0
   },
   "_index": "test",
   "_type": "_doc",
   "_id": "1",
   "_version": 7,
   "_primary_term": 1,
   "_seq_no": 6,
   "result": "noop"
}
You can disable this behavior by setting "detect_noop": false:

```json
POST test/_update/1
{
  "doc": {
    "name": "new_name"
  },
  "detect_noop": false
}
```
 
###Upsert

如果文档不存在，`upsert`元素的内容将作为新文档插入。如果文档存在，则执行`script`：

```json
POST test/_update/1
{
  "script": {
    "source": "ctx._source.counter += params.count",
    "lang": "painless",
    "params": {
      "count": 4
    }
  },
  "upsert": {
    "counter": 1
  }
}
```

Scripted upsert
To run the script whether or not the document exists, set scripted_upsert to true:

```json
POST sessions/_update/dh3sgudg8gsrgl
{
  "scripted_upsert": true,
  "script": {
    "id": "my_web_session_summariser",
    "params": {
      "pageViewEvent": {
        "url": "foo.com/bar",
        "response": 404,
        "time": "2014-01-01 12:32"
      }
    }
  },
  "upsert": {}
}
```
 
Doc as upsert
Instead of sending a partial doc plus an upsert doc, you can set doc_as_upsert to true to use the contents of doc as the upsert value:

```json
POST test/_update/1
{
  "doc": {
    "name": "new_name"
  },
  "doc_as_upsert": true
}
```
 
Using ingest pipelines with doc_as_upsert is not supported.