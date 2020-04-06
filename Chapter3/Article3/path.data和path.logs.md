## `path.data`和`path.logs`

如果您使用`.zip`或`.tar.gz`存档，则`data`和`logs` 目录是`$ES_HOME`的子文件夹。如果这些重要文件夹保留在其默认位置，则在将Elasticsearch升级到新版本时，很有可能将其删除。

在生产中使用时，几乎可以肯定要更改data和log文件夹的位置：

```yaml
path:
  logs: /var/log/elasticsearch
  data: /var/data/elasticsearch 
```

RPM和Debian发行版已经为`data`和使用了自定义路径`logs`。

该`path.data`设置可以被设置为多条路径，在这种情况下，所有的路径将被用于存储数据（虽然属于单个碎片文件将全部存储相同的数据路径上）：

```yaml
path:
  data:
    - /mnt/elasticsearch_1
    - /mnt/elasticsearch_2
    - /mnt/elasticsearch_3   
```