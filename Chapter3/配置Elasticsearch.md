## 配置Elasticsearch

Elasticsearch具有良好的默认设置，并且只需要很少的配置。可以使用[群集更新设置](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/cluster-update-settings.html) API 在正在运行的群集上更改大多数设置 。

配置文件应包含特定于节点的设置（例如`node.name`和paths），或包含节点才能加入集群所需的设置，例如`cluster.name`和`network.host`。

### 本地配置文件

Elasticsearch具有三个配置文件：

- `elasticsearch.yml` 用于配置Elasticsearch
- `jvm.options` 用于配置Elasticsearch JVM设置
- `log4j2.properties` 用于配置Elasticsearch日志记录

这些文件位于config目录中，其默认位置取决于安装是来自归档发行版（`tar.gz`或 `zip`）还是软件包发行版（Debian或RPM软件包）。

对于存档分发，配置目录位置默认为 `$ES_HOME/config`。可以通过`ES_PATH_CONF`环境变量来更改config目录的位置， 如下所示：

```sh
ES_PATH_CONF=/path/to/my/config ./bin/elasticsearch
```

或者，您可以通过命令行或您的Shell配置文件来`export`使用`ES_PATH_CONF`环境变量。

对于软件包分发，config目录位置默认为 `/etc/elasticsearch`。config目录的位置也可以通过`ES_PATH_CONF`环境变量来更改，但是请注意，在您的shell中进行设置是不够的。而是，此变量来自 `/etc/default/elasticsearch`（对于Debian软件包）和 `/etc/sysconfig/elasticsearch`（对于RPM软件包）。您将需要相应地`ES_PATH_CONF=/etc/elasticsearch`在这些文件之一中编辑 条目，以更改配置目录位置。

### 配置文件格式

配置格式为[YAML](http://www.yaml.org/)。这是更改数据和日志目录的路径的示例：

```yaml
path:
    data: /var/lib/elasticsearch
    logs: /var/log/elasticsearch   
```

设置也可以按以下方式展平：

```yaml
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch
```

### 环境变量替换

`${...}`在配置文件中用符号引用的环境变量将替换为环境变量的值，例如：

```yaml
node.name:    ${HOSTNAME}
network.host: ${ES_NETWORK_HOST}
```

