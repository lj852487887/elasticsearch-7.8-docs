#调整搜索速度

##为文件系统缓存提供内存

Elasticsearch严重依赖文件系统缓存，以加快搜索速度。通常，您应该确保至少有一半的可用内存进入文件系统缓存，以便Elasticsearch可以将索引的热区域保留在物理内存中。

##使用更快的硬件

如果您的搜索是I/O绑定的，您应该研究为文件系统缓存提供更多内存（见上文）或购买更快的驱动器。尤其是SSD驱动器的性能比旋转磁盘更好。始终使用本地存储，应避免使用`NFS`或`SMB`等远程文件系统。还要注意虚拟化存储，如亚马逊的`Elastic Block Storage`。虚拟化存储与Elasticsearch配合得非常好，它非常吸引人，因为它的设置非常快速和简单，但不幸的是，与专用本地存储相比，虚拟化存储在运行过程中固有的速度较慢。如果在`EBS`上放置索引，请确保使用已设置的IOPS，否则操作可能会很快受到限制。

如果您的搜索是CPU受限的，您应该调查购买更快的CPU。

##文档建模

文档应该建模，以便搜索时操作尽可能便宜。

特别是，应避免使用连接关系。[`nested`]()可以使查询速度慢几倍，[父子关系]()可以使查询速度慢几百倍。因此，如果同样的问题可以通过非规范化文档在没有连接的情况下得到解决，那么可以显著地加速。

##搜索尽可能少的字段

一个[`query_string`]() 或者 [`multi_match`]() 搜索目标的字段越多，速度越慢。提高多个字段搜索速度的常用技术是在索引时将其值复制到单个字段中，然后在搜索时使用此字段。这可以通过映射的[`copy-to`]()指令实现自动化，而无需更改文档源。下面是一个包含电影的索引示例，该索引通过将两个值都索引到`name_and_plot`字段来优化搜索电影名称和情节的查询。


```json
PUT movies
{
  "mappings": {
    "properties": {
      "name_and_plot": {
        "type": "text"
      },
      "name": {
        "type": "text",
        "copy_to": "name_and_plot"
      },
      "plot": {
        "type": "text",
        "copy_to": "name_and_plot"
      }
    }
  }
}
```
##索引前数据

您应该在查询中利用模式来优化数据的索引方式。例如，如果您的所有文档都有一个`price`字段，并且大多数查询在固定的范围列表上运行[`range`]()聚合，则可以通过将范围预先索引到索引中并使用[`terms`]()聚合来加快聚合速度。

例如，如果文档看起来像：

```json
PUT index/_doc/1
{
  "designation": "spoon",
  "price": 13
}
```

搜索请求如下所示：

```json
GET index/_search
{
  "aggs": {
    "price_ranges": {
      "range": {
        "field": "price",
        "ranges": [
          { "to": 10 },
          { "from": 10, "to": 100 },
          { "from": 100 }
        ]
      }
    }
  }
}
```

然后，可以在索引时使用`price_range`字段来丰富文档，该字段应映射为[`keyword`]()：

```json
PUT index
{
  "mappings": {
    "properties": {
      "price_range": {
        "type": "keyword"
      }
    }
  }
}

PUT index/_doc/1
{
  "designation": "spoon",
  "price": 13,
  "price_range": "10-100"
}
```

然后搜索请求可以聚合这个新字段，而不是在`price`字段上运行范围聚合。

```json
GET index/_search
{
  "aggs": {
    "price_ranges": {
      "terms": {
        "field": "price_range"
      }
    }
  }
}
```

##考虑将标识符映射为关键字

并非所有数值数据都应映射为数值字段数据类型。Elasticsearch为范围查询优化数值字段，如整型或长型。但是，关键字字段更适合术语和其他术语级查询。

范围查询中很少使用诸如ISBN或产品ID之类的标识符。但是，它们通常使用术语级查询进行检索。

如果将数字标识符映射为关键字，请考虑：

您不打算使用范围查询搜索标识符数据。

快速检索很重要。关键字字段上的术语查询搜索通常比数字字段上的术语搜索快。

如果不确定要使用哪个字段，可以使用多字段将数据映射为关键字和数字数据类型。

##避免使用脚本

如果可能，避免在搜索中使用[脚本]()或[脚本字段]()。请参阅[脚本和搜索速度]()。

##搜索四舍五入日期

对使用`now`的日期字段的查询通常不可缓存，因为要匹配的范围一直在更改。然而，就用户体验而言，切换到四舍五入日期通常是可以接受的，并且具有更好地利用查询缓存的好处。

例如，下面的查询：

```json
PUT index/_doc/1
{
  "my_date": "2016-05-11T16:30:55.328Z"
}

GET index/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "range": {
          "my_date": {
            "gte": "now-1h",
            "lte": "now"
          }
        }
      }
    }
  }
}
```
可以替换为以下查询：

```json
GET index/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "range": {
          "my_date": {
            "gte": "now-1h/m",
            "lte": "now/m"
          }
        }
      }
    }
  }
}
```

##预热文件系统缓存

如果重新启动运行Elasticsearch的计算机，则文件系统缓存将为空，因此操作系统需要一段时间才能将索引的热点区域加载到内存中，以便快速执行搜索操作。您可以使用[`index.store.preload`]()设置，根据文件扩展名明确告诉操作系统哪些文件应该加载到内存中。

> 如果文件系统缓存不够大，无法容纳所有数据，那么在太多索引或太多文件上急切地将数据加载到文件系统缓存会使搜索速度变慢。小心使用。

##使用索引排序加快连接

为了使连接更快，[索引排序]()很有用，但代价是索引速度稍慢。请在[索引排序文档]()中了解更多信息。

##使用`preference`优化缓存利用率

有多个缓存可以帮助提高搜索性能，例如[文件系统缓存]()、[请求缓存]()或[查询缓存]()。然而，所有这些缓存都是在节点级别维护的，这意味着如果您连续运行同一请求两次，拥有一个或多个[副本]()，并使用默认路由算法[round-robin]()，那么这两个请求将转到不同的分片片拷贝，从而阻止节点级别的缓存发挥作用。

由于搜索应用程序的用户通常会一个接一个地运行类似的请求，例如为了分析索引的较窄子集，因此使用标识当前用户或会话的首选项值可以帮助优化缓存的使用。

##副本可能有助于提高吞吐量，但并非总是如此

除了提高恢复能力之外，副本还可以帮助提高吞吐量。例如，如果您有一个单分片的索引和三个节点，则需要将副本数设置为2，以便总共有3个分片副本，以便利用所有节点。

现在假设您有一个2分片的索引和两个节点。在一种情况下，副本的数量为0，这意味着每个节点持有一个分片。在第二种情况下，副本的数量是1，这意味着每个节点有两个分片。哪种设置在搜索性能方面表现最好？通常，每个节点的碎片总数越少的设置性能越好。这样做的原因是，它为每个分片提供了更大的可用文件系统缓存份额，而文件系统缓存可能是Elasticsearch的头号性能因素。同时，请注意，如果单个节点出现故障，则没有副本的设置可能会失败，因此吞吐量和可用性之间需要权衡。

那么，正确的副本数量是多少？如果您的集群总共有`num_nodes`个节点、`num_primaries`个主分片，并且如果您希望最多能够一次处理`max_failures`节点故障，那么适合您的副本数量应该是`max(max_failures, ceil(num_nodes / num_primaries) - 1)`。
