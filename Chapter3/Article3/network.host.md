## `network.host`

默认情况下，Elasticsearch仅绑定到环回地址（例如`127.0.0.1` 和）`[::1]`。这足以在服务器上运行单个开发节点。

> 实际上，可以从`$ES_HOME` 单个节点上的相同位置启动多个节点。这对于测试Elasticsearch形成集群的能力很有用，但不是建议用于生产的配置。

为了与其他服务器上的节点形成集群，您的节点将需要绑定到非环回地址。尽管[网络设置](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/modules-network.html)很多 ，通常您需要配置的是 `network.host`：

```yaml
network.host: 192.168.1.10
```



该`network.host`设置也了解一些特殊的值，比如 `_local_`，`_site_`，`_global_`和喜欢修饰`:ip4`和`:ip6`，其中的细节中可以找到[特殊值`network.host`](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/modules-network.html#network-interface-values)。

> 一旦为提供了自定义设置`network.host`，Elasticsearch就会假设您正在从开发模式转换为生产模式，并将许多系统启动检查从警告升级为异常。有关更多信息，请参见[开发模式与生产模式](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/system-config.html#dev-vs-prod)。