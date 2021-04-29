#Put mapping API

向现有数据流或索引添加新字段。您还可以使用put mapping API更改现有字段的搜索设置。



对于数据流，这些更改默认应用于所有备份索引。


```json
PUT /my-index-000001/_mapping
{
  "properties": {
    "email": {
      "type": "keyword"
    }
  }
}
```

> 在7.0.0之前，*映射*定义包含类型名。尽管现在不赞成在请求中指定类型，但如果设置了请求参数`include_type_name`，仍然可以提供类型。有关详细信息，请参阅[删除映射类型]()。

##请求

`PUT /<target>/_mapping`

`PUT /_mapping`



##先决条件

* 如果启用了Elasticsearch安全功能，则必须对目标数据流、索引或索引别名具有`manage` [索引权限]()。



* [在7.9中已弃用]如果请求以索引或索引别名为目标，您还可以使用`create`, `create_doc`, `index`或`write`权限更新其映射。


##路径参数

###`<target>`

（可选，字符串）使用逗号分隔的数据流、索引和索引别名的列表，用于限制请求。支持通配符表达式（`*`）。



若要以群集中的所有数据流和索引为目标，请省略此参数或使用`_all`或`*`。

##请求参数

###`allow_no_indices`

（可选，布尔值）如果为`false`，则如果任何通配符表达式、[索引别名]()或`_all`值没有命中或只命中关闭的索引，则请求将返回错误。即使请求命中了其他已开启的索引，此行为也适用。例如，如果有索引以`foo`开头，而没有索引以`bar`开头，则以`foo*,bar*`为目标的请求将返回错误。



默认为`false`。



###`expand_wildcards`

（可选，字符串）通配符表达式可以匹配的索引类型。如果请求可以以数据流为目标，则此参数确定通配符表达式是否匹配隐藏的数据流。支持逗号分隔的值，例如`open、hidden`。有效值为：



* ####`all`

 匹配任何数据流或索引，包括隐藏的数据流或索引。

* ####`open`

 匹配开放的、非隐藏的索引。也匹配任何非隐藏的数据流。

* ####`closed`

 匹配封闭的非隐藏索引。也匹配任何非隐藏的数据流。数据流不能是关闭的。

* ####`hidden`

 匹配隐藏数据流和隐藏索引。必须与`open, closed`或两者结合使用。

* ####`none`

 不接受通配符表达式。

默认为`open`。



###`include_type_name`

[在7.0.0中已弃用]（可选，布尔值）如果为`true`，则映射体中应包含映射类型。默认为`false`。

###`ignore_unavailable`

（可选，布尔值）如果为`true`，则响应中不包括缺失或关闭的索引。默认为`false`。

###`master_timeout`

（可选，[时间单位]()）等待连接到主节点的时间。如果在超时过期之前没有收到响应，则请求失败并返回错误。默认为`30秒`。

###`timeout`

（可选，[时间单位]()）等待响应的时间。如果在超时过期之前没有收到响应，则请求失败并返回错误。默认为`30秒`。

###`write_index_only`

（可选，布尔值）如果为`true`，则映射仅应用于当前写入的目标索引。默认为`false`。

##请求正文

###`properties`

（必需，[映射对象]()）字段的映射。对于新字段，此映射可以包括：

* 字段名称
* [字段数据类型]()
* [映射参数]()

有关现有字段，请参见[更改现有字段的映射]()。

##范例

###单目标示例

put mapping API需要一个现有的数据流或索引。下面的[create index]() API请求创建了一个不带映射的`publications`索引。

```json
PUT /publications
```

下面的put mapping API请求将`title`（一个新的[`text`]()字段）添加到`publications`索引中。


```json
PUT /publications/_mapping
{
  "properties": {
    "title":  { "type": "text"}
  }
}
```

###多目标

put mapping API可以通过一个请求应用于多个数据流或索引。例如，可以同时更新索引`my-index-000001`和索引`my-index-000002`的映射：


```json
# Create the two indices
PUT /my-index-000001
PUT /my-index-000002

# Update both mappings
PUT /my-index-000001,my-index-000002/_mapping
{
  "properties": {
    "user": {
      "properties": {
        "name": {
          "type": "keyword"
        }
      }
    }
  }
}
```

###向现有对象字段添加新属性

可以使用put mapping API向现有[`object`]()字段添加新属性。要了解其工作原理，请尝试以下示例。



使用[create index]() API创建一个包含`name`对象字段和内部`first`文本字段的索引。


```json
PUT /my-index-000001
{
  "mappings": {
    "properties": {
      "name": {
        "properties": {
          "first": {
            "type": "text"
          }
        }
      }
    }
  }
}
```

使用put mapping API向`name`字段添加一个新的内部`last`文本字段。


```json
PUT /my-index-000001/_mapping
{
  "properties": {
    "name": {
      "properties": {
        "last": {
          "type": "text"
        }
      }
    }
  }
}
```

###向现有字段添加多个字段

[多字段]()允许您以不同的方式索引同一字段。可以使用put mapping API更新`fields`映射参数，并为现有字段启用多字段。



要了解其工作原理，请尝试以下示例。



使用[create index]() API创建包含`city`[文本]()字段的索引。


```json
PUT /my-index-000001
{
  "mappings": {
    "properties": {
      "city": {
        "type": "text"
      }
    }
  }
}
```

虽然文本字段对于全文搜索效果很好，但[keyword]()字段不会被分词，对于排序或聚合可能效果更好。



使用put mapping API为`city`字段启用多字段。此请求添加用于排序的`city.raw`关键字多字段。


```json
PUT /my-index-000001/_mapping
{
  "properties": {
    "city": {
      "type": "text",
      "fields": {
        "raw": {
          "type": "keyword"
        }
      }
    }
  }
}
```

###更改现有字段支持的映射参数

每个[映射参数]()的文档指示是否可以使用put mapping API对现有字段进行更新。例如，您可以使用put mapping API来更新[`ignore_above`]()参数。



要了解其工作原理，请尝试以下示例。



使用[create index]() API创建包含`user_id`关键字字段的索引。`user_id`字段的`ignore_above`参数值为`20`。


```json
PUT /my-index-000001
{
  "mappings": {
    "properties": {
      "user_id": {
        "type": "keyword",
        "ignore_above": 20
      }
    }
  }
}
```

使用put mapping API将`ignore_above`参数值更改为`100`。

```json
PUT /my-index-000001/_mapping
{
  "properties": {
    "user_id": {
      "type": "keyword",
      "ignore_above": 100
    }
  }
}
```

###更改现有字段的映射

除了支持的[映射参数]()外，不能更改现有字段的映射或字段类型。更改现有字段可能会使已索引的数据无效。



如果需要更改数据流备份索引中字段的映射，请参阅[更改数据流的映射和设置]()。



如果需要更改字段在其他索引中的映射，请使用正确的映射创建一个新索引，并将数据[重新索引]()到该索引中。



要查看如何更改索引中现有字段的映射，请尝试以下示例。



使用[create index]() API创建一个具有[`long`]()字段类型的`user_id`字段的索引。

```json
PUT /my-index-000001
{
  "mappings" : {
    "properties": {
      "user_id": {
        "type": "long"
      }
    }
  }
}
```

使用[index]() API索引多个具有`user_id`字段值的文档。


```json
POST /my-index-000001/_doc?refresh=wait_for
{
  "user_id" : 12345
}

POST /my-index-000001/_doc?refresh=wait_for
{
  "user_id" : 12346
}
```

要将`user_id`字段更改为[`keyword`]()字段类型，请使用create index API创建具有正确映射的新索引。


```json
PUT /my-new-index-000001
{
  "mappings" : {
    "properties": {
      "user_id": {
        "type": "keyword"
      }
    }
  }
}
```

使用[reindex]() API将文档从旧索引复制到新索引。

```json
POST /_reindex
{
  "source": {
    "index": "my-index-000001"
  },
  "dest": {
    "index": "my-new-index-000001"
  }
}
```

###重命名字段

重命名字段将使已在旧字段名下索引的数据无效。应该添加一个[`alias`]()字段来创建备用字段名。



例如，使用[create index]() API创建一个带有`user_identifier`字段的索引。


```json
PUT /my-index-000001
{
  "mappings": {
    "properties": {
      "user_identifier": {
        "type": "keyword"
      }
    }
  }
}
```

使用put mapping API为现有的`user_identifier`字段添加别名`user_id`字段。


```json
PUT /my-index-000001/_mapping
{
  "properties": {
    "user_id": {
      "type": "alias",
      "path": "user_identifier"
    }
  }
}
```