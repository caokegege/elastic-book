## 启动Elasticsearch并运行

要将Elasticsearch用作测试驱动器，您可以 在Elasticsearch Service上创建 [托管部署](https://www.elastic.co/cloud/elasticsearch-service/signup?baymax=docs-body&elektra=docs)，或在您自己的Linux，macOS或Windows计算机上设置多节点Elasticsearch集群。

### 在Elastic Cloud上运行Elasticsearch

在Elasticsearch Service上创建部署时，该服务与Kibana和APM一起预配一个三节点Elasticsearch集群。

要创建部署：

1. 注册[免费试用版，](https://www.elastic.co/cloud/elasticsearch-service/signup?baymax=docs-body&elektra=docs) 然后验证您的电子邮件地址。
2. 为您的帐户设置密码。
3. 单击**创建部署**。

创建部署后，就可[*以为一些文档*](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/getting-started-index.html)建立[*索引了*](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/getting-started-index.html)。

### 运行Elasticsearch本地的Linux，MacOS或Windows

在Elasticsearch Service上创建部署时，将自动设置一个主节点和两个数据节点。通过从tar或zip存档安装，您可以在本地启动Elasticsearch的多个实例，以查看多节点集群的行为。

要在本地运行三节点的Elasticsearch集群：

1. 下载适用于您的操作系统的Elasticsearch存档：

    Linux：[elasticsearch-7.8.0-linux-x86_64.tar.gz](https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.8.0-linux-x86_64.tar.gz)

    ```sh
    curl -L -O https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.8.0-linux-x86_64.tar.gz
    ```

    macOS：[elasticsearch-7.8.0-darwin-x86_64.tar.gz](https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.8.0-darwin-x86_64.tar.gz)

    ```sh
    curl -L -O https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.8.0-darwin-x86_64.tar.gz
    ```

    Windows： [elasticsearch-7.8.0-windows-x86_64.zip](https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.8.0-windows-x86_64.zip)

2. 提取档案：

    Linux：

    ```sh
    tar -xvf elasticsearch-7.8.0-linux-x86_64.tar.gz
    ```

    苹果系统：

    ```sh
    tar -xvf elasticsearch-7.8.0-darwin-x86_64.tar.gz
    ```

    Windows PowerShell：

    ```powershell
    Expand-Archive elasticsearch-7.8.0-windows-x86_64.zip
    ```

3. 从`bin`目录启动Elasticsearch ：

    Linux和macOS：

    ```sh
    cd elasticsearch-7.8.0/bin
    ./elasticsearch
    ```

    Windows：

    ```powershell
    cd elasticsearch-7.8.0\bin
    .\elasticsearch.bat
    ```

    现在，您已经建立并运行了一个单节点Elasticsearch集群！

4. 启动另外两个Elasticsearch实例，以便您可以看到典型的多节点集群的行为。您需要为每个节点指定唯一的数据和日志路径。

    Linux和macOS：

    ```sh
    ./elasticsearch -Epath.data=data2 -Epath.logs=log2
    ./elasticsearch -Epath.data=data3 -Epath.logs=log3
    ```

    Windows：

    ```powershell
    .\elasticsearch.bat -E path.data=data2 -E path.logs=log2
    .\elasticsearch.bat -E path.data=data3 -E path.logs=log3
    ```

    为其他节点分配了唯一的ID。因为您在本地运行所有三个节点，所以它们会自动与第一个节点一起加入群集。

5. 使用cat health API验证三节点集群是否正在运行。cat API以比原始JSON更易于阅读的格式返回有关集群和索引的信息。

    您可以通过向Elasticsearch REST API提交HTTP请求来直接与集群交互。如果已安装并正在运行Kibana，则也可以打开Kibana并通过开发控制台提交请求。

    > 准备在您自己的应用程序中开始使用Elasticsearch时，您将需要签出 [Elasticsearch语言客户端](https://www.elastic.co/guide/en/elasticsearch/client/index.html)。

    ```console
    GET /_cat/health?v
    
    ```

    该响应应表明`elasticsearch`集群的状态为，`green`并且它具有三个节点：

    ```txt
    epoch      timestamp cluster       status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
    1565052807 00:53:27  elasticsearch green           3         3      6   3    0    0        0             0                  -                100.0%                                                                                
    ```

> 如果您仅运行单个Elasticsearch实例，则集群状态将保持黄色。单节点群集具有完整的功能，但是无法将数据复制到另一个节点以提供弹性。副本分片必须可用，群集状态为绿色。如果群集状态为红色，则某些数据不可用。

### 谈起Elasticsearch与卷曲的命令[编辑](https://github.com/elastic/elasticsearch/edit/7.x/docs/reference/getting-started.asciidoc)

本指南中的大多数示例使您能够复制适当的cURL命令并将请求从命令行提交到本地Elasticsearch实例。

对Elasticsearch的请求包含与任何HTTP请求相同的部分：

```sh
curl -X<VERB> '<PROTOCOL>://<HOST>:<PORT>/<PATH>?<QUERY_STRING>' -d '<BODY>'
```

本示例使用以下变量：

- **<VERB>**

    适当的HTTP方法或动词。例如，`GET`，`POST`， `PUT`，`HEAD`，或`DELETE`。

- **<PROTOCOL>**

    无论是`http`或`https`。如果您在Elasticsearch前面有一个HTTPS代理，或者您使用Elasticsearch安全功能来加密HTTP通信，请使用后者。

- **<HOST>**

    Elasticsearch集群中任何节点的主机名。或者， `localhost`用于本地计算机上的节点。

- **<PORT>**

    运行Elasticsearch HTTP服务的端口，默认为`9200`。

- **<PATH>**

    API端点，可以包含多个组件，例如 `_cluster/stats`或`_nodes/stats/jvm`。

- **<QUERY_STRING>**

    任何可选的查询字符串参数。例如，`?pretty` 将*漂亮地打印* JSON响应以使其更易于阅读。

- **<BODY>**

    JSON编码的请求正文（如有必要）。

如果启用了Elasticsearch安全功能，则还必须提供有权运行API的有效用户名（和密码）。例如，使用 `-u`或`--u`cURL命令参数。有关运行每个API所需的安全特权的详细信息，请参阅[REST API](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/rest-apis.html)。

Elasticsearch使用HTTP状态代码（例如）响应每个API请求`200 OK`。除`HEAD`请求外，它还返回JSON编码的响应正文。

### 其他安装选项

从存档文件安装Elasticsearch使您能够轻松地在本地安装和运行多个实例，以便您可以尝试一下。要运行一个实例，您可以在Docker容器中运行Elasticsearch，在Linux上使用DEB或RPM软件包安装Elasticsearch，在macOS上使用Homebrew进行安装，或者在Windows上使用MSI软件包安装程序进行安装。有关更多信息，请参见[*安装Elasticsearch*](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/install-elasticsearch.html)。