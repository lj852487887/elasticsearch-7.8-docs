##信息输出：搜索和分析

虽然您可以将Elasticsearch用作文档存储和检索文档及其元数据，但真正的强大之处在于能够轻松访问构建在Apache Lucene搜索引擎库上的全套搜索功能。

Elasticsearch提供了一个简单、一致的REST API，用于管理集群、索引和搜索数据。出于测试目的，您可以轻松地直接从命令行或通过Kibana中的开发人员控制台提交请求。从您的应用程序中，您可以使用[Elasticsearch客户端]()来选择您的语言：Java、JavaScript、Go、.NET、PHP、Perl、Python或Ruby。

###搜索数据

Elasticsearch REST API支持结构化查询、全文查询和将两者结合起来的复杂查询。结构化查询类似于可以在SQL中构造的查询类型。例如，您可以搜索`employee`索引中的`gender`和`age`字段，并按`hire_date`字段对匹配项进行排序。全文查询将查找与查询字符串匹配的所有文档，并返回按*相关性*排序的文档它们与搜索词的匹配程度如何。



除了搜索单个术语外，还可以执行短语搜索、相似性搜索和前缀搜索，并获取自动完成建议。



是否有要搜索的地理空间或其他数字数据？Elasticsearch支持在高性能地理和数字查询的优化数据结构中索引非文本数据。



您可以使用Elasticsearch的综合JSON风格查询语言（[Query DSL]()）访问所有这些搜索功能。您还可以构造[SQL-style queries]()来搜索和聚合Elasticsearch内部的本机数据，JDBC和ODBC驱动程序使各种第三方应用程序能够通过SQL与Elasticsearch交互。



###分析数据

Elasticsearch聚合使您能够构建数据的复杂摘要，并深入了解关键指标、模式和趋势。聚合让您能够回答以下问题，而不仅仅是找到众所周知的“干草堆里捞针”：



* 干草堆里针有多少根？
* 针的平均长度是多少？
* 按制造商细分的针的平均长度是多少？
* 在过去的六个月里，每一个干草堆里加了多少针？

您还可以使用聚合来回答更微妙的问题，例如：

* 你们最受欢迎的针制造商是什么？
* 是否有异常或异常的针束？

因为聚合利用了用于搜索的相同数据结构，所以它们的速度也非常快。这使您能够实时分析和可视化数据。您的报告和仪表盘会随着数据的更改而更新，以便您可以根据最新信息采取行动。



此外，聚合与搜索请求一起运行。您可以搜索文档，过滤结果，并在同一时间，对同一数据，在一个单一的请求执行分析。由于聚合是在特定搜索的上下文中计算的，因此您不仅显示了所有70号针的计数，还显示了与用户搜索条件匹配的70号针的计数—例如，所有70号不粘绣花针。


###但是等等，还有更多

想自动分析时间序列数据吗？您可以使用[机器学习]()功能创建数据中正常行为的准确基线，并识别异常模式。通过机器学习，您可以检测：


* 与值、计数或频率的时间偏差有关的异常
* 统计稀有性
* 群体成员的不寻常行为

最好的部分呢？您可以这样做，而不必指定算法、模型或其他与数据科学相关的配置。