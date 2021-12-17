#JVM致命错误日志设置

默认情况下，Elasticsearch将JVM配置为将致命错误日志写入默认日志目录。在[RPM]()和[Debian]()软件包上，这个目录是`/var/log/elasticsearch`。在[Linux、MacOS]()和[Windows]()发行版上，日志目录位于Elasticsearch安装的根目录下。



这些是JVM遇到致命错误（如分段错误）时生成的日志。如果此路径不适合接收日志，请修改`-XX:ErrorFile=...`进入[`jvm.options`]()选项.
