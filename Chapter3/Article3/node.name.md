## `node.name`

Elasticsearch `node.name`用作Elasticsearch 特定实例的人类可读标识符，因此它包含在许多API的响应中。它默认为计算机在Elasticsearch启动时具有的主机名，但可以`elasticsearch.yml`按以下方式显式配置 ：

```yaml
node.name: prod-data-2
```

