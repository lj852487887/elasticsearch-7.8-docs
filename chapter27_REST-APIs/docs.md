#Document APIs

本节首先简要介绍Elasticsearch的[数据复制模型]()，然后详细介绍以下CRUD API：

##单文档 APIs

* [Index]()
* [Get]()
* [Delete]()
* [Update]()

##多文档 APIs

* [Multi get]()
* [Bulk]()
* [Delete by query]()
* [Update by query]()
* [Reindex](docs-reindex.md)


> 所有CRUD API都是单索引API。`index`参数接受单个索引名称或指向单个索引的`alias`。