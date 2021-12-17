#启用Elasticsearch安全功能

使用基本许可证和试用许可证时，默认情况下Elasticsearch安全功能是禁用的。为了启用它：

1. 停止Kibana。启动和停止Kibana的方法因安装方式而异。例如，如果您是从存档发行版（`.tar.gz`或`.zip`）安装Kibana的，请通过在命令行中输入`Ctrl-C`来停止它。参见[启动和停止Kibana]()。

1. 停止弹性搜索。例如，如果从存档发行版安装了Elasticsearch，请在命令行中输入`Ctrl-C`。请参阅[停止Elasticsearch]()。

1. 将`xpack.security.enabled`设置添加到`ES_PATH_CONF/elasticsearch.yml`文件中。


> `ES_PATH_CONF`环境变量包含Elasticsearch配置文件的路径。如果您使用归档发行版（`zip`或`tar.gz`）安装了Elasticsearch，它默认为`ES_HOME/config`。如果使用包发行版（Debian或RPM），则默认为`/etc/elasticsearch`。有关更多信息，请参阅[配置Elasticsearch]()。

例如，添加以下设置：

```yaml
xpack.security.enabled: true
```

> 如果您有基本许可证或试用许可证，则此设置的默认值为`false`。如果您拥有`gold`或更高的许可证，则默认值为`true`。因此，最好明确添加此设置，以避免混淆是否启用了安全功能。



本教程涉及单节点集群，但如果您有多个节点，则可以在集群中的每个节点上启用Elasticsearch安全功能，并为节点间通信配置传输层安全性（TLS），这超出了本教程的范围。通过启用单节点发现，我们推迟了TLS的配置。例如，添加以下设置：

```yaml
discovery.type:single-node
```

有关详细信息，请参见单节点发现。

启用Elasticsearch安全功能时，默认情况下会启用基本身份验证。要与集群通信，必须指定用户名和密码。除非启用匿名访问，否则所有不包含用户名和密码的请求都将被拒绝。

2. 在`ES_PATH_CONF/elasticsearch.yml`文件中启用单节点发现。







