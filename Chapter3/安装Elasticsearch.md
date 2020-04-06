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