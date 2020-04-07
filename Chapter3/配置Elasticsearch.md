[toc]

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

## 设置JVM选项

您几乎不需要更改Java虚拟机（JVM）选项。如果这样做，最可能的更改是设置[堆大小](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/heap-size.html)。本文档的其余部分详细说明了如何设置JVM选项。您可以使用`jvm.options`文件或`ES_JAVA_OPTS`环境变量来设置选项。

设置或覆盖JVM选项的首选方法是通过JVM选项文件。从tar或zip发行版安装时，根`jvm.options` 配置文件为，`config/jvm.options`并且可以将自定义JVM选项文件添加到`config/jvm.options.d/`。从Debian或RPM软件包安装时，根`jvm.options`配置文件为， `/etc/elasticsearch/jvm.options`并且可以将自定义JVM选项文件添加到 `/etc/elasticsearch/jvm.options.d/`。使用[Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/docker.html)的[Docker发行版时，](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/docker.html)您可以将装入的自定义JVM选项文件绑定到中 `/usr/share/elasticsearch/config/jvm.options.d/`。您永远不需要修改根`jvm.options`文件，而宁愿使用自定义JVM选项文件。自定义JVM选项的处理顺序是按字典顺序。

JVM选项文件必须具有后缀*.options，*并包含遵循特殊语法的以行分隔的JVM参数列表：

- 仅由空格组成的行将被忽略

- 以开头的行`#`被视为注释，并被忽略

    ```text
    # this is a comment
    ```

- 以`-`开头的行被视为独立于JVM版本而应用的JVM选项

    ```text
    - Xmx2g
    ```

- 以数字开头`:`后接`-`的行被视为仅在JVM版本与该数字匹配时才适用的JVM选项

    ```text
    8:-Xmx2g
    ```

- 以数字开头`-`后接`:`的行被视为JVM选项，仅在JVM版本大于或等于该数字时才适用

    ```text
    8-:-Xmx2g
    ```

- 以数字开头`-`后接数字，后跟`:`的行被视为JVM选项，仅当JVM版本在两个数字范围内时才适用

    ```text
    8-9:-Xmx2g
    ```

- 其他所有行均被拒绝

设置Java虚拟机选项的另一种机制是通过 `ES_JAVA_OPTS`环境变量。例如：

```sh
export ES_JAVA_OPTS="$ES_JAVA_OPTS -Djava.io.tmpdir=/path/to/temp/dir"
./bin/elasticsearch
```

使用RPM或Debian软件包时，`ES_JAVA_OPTS`可以在[系统配置文件中](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/setting-system-settings.html#sysconfig)指定 。

JVM具有用于观察`JAVA_TOOL_OPTIONS` 环境变量的内置机制。我们有意在打包脚本中忽略此环境变量。这样做的主要原因是，在某些操作系统（例如Ubuntu）上，默认情况下通过此环境变量安装了代理，我们不希望它们干扰Elasticsearch。

此外，其他一些Java程序也支持`JAVA_OPTS`环境变量。这**不是** JVM内置的机制，而是生态系统中的约定。但是，我们不支持此环境变量，而是通过上述`jvm.options`文件或环境变量来支持设置JVM选项`ES_JAVA_OPTS`。

## 日志配置

Elasticsearch使用[Log4j 2](https://logging.apache.org/log4j/2.x/)进行日志记录。可以使用log4j2.properties文件配置Log4j 2。Elasticsearch公开三个属性`${sys:es.logs.base_path}`， `${sys:es.logs.cluster_name}`以及`${sys:es.logs.node_name}`可以在配置文件中引用，以确定日志文件的位置。该属性`${sys:es.logs.base_path}`将解析为日志目录， `${sys:es.logs.cluster_name}`并将解析为群集名称（在默认配置中用作日志文件名的前缀）， `${sys:es.logs.node_name}`并将解析为节点名称（如果显式设置了节点名称）。

例如，如果你的日志目录（`path.logs`）是`/var/log/elasticsearch`和您的群集名为`production`然后`${sys:es.logs.base_path}`将解析`/var/log/elasticsearch`和 `${sys:es.logs.base_path}${sys:file.separator}${sys:es.logs.cluster_name}.log` 将解析`/var/log/elasticsearch/production.log`。

```properties
######## Server JSON ############################
appender.rolling.type = RollingFile 
appender.rolling.name = rolling
appender.rolling.fileName = ${sys:es.logs.base_path}${sys:file.separator}${sys:es.logs.cluster_name}_server.json 
appender.rolling.layout.type = ESJsonLayout 
appender.rolling.layout.type_name = server 
appender.rolling.filePattern = ${sys:es.logs.base_path}${sys:file.separator}${sys:es.logs.cluster_name}-%d{yyyy-MM-dd}-%i.json.gz 
appender.rolling.policies.type = Policies
appender.rolling.policies.time.type = TimeBasedTriggeringPolicy 
appender.rolling.policies.time.interval = 1 
appender.rolling.policies.time.modulate = true 
appender.rolling.policies.size.type = SizeBasedTriggeringPolicy 
appender.rolling.policies.size.size = 256MB 
appender.rolling.strategy.type = DefaultRolloverStrategy
appender.rolling.strategy.fileIndex = nomax
appender.rolling.strategy.action.type = Delete 
appender.rolling.strategy.action.basepath = ${sys:es.logs.base_path}
appender.rolling.strategy.action.condition.type = IfFileName 
appender.rolling.strategy.action.condition.glob = ${sys:es.logs.cluster_name}-* 
appender.rolling.strategy.action.condition.nested_condition.type = IfAccumulatedFileSize 
appender.rolling.strategy.action.condition.nested_condition.exceeds = 2GB 
################################################
```

可以查看[释义](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/logging.html)

```properties
######## Server -  old style pattern ###########
appender.rolling_old.type = RollingFile
appender.rolling_old.name = rolling_old
appender.rolling_old.fileName = ${sys:es.logs.base_path}${sys:file.separator}${sys:es.logs.cluster_name}_server.log 
appender.rolling_old.layout.type = PatternLayout
appender.rolling_old.layout.pattern = [%d{ISO8601}][%-5p][%-25c{1.}] [%node_name]%marker %m%n
appender.rolling_old.filePattern = ${sys:es.logs.base_path}${sys:file.separator}${sys:es.logs.cluster_name}-%d{yyyy-MM-dd}-%i.old_log.gz
```

`old style`模式追加器的配置。这些日志将保存在`*.log`文件中，如果存档则将保存在文件中`* .log.gz`。请注意，应将其视为已弃用，并将在将来删除。

> Log4j的配置解析被任何多余的空白所迷惑；如果您在此页面上复制并粘贴任何Log4j设置，或通常输入任何Log4j配置，请确保修剪所有前导和尾随空格。

请注意，您可以替换`.gz`为`.zip`in `appender.rolling.filePattern`以使用zip格式压缩滚动日志。如果删除`.gz` 扩展名，则日志在滚动时将不会被压缩。

如果要在指定时间段内保留日志文件，则可以将过渡策略与删除操作一起使用。

```properties
appender.rolling.strategy.type = DefaultRolloverStrategy 
appender.rolling.strategy.action.type = Delete 
appender.rolling.strategy.action.basepath = ${sys:es.logs.base_path} 
appender.rolling.strategy.action.condition.type = IfFileName 
appender.rolling.strategy.action.condition.glob = ${sys:es.logs.cluster_name}-* 
appender.rolling.strategy.action.condition.nested_condition.type = IfLastModified 
appender.rolling.strategy.action.condition.nested_condition.age = 7D 
```

点击查看[释义](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/logging.html)

可以加载多个配置文件（在这种情况下，它们将被合并），只要它们被命名`log4j2.properties`并将Elasticsearch config目录作为祖先即可。这对于公开其他记录器的插件很有用。记录器部分包含Java软件包及其相应的日志级别。附加程序部分包含日志的目标。在[Log4j文档中](http://logging.apache.org/log4j/2.x/manual/configuration.html)可以找到有关如何自定义日志记录和所有受支持的附加程序的广泛信息 。

### 配置日志记录级别

有四种配置日志记录级别的方法，每种方法都有适合使用的情况。

1. 通过命令行：`-E <name of logging hierarchy>=<level>`(例如， `-E logger.org.elasticsearch.transport=trace`）。当您临时调试单个节点上的问题（例如，启动问题或开发过程中）时，这是最合适的。

2. 通过`elasticsearch.yml`：（ 例如， `logger.org.elasticsearch.transport: trace`）。当您临时调试问题但未通过命令行（例如，通过服务）启动Elasticsearch或希望更永久地调整日志记录级别时，这是最合适的。

3. 通过[集群设置](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/misc-cluster.html#cluster-logger)：

    ```js
    PUT /_cluster/settings
    {
      "transient": {
        "<name of logging hierarchy>": "<level>"
      }
    }
    ```

    例如：

    ```console
    PUT /_cluster/settings
    {
      "transient": {
        "logger.org.elasticsearch.transport": "trace"
      }
    } 
    ```

    当您需要动态地调整正在运行的群集上的日志记录级别时，这是最合适的。

4. 通过`log4j2.properties`：

    ```properties
    logger.<unique_identifier>.name = <name of logging hierarchy>
    logger.<unique_identifier>.level = <level>
    ```

    例如：

    ```properties
    logger.transport.name = org.elasticsearch.transport
    logger.transport.level = trace
    ```

    当您需要对记录器进行细粒度控制时（例如，要将记录器发送到另一个文件或以其他方式管理记录器；这是一种罕见的用例），这是最合适的。

### 过时的日志

除了常规日志记录外，Elasticsearch还允许您启用不赞成使用的操作的日志记录。例如，这使您可以及早确定是否将来需要迁移某些功能。默认情况下，在WARN级别启用弃用日志记录，该级别将发出所有弃用日志消息。

```properties
logger.deprecation.level = warn
```

这将在您的日志目录中创建每日滚动弃用日志文件。定期检查此文件，尤其是在您打算升级到新的主要版本时。

默认的日志记录配置已将弃用日志的滚动策略设置为在1 GB之后滚动和压缩，并最多保留五个日志文件（四个滚动日志和活动日志）。

您可以`config/log4j2.properties`通过将弃用日志级别设置为以下方式在文件中禁用它`error`：

```properties
logger.deprecation.name = org.elasticsearch.deprecation
logger.deprecation.level = error
```

如果`X-Opaque-Id`用作HTTP标头，则可以确定触发过时功能的原因。用户ID包含`X-Opaque-ID`在弃用JSON日志的字段中。

```js
{
  "type": "deprecation",
  "timestamp": "2019-08-30T12:07:07,126+02:00",
  "level": "WARN",
  "component": "o.e.d.r.a.a.i.RestCreateIndexAction",
  "cluster.name": "distribution_run",
  "node.name": "node-0",
  "message": "[types removal] Using include_type_name in create index requests is deprecated. The parameter will be removed in the next major version.",
  "x-opaque-id": "MY_USER_ID",
  "cluster.uuid": "Aq-c-PAeQiK3tfBYtig9Bw",
  "node.id": "D7fUYfnfTLa2D7y-xw6tZg"
}  
```

### JSON日志格式

为了简化对Elasticsearch日志的解析，现在以JSON格式打印日志。这由Log4J布局属性配置`appender.rolling.layout.type = ESJsonLayout`。此布局需要`type_name`设置一个属性，该属性用于在解析时区分日志流。

```properties
appender.rolling.layout.type = ESJsonLayout
appender.rolling.layout.type_name = server
```

每行包含一个JSON文档，其属性在中配置`ESJsonLayout`。有关更多详细信息，请参见此类[javadoc](https://snapshots.elastic.co/javadoc/org/elasticsearch/elasticsearch/7.8.0-SNAPSHOT/org/elasticsearch/common/logging/ESJsonLayout.html)。但是，如果JSON文档包含异常，它将被打印在多行上。第一行将包含常规属性，随后的行将包含格式为JSON数组的stacktrace。

> 您仍然可以使用自己的自定义布局。为此，请`appender.rolling.layout.type`使用其他布局替换该行 。请参阅以下示例：

```properties
appender.rolling.type = RollingFile
appender.rolling.name = rolling
appender.rolling.fileName = ${sys:es.logs.base_path}${sys:file.separator}${sys:es.logs.cluster_name}_server.log
appender.rolling.layout.type = PatternLayout
appender.rolling.layout.pattern = [%d{ISO8601}][%-5p][%-25c{1.}] [%node_name]%marker %.-10000m%n
appender.rolling.filePattern = ${sys:es.logs.base_path}${sys:file.separator}${sys:es.logs.cluster_name}-%d{yyyy-MM-dd}-%i.log.gz
```

## 索引生命周期管理设置

这些是可用于配置索引生命周期管理的设置

### 集群级别设置[编辑](https://github.com/elastic/elasticsearch/edit/7.x/docs/reference/settings/ilm-settings.asciidoc)

- **`xpack.ilm.enabled`**

    无论是启用还是禁用ILM，将其设置为`false`禁用任何ILM REST API端点和功能。默认为`true`。

- **`indices.lifecycle.poll_interval`**

    （[时间单位](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/common-options.html#time-units)）索引生命周期管理检查符合策略标准的索引的频率。默认为`10m`。

- **`indices.lifecycle.history_index_enabled`**

    是否启用ILM的历史记录索引。如果启用，ILM会将作为ILM策略一部分而采取的操作的历史记录到`ilm-history-*` 索引中。默认为`true`。

### 索引级别设置

这些索引级别的ILM设置通常是通过索引模板配置的。有关更多信息，请参阅[创建生命周期策略](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/getting-started-index-lifecycle-management.html#ilm-gs-create-policy)。

- **`index.lifecycle.name`**

    用于管理索引的策略的名称。

- **`index.lifecycle.rollover_alias`**

    索引翻转时要更新的索引别名。在使用包含过渡操作的策略时指定。当索引翻转时，别名将更新以反映该索引不再是写索引。有关过渡的更多信息，请参见[*自动滚动*](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/using-policies-rollover.html)。

- **`index.lifecycle.parse_origination_date`**

    当配置为`true`原始日期时，将从索引名称中解析。索引格式必须匹配pattern `^.*-{date_format}-\\d+`，其中`date_format`is `yyyy.MM.dd`和结尾的数字是可选的（滚动的索引通常会匹配完整格式，例如 `logs-2016.10.31-000002`）。如果索引名称与模式不匹配，则索引创建将失败。

- **`index.lifecycle.origination_date`**

    将用于计算其相变的索引寿命的时间戳。这样，用户可以创建包含旧数据的索引，并使用旧数据的原始创建日期来计算索引使用期限。必须为长（Unix纪元）值。

## 在Elasticsearch监控设置

默认情况下，启用监视，但禁用数据收集。要启用数据收集，请使用该`xpack.monitoring.collection.enabled`设置。

您可以在`elasticsearch.yml`文件中配置这些监视设置。您还可以使用[群集更新设置API](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/cluster-update-settings.html)动态设置其中一些 [设置](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/cluster-update-settings.html)。

> 群集设置优先于`elasticsearch.yml` 文件中的设置。

要调整如何监控数据显示在监控界面，配置 [`xpack.monitoring`设置](https://www.elastic.co/guide/en/kibana/7.x/monitoring-settings-kb.html)中 `kibana.yml`。要控制如何从Logstash收集监视数据，请在中配置监视设置`logstash.yml`。

有关更多信息，请参阅[监视集群](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/monitor-elasticsearch-cluster.html)。

#### 常规监视设置

- **`xpack.monitoring.enabled`**

    设置为`true`（默认值）以对节点上的Elasticsearch启用Elasticsearch X-Pack监视。

    > 要启用数据收集，还必须将设置`xpack.monitoring.collection.enabled` 为`true`。默认值为`false`。

#### 监视收集设置

这些`xpack.monitoring.collection`设置控制如何从Elasticsearch节点收集数据。您可以使用[群集更新设置API](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/cluster-update-settings.html)动态更改所有监视收集设置。

- **`xpack.monitoring.collection.enabled`（[Dynamic](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/cluster-update-settings.html)）**

    在6.3.0中添加。设置为`true`启用监视数据收集。当此设置为`false`默认值时，将不会收集Elasticsearch监视数据，并且会忽略来自其他来源（如Kibana，Beats和Logstash）的所有监视数据。

- **`xpack.monitoring.collection.interval`（[Dynamic](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/cluster-update-settings.html)）**

    设置为`-1`禁用数据收集,从7.0.0开始不再支持 。 在6.3.0中弃用。使用`xpack.monitoring.collection.enabled`设置为`false`来替代。控制收集数据样本的频率。默认为`10s`。如果您修改收集间隔，则将`xpack.monitoring.min_interval_seconds` 选项设置`kibana.yml`为相同的值。

- **`xpack.monitoring.elasticsearch.collection.enabled`（[动态](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/cluster-update-settings.html)）**

    控制是否应收集有关Elasticsearch集群的统计信息。默认为`true`。这与xpack.monitoring.collection.enabled不同，后者允许您启用或禁用所有监视收集。但是，此设置只是禁用了Elasticsearch数据的收集，同时仍然允许其他数据（例如，Kibana，Logstash，Beats或APM Server监视数据）通过此群集。

- **`xpack.monitoring.collection.cluster.stats.timeout`（[动态](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/cluster-update-settings.html)）**

    （[时间值](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/common-options.html#time-units)）用于收集集群统计信息的超时。默认为`10s`。

- **`xpack.monitoring.collection.node.stats.timeout`（[动态](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/cluster-update-settings.html)）**

    （[时间值](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/common-options.html#time-units)）收集节点统计信息的超时[时间](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/common-options.html#time-units)。默认为`10s`。

- **`xpack.monitoring.collection.indices`（[动态](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/cluster-update-settings.html)）**

    控制监控从哪个索引收集数据。默认为所有索引。将索引名称指定为以逗号分隔的列表，例如`test1,test2,test3`。名称可以包括通配符，例如`test*`。您可以在前缀前明确排除索引`-`。例如，`test*,-test3`将监视`test`除以外的所有所有索引`test3`。系统索引（如.security *或.kibana *）始终以开头`.`，通常应对其进行监视。考虑添加`.*`到索引列表中，以确保监视系统索引。例如`.*,test*,-test3`

- **`xpack.monitoring.collection.index.stats.timeout`（[动态](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/cluster-update-settings.html)）**

    （[时间值](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/common-options.html#time-units)）用于收集索引统计信息的超时。默认为`10s`。

- **`xpack.monitoring.collection.index.recovery.active_only`（[动态](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/cluster-update-settings.html)）**

    控制是否收集所有回收率。设置为`true`仅收集活动的恢复。默认为`false`。

- **`xpack.monitoring.collection.index.recovery.timeout`（[动态](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/cluster-update-settings.html)）**

    （[时间值](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/common-options.html#time-units)）用于收集恢复信息的超时。默认为`10s`。

- **`xpack.monitoring.history.duration`（[动态](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/cluster-update-settings.html)）**

    （[时间值](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/common-options.html#time-units)）保留期限，超过该期限后，将自动删除监视导出器创建的索引。默认为`7d`（7天）。此设置的最小值`1d`（1天）以确保正在监视某些内容，并且不能将其禁用。

    > 此设置当前仅影响`local`类型出口商。使用`http`导出器创建的索引不会自动删除。

- **`xpack.monitoring.exporters`**

    配置代理在何处存储监视数据。默认情况下，代理使用本地导出程序，该导出程序对安装了群集的监视数据进行索引。使用HTTP导出器将数据发送到单独的监视群集。有关更多信息，请参阅[本地导出器设置](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/monitoring-settings.html#local-exporter-settings)， [HTTP导出器设置](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/monitoring-settings.html#http-exporter-settings)及其 [*工作方式*](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/how-monitoring-works.html)。

#### 本地出口商设置

该`local`出口是通过监测使用的默认出口国。顾名思义，它会将数据导出到*本地*群集，这意味着不需要太多配置。

如果您不提供*任何*出口商，Monitoring会自动为您创建一个。如果提供了任何导出器，则不添加默认值。

```yaml
xpack.monitoring.exporters.my_local:
  type: local
```

- **`type`**

    本地出口商的值必须始终为`local`并且是必需的。

- **`use_ingest`**

    是否在每个批量请求中向集群和管道处理器提供占位符管道。默认值为`true`。如果被禁用，则意味着它将不使用管道，这意味着将来的发行版无法将批量请求自动升级为可用于将来的请求。

- **`cluster_alerts.management.enabled`**

    是否为此集群创建集群警报。默认值为`true`。要使用此功能，必须启用Watcher。如果您具有基本许可证，则不会显示群集警报。

#### HTTP导出器设置

以下列出了`http`导出器可以提供的设置。所有设置均显示在您为导出器选择的名称后面：

```yaml
xpack.monitoring.exporters.my_remote:
  type: http
  host: ["host:port", ...]
```

- **`type`**

    HTTP导出程序的值必须始终为`http`并且是必需的。

- **`host`**

    主机支持多种格式，既可以是数组，也可以是单个值。支持的格式包括 `hostname`，`hostname:port`，`http://hostname`, `http://hostname:port`，`https://hostname`，和 `https://hostname:port`。不能假定主机。如果未作为字符串的一部分提供，则默认方案始终为`http`，默认端口始终`9200`为`host`。

    ```properties
    xpack.monitoring.exporters:
      example1:
        type: http
        host: "10.1.2.3"
      example2:
        type: http
        host: ["http://10.1.2.4"]
      example3:
        type: http
        host: ["10.1.2.5", "10.1.2.6"]
      example4:
        type: http
        host: ["https://10.1.2.3:9200"]
    ```

    

- **`auth.username`**

    如果提供`auth.secure_password`或，`auth.password`则用户名是必需的。

- **`auth.secure_password`（[安全](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/secure-settings.html)，[可重新加载](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/secure-settings.html#reloadable-secure-settings)）**

    的密码`auth.username`。`auth.password`如果也指定了优先级。

- **`auth.password`**

    的密码`auth.username`。如果`auth.secure_password`还指定，则将忽略此设置。

### 在7.7.0中弃用。

使用`auth.secure_password`代替。

- **`connection.timeout`**

    （[时间值](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/common-options.html#time-units)）HTTP连接应该等待套接字打开以请求的时间。默认值为`6s`。

- **`connection.read_timeout`**

    （[时间值](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/common-options.html#time-units)）HTTP连接应该等待套接字发送回响应的时间。默认值为`10 * connection.timeout`（`60s`如果未设置）。

- **`ssl`**

    每个HTTP导出器都可以定义自己的TLS / SSL设置或继承它们。请参阅[下面](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/monitoring-settings.html#ssl-monitoring-settings)的“ [TLS / SSL”部分](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/monitoring-settings.html#ssl-monitoring-settings)。

- **`proxy.base_path`**

    前缀任何传出请求的基本路径，例如`/base/path`（然后，批量请求将作为发送`/base/path/_bulk`）。没有默认值。

- **`headers`**

    添加到每个请求的可选标头，可以帮助通过代理路由请求。

    ```properties
    xpack.monitoring.exporters.my_remote:
      headers:
        X-My-Array: [abc, def, xyz]
        X-My-Header: abc123
    ```

    发送基于数组的标头的`n`时间`n`为数组的大小。`Content-Type` 并且`Content-Length`无法设置。监视代理程序创建的任何标题都将覆盖此处定义的任何内容。

- **`index.name.time_format`**

    一种用于更改默认情况下每日监控索引的默认日期后缀的机制。默认值为`YYYY.MM.DD`，这就是为什么每天创建索引的原因。

- **`use_ingest`**

    是否在每个批量请求时向监视集群和管道处理器提供占位符管道。默认值为`true`。如果被禁用，则意味着它将不使用管道，这意味着将来的发行版无法将批量请求自动升级为可用于将来的请求。

- **`cluster_alerts.management.enabled`**

    是否为此集群创建集群警报。默认值为`true`。要使用此功能，必须启用Watcher。如果您具有基本许可证，则不会显示群集警报。

- **`cluster_alerts.management.blacklist`**

    阻止创建特定的群集警报。它还会删除当前群集中已经存在的所有适用监视。 您可以将以下任意手表标识符添加到黑名单中：

    - `elasticsearch_cluster_status`
    - `elasticsearch_version_mismatch`
    - `elasticsearch_nodes`
    - `kibana_version_mismatch`
    - `logstash_version_mismatch`
    - `xpack_license_expiration`

    例如：`["elasticsearch_version_mismatch","xpack_license_expiration"]`。

### X-Pack监视TLS / SSL设置

您可以配置以下TLS / SSL设置。

- **`xpack.monitoring.exporters.$NAME.ssl.supported_protocols`**

    支持的协议版本。有效协议：`SSLv2Hello`， `SSLv3`，`TLSv1`，`TLSv1.1`，`TLSv1.2`，`TLSv1.3`。如果JVM的SSL提供程序支持TLSv1.3，则默认值为`TLSv1.3,TLSv1.2,TLSv1.1`。否则，默认值为 `TLSv1.2,TLSv1.1`。如果`xpack.security.fips_mode.enabled`是`true`，则不能使用`SSLv2Hello` 或`SSLv3`。请参阅[FIPS 140-2](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/fips-140-compliance.html)。

- **`xpack.monitoring.exporters.$NAME.ssl.verification_mode`**

    控制证书的验证。有效值为：`full`，它验证所提供的证书是否由受信任的机构（CA）签名，还验证服务器的主机名（或IP地址）是否与证书中标识的名称匹配。`certificate`，它验证提供的证书是否由受信任的权威（CA）签名，但不执行任何主机名验证。`none`，*不对*服务器证书进行*任何验证*。此模式禁用了SSL / TLS的许多安全性优点，只有在仔细考虑后才能使用。它主要是在尝试解决TLS错误时作为一种临时诊断机制。强烈建议不要将其用于生产集群。默认值为`full`。

- **`xpack.monitoring.exporters.$NAME.ssl.cipher_suites`**

    支持的密码套件取决于您使用的Java版本。例如，对于11版的默认值是`TLS_AES_256_GCM_SHA384`， `TLS_AES_128_GCM_SHA256`，`TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384`， `TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256`，`TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384`， `TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256`，`TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384`， `TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256`，`TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384`， `TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256`，`TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA`， `TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA`，`TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA`， `TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA`，`TLS_RSA_WITH_AES_256_GCM_SHA384`， `TLS_RSA_WITH_AES_128_GCM_SHA256`，`TLS_RSA_WITH_AES_256_CBC_SHA256`， `TLS_RSA_WITH_AES_128_CBC_SHA256`，`TLS_RSA_WITH_AES_256_CBC_SHA`， `TLS_RSA_WITH_AES_128_CBC_SHA`。上面的默认密码套件列表包括TLSv1.3密码，以及需要256位AES加密的*Java密码学扩展（JCE）无限强度管辖权策略文件*的密码。如果TLSv1.3是不可用，TLSv1.3密码`TLS_AES_256_GCM_SHA384`和 `TLS_AES_128_GCM_SHA256`不包括在默认列表。如果256位AES不可用，则`AES_256`其名称中带有密码的密码将不包括在默认列表中。最后，AES GCM在11之前的Java版本中存在性能问题，并且仅在使用Java 11或更高版本时才包含在默认列表中。有关更多信息，请参见Oracle的 [Java密码体系结构文档](https://docs.oracle.com/en/java/javase/11/security/oracle-providers.html#GUID-7093246A-31A3-4304-AC5F-5FB6400405E2)。

#### X-Pack监视TLS / SSL密钥和受信任的证书设置

以下设置用于指定通过SSL / TLS连接进行通信时应使用的私钥，证书和受信任证书。私钥和证书是可选的，如果服务器要求客户端认证进行PKI认证，则将使用私钥和证书。

#### PEM编码的文件

使用PEM编码的文件时，请使用以下设置：

- **`xpack.monitoring.exporters.$NAME.ssl.key`**

    包含私钥的PEM编码文件的路径。

- **`xpack.monitoring.exporters.$NAME.ssl.key_passphrase`**

    用于解密私钥的密码。由于密钥可能未加密，因此此值是可选的。

- **`xpack.monitoring.exporters.$NAME.ssl.secure_key_passphrase`（[安全](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/secure-settings.html)）**

    用于解密私钥的密码。由于密钥可能未加密，因此此值是可选的。

- **`xpack.monitoring.exporters.$NAME.ssl.certificate`**

    指定与密钥关联的PEM编码证书（或证书链）的路径。

- **`xpack.monitoring.exporters.$NAME.ssl.certificate_authorities`**

    应当信任的PEM编码证书文件的路径列表。

#### Java密钥库文件

使用包含私钥，证书和应受信任的证书的Java密钥库文件（JKS）时，请使用以下设置：

- **`xpack.monitoring.exporters.$NAME.ssl.keystore.path`**

    包含私钥和证书的密钥库文件的路径。

- **`xpack.monitoring.exporters.$NAME.ssl.keystore.password`**

    密钥库的密码。

- **`xpack.monitoring.exporters.$NAME.ssl.keystore.secure_password`（[安全](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/secure-settings.html)）**

    密钥库的密码。

- **`xpack.monitoring.exporters.$NAME.ssl.keystore.key_password`**

    密钥库中密钥的密码。缺省值为密钥库密码。

- **`xpack.monitoring.exporters.$NAME.ssl.keystore.secure_key_password`（[安全](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/secure-settings.html)）**

    密钥库中密钥的密码。缺省值为密钥库密码。

- **`xpack.monitoring.exporters.$NAME.ssl.truststore.path`**

    包含要信任的证书的密钥库的路径。它必须是Java密钥库（jks）或PKCS＃12文件。

- **`xpack.monitoring.exporters.$NAME.ssl.truststore.password`**

    信任库的密码。

- **`xpack.monitoring.exporters.$NAME.ssl.truststore.secure_password`（[安全](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/secure-settings.html)）**

    信任库的密码。

#### PKCS＃12文件

可以将Elasticsearch配置为使用包含私有密钥，证书和应受信任的证书的PKCS＃12容器文件（`.p12`或多个`.pfx`文件）。

PKCS＃12文件的配置方式与Java密钥库文件相同：

- **`xpack.monitoring.exporters.$NAME.ssl.keystore.path`**

    包含私钥和证书的密钥库文件的路径。

- **`xpack.monitoring.exporters.$NAME.ssl.keystore.type`**

    密钥库文件的格式。它必须是`jks`或`PKCS12`。如果密钥库路径以“ .p12”，“。pfx”或“ .pkcs12”结尾，则此设置默认为`PKCS12`。否则，默认为`jks`。

- **`xpack.monitoring.exporters.$NAME.ssl.keystore.password`**

    密钥库的密码。

- **`xpack.monitoring.exporters.$NAME.ssl.keystore.secure_password`（[安全](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/secure-settings.html)）**

    密钥库的密码。

- **`xpack.monitoring.exporters.$NAME.ssl.keystore.key_password`**

    密钥库中密钥的密码。缺省值为密钥库密码。

- **`xpack.monitoring.exporters.$NAME.ssl.keystore.secure_key_password`（[安全](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/secure-settings.html)）**

    密钥库中密钥的密码。缺省值为密钥库密码。

- **`xpack.monitoring.exporters.$NAME.ssl.truststore.path`**

    包含要信任的证书的密钥库的路径。它必须是Java密钥库（jks）或PKCS＃12文件。

- **`xpack.monitoring.exporters.$NAME.ssl.truststore.type`**

    进行设置`PKCS12`以指示信任库是PKCS＃12文件。

- **`xpack.monitoring.exporters.$NAME.ssl.truststore.password`**

    信任库的密码。

- **`xpack.monitoring.exporters.$NAME.ssl.truststore.secure_password`（[安全](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/secure-settings.html)）**

    信任库的密码。

#### PKCS＃11令牌

可以将Elasticsearch配置为使用PKCS＃11令牌，该令牌包含私钥，证书和应信任的证书。

PKCS＃11令牌需要在JVM级别进行其他配置，并且可以通过以下设置启用：

- **`xpack.monitoring.exporters.$NAME.keystore.type`**

    进行设置`PKCS11`以指示应将PKCS＃11令牌用作密钥库。

- **`xpack.monitoring.exporters.$NAME.truststore.type`**

    信任库文件的格式。对于Java密钥库格式，请使用`jks`。对于PKCS＃12文件，使用`PKCS12`。对于PKCS＃11令牌，请使用`PKCS11`。默认值为 `jks`。

在配置将JVM配置为用作Elasticsearch的密钥库或信任库的PKCS＃11令牌时，可以通过将适当的值设置为`ssl.truststore.password` 或`ssl.truststore.secure_password`在您配置的上下文中配置令牌的PIN 。由于只能配置一个PKCS＃11令牌，因此只有一个密钥库和信任库可用于在Elasticsearch中进行配置。反过来，这意味着在传输层和http层中，只能将一个证书用于TLS。