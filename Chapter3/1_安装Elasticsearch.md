## 安装Elasticsearch

### 托管的Elasticsearch

您可以在自己的硬件上运行Elasticsearch，或者 在Elastic Cloud上使用我们 [托管的Elasticsearch Service](https://www.elastic.co/cloud/elasticsearch-service)。Elasticsearch Service在AWS和GCP上均可用。 [免费试用Elasticsearch Service](https://www.elastic.co/cloud/elasticsearch-service/signup?baymax=docs-body&elektra=docs)。

### 自己安装Elasticsearch

Elasticsearch以下列软件包格式提供：

| Linux and MacOS `tar.gz` archives | 这些`tar.gz`归档文件可用于在任何Linux发行版和MacOS上安装。[从Linux或MacOS上的存档安装Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/targz.html) |
| --------------------------------- | ------------------------------------------------------------ |
| Windows `.zip` archive            | 该`zip`归档文件适合在Windows上安装。[`.zip`在Windows上安装Elasticsearch with](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/zip-windows.html) |
| `deb`                             | 该`deb`软件包适用于Debian，Ubuntu和其他基于Debian的系统。Debian软件包可以从Elasticsearch网站或我们的Debian存储库下载。[使用Debian软件包安装Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/deb.html) |
| `rpm`                             | 该`rpm`软件包适合在Red Hat，Centos，SLES，OpenSuSE和其他基于RPM的系统上安装。RPM可以从Elasticsearch网站或我们的RPM存储库下载。[使用RPM安装Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/rpm.html) |
| `msi`                             | 此功能处于beta中，可能会更改。该设计和代码不如正式的GA功能成熟，并且按原样提供，不提供任何担保。Beta功能不受官方GA功能的支持SLA约束。该`msi`软件包适合在至少安装了.NET 4.5框架的Windows 64位系统上安装，并且是在Windows上开始使用Elasticsearch的最简单选择。MSI可以从Elasticsearch网站下载。[使用Windows MSI安装程序安装Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/windows.html) |
| `docker`                          | 图像可用于将Elasticsearch作为Docker容器运行。它们可以从Elastic Docker Registry下载。[使用Docker安装Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/docker.html) |
| `brew`                            | 可以从Elastic Homebrew点击获取公式，以便使用Homebrew软件包管理器在macOS上安装Elasticsearch。[使用Homebrew在macOS上安装Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/brew.html) |

### 配置管理工具

我们还提供以下配置管理工具来帮助进行大型部署：

| Puppet  | [puppet-elasticsearch](https://github.com/elastic/puppet-elasticsearch) |
| ------- | ------------------------------------------------------------ |
| Chef    | [cookbook-elasticsearch](https://github.com/elastic/cookbook-elasticsearch) |
| Ansible | [ansible-elasticsearch](https://github.com/elastic/ansible-elasticsearch) |

## 从Linux或MacOS的存档安装Elasticsearch

Elasticsearch是`.tar.gz`Linux和MacOS 的存档。

根据弹性许可，可以免费使用此软件包。它包含开源和免费的商业功能，以及对付费商业功能的访问。 [开始30天试用，](https://www.elastic.co/guide/en/elastic-stack-overview/7.x/license-management.html)以试用所有付费商业功能。有关弹性许可级别的信息，请参阅[Subscriptions](https://www.elastic.co/subscriptions) 页面。

最新的稳定版本的Elasticsearch可在[Download Elasticsearch](https://www.elastic.co/downloads/elasticsearch)页面上找到 。其他版本可以在 [Past Releases page](https://www.elastic.co/downloads/past-releases)上找到 。

> Elasticsearch包含 来自JDK维护者（GPLv2 + CE）的[OpenJDK](http://openjdk.java.net/)捆绑版。要使用自己的Java版本，请参阅[JVM版本要求.](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/setup.html#jvm-version)

### 下载并安装用于Linux的存档

Elasticsearch的7.8.0版本尚未发布。

### 下载并安装MacOS的存档

Elasticsearch的7.8.0版本尚未发布。

### 启用自动创建系统索引

一些商业功能会在Elasticsearch中自动创建系统索引。默认情况下，Elasticsearch配置为允许自动创建索引，并且不需要其他步骤。但是，如果你有Elasticsearch禁用自动创建索引，您必须配置 [`action.auto_create_index`](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/docs-index_.html#index-creation)在`elasticsearch.yml`允许商业功能创建以下指标：

```yaml
action.auto_create_index: .monitoring*,.watches,.triggered_watches,.watcher-history*,.ml*
```

> 如果使用[Logstash](https://www.elastic.co/products/logstash) 或[Beats，](https://www.elastic.co/products/beats)则很可能需要在`action.auto_create_index`设置中使用其他索引名称，而确切的值将取决于本地配置。如果不确定环境的正确值，则可以考虑将值设置为 `*`允许自动创建所有索引的值。

### 在命令行中运行Elasticsearch

可以从命令行启动Elasticsearch，如下所示：

```sh
./bin/elasticsearch
```

如果您已经用密码保护了Elasticsearch密钥库，那么将提示您输入密钥库的密码。有关更多详细信息，请参见[安全设置](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/secure-settings.html)。

默认情况下，Elasticsearch在前台运行，将其日志打印到标准输出（`stdout`），可以通过按停止`Ctrl-C`。

> 与Elasticsearch打包在一起的所有脚本都需要支持阵列的Bash版本，并假定Bash在以下位置可用`/bin/bash`。因此，Bash应该直接或通过符号链接在此路径上可用。

### 检查Elasticsearch运行

您可以测试你的Elasticsearch节点通过发送一个HTTP请求的端口上运行`9200`上`localhost`：

```console
GET / 
```

应该会给您这样的答复：

```js
{
  "name" : "Cp8oag6",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "AT69_T_DTp-1qgIJlatQqA",
  "version" : {
    "number" : "7.8.0-SNAPSHOT",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "f27399d",
    "build_date" : "2016-03-30T09:51:41.449Z",
    "build_snapshot" : false,
    "lucene_version" : "8.5.0",
    "minimum_wire_compatibility_version" : "1.2.3",
    "minimum_index_compatibility_version" : "1.2.3"
  },
  "tagline" : "You Know, for Search"
}
```

`stdout`可以使用 命令行上的`-q`或`--quiet`选项禁用日志打印到。

### 作为守护程序运行

要将Elasticsearch作为守护程序运行，请`-d`在命令行上指定，然后使用以下`-p`选项将进程ID记录在文件中：

```sh
./bin/elasticsearch -d -p pid
```

如果您已经用密码保护了Elasticsearch密钥库，那么将提示您输入密钥库的密码。有关更多详细信息，请参见[安全设置](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/secure-settings.html)。

日志消息可以在`$ES_HOME/logs/`目录中找到。

要关闭Elasticsearch，请终止`pid`文件中记录的进程ID ：

```sh
pkill -F pid
```

> [RPM](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/rpm.html)和[Debian](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/deb.html) 软件包中提供的启动脚本会为您启动和停止Elasticsearch进程。

### 在命令行配置Elasticsearch

`$ES_HOME/config/elasticsearch.yml` 默认情况下，Elasticsearch从文件中加载其配置。该配置文件的格式在[*配置Elasticsearch*](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/settings.html)中进行了说明 。

可以在命令行上使用以下`-E`语法在配置文件中指定的任何设置：

```sh
./bin/elasticsearch -d -Ecluster.name=my_cluster -Enode.name=node_1
```

> 通常，`cluster.name`应将任何群集范围的设置（如）添加到`elasticsearch.yml`配置文件中，而任何特定于节点的设置（例如`node.name`可以在命令行上指定）。

### 文档目录列表

存档分发完全是独立的。默认情况下，所有文件和目录都包含在`$ES_HOME` 解压缩归档文件时创建的目录中。

这非常方便，因为您无需创建任何目录即可开始使用Elasticsearch，并且卸载Elasticsearch就像删除`$ES_HOME`目录一样容易。但是，建议更改配置目录，数据目录和日志目录的默认位置，以便以后不再删除重要数据。

| 类型        | 描述                                                         | 默认位置               | 设置           |
| ----------- | ------------------------------------------------------------ | ---------------------- | -------------- |
| **home**    | Elasticsearch主目录或 `$ES_HOME`                             | 通过解压缩存档创建目录 |                |
| **bin**     | 二进制脚本，包括`elasticsearch`启动节点和`elasticsearch-plugin`安装插件 | `$ES_HOME/bin`         |                |
| **conf**    | 配置文件包括 `elasticsearch.yml`                             | `$ES_HOME/config`      | `ES_PATH_CONF` |
| **data**    | 节点上分配的每个索引/分片的数据文件的位置。可以容纳多个位置。 | `$ES_HOME/data`        | `path.data`    |
| **logs**    | 日志文件位置。                                               | `$ES_HOME/logs`        | `path.logs`    |
| **plugins** | 插件文件位置。每个插件将包含在一个子目录中。                 | `$ES_HOME/plugins`     |                |
| **repo**    | 共享文件系统存储库位置。可以容纳多个位置。可以将文件系统存储库放置在此处指定的任何目录的任何子目录中。 | 未配置                 | `path.repo`    |

### 接下来的步骤

现在，您已经建立了一个测试Elasticsearch环境。在开始进行认真的开发或使用Elasticsearch投入生产之前，您必须进行一些附加设置：

- 了解如何[配置Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/settings.html)。
- 配置[重要的Elasticsearch设置](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/important-settings.html)。
- 配置[重要的系统设置](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/system-config.html)。