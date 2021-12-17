#注册快照存储库

必须先注册快照存储库，然后才能执行快照和还原操作。使用[put snapshot repository API]()注册或更新快照存储库。我们建议为每个主要版本创建一个新的快照存储库。有效的存储库设置取决于存储库类型。

如果将同一快照存储库注册到多个集群，则只有一个集群应具有对存储库的写访问权限。连接到该存储库的所有其他集群应将存储库设置为只读模式。

> 快照格式可以跨主要版本更改，因此，如果不同版本上的集群试图写入同一存储库，则一个版本写入的快照可能对另一个版本不可见，并且存储库可能已损坏。虽然将存储库设置为除一个集群外的所有集群上的`readonly`存储库都应适用于多个不同于一个主要版本的集群，但不支持此配置。

```json
PUT /_snapshot/my_backup
{
  "type": "fs",
  "settings": {
    "location": "my_backup_location"
  }
}
```

使用[get snapshot API]()检索有关已注册存储库的信息：

```json
GET /_snapshot/my_backup
```

请求返回如下响应：

```json
{
  "my_backup": {
    "type": "fs",
    "settings": {
      "location": "my_backup_location"
    }
  }
}
```

要检索有关多个存储库的信息，请指定以逗号分隔的存储库列表。在指定存储库名称时，还可以使用通配符（*）。例如，以下请求检索有关以repo开头或包含backup的所有快照存储库的信息：

```json
GET /_snapshot/repo*,*backup*
```

要检索有关所有已注册快照存储库的信息，请省略存储库名称：

```json
GET /_snapshot
```

或者，你也可以指定 `_all`

```json
GET /_snapshot/_all
```

你可以使用[delete snapshot repository API]()移除存储库

```json
DELETE /_snapshot/my_backup
```
取消注册存储库时，Elasticsearch仅删除对存储库存储快照的位置的引用。快照本身保持不变并处于原位。

##共享文件系统存储库

共享文件系统存储库（`"type": "fs"`）使用共享文件系统存储快照。为了注册共享文件系统存储库，必须将相同的共享文件系统挂载到所有主节点和数据节点上的相同位置。必须在所有主节点和数据节点上的`path.repo`设置中注册此位置（或其父目录之一）。

假设共享文件系统已装入`/mount/backups/my_fs_backup_location`，则应将以下设置添加到`elasticsearch.yml`文件中：

```json
path.repo: ["/mount/backups", "/mount/longterm_backups"]
```
`path.repo`设置支持Microsoft Windows UNC路径，只要至少将服务器名称和共享指定为前缀，并正确转义反斜杠：








