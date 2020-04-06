## `cluster.name`

当`cluster.name`节点与集群中的所有其他节点共享节点时，该节点只能加入集群。默认名称为`elasticsearch`，但您应将其更改为描述群集用途的适当名称。

```yaml
cluster.name: logging-prod
```



确保不要在不同的环境中重复使用相同的集群名称，否则最终可能会导致节点加入错误的集群。