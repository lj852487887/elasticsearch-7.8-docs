#集群名称设置

节点只能在共享`cluster.name`与群集中的所有其他节点时加入群集。默认名称是`elasticsearch`，但您应该将其更改为描述集群用途的适当名称。

```json
cluster.name: logging-prod
```

不要在不同的环境中重用相同的集群名称。否则，节点可能会加入错误的集群。