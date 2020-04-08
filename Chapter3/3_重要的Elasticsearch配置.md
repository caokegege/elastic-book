[toc]

## 重要Elasticsearch配置

尽管Elasticsearch需要很少的配置，但是在投入生产之前，需要考虑许多设置。

进入生产之前，**必须**考虑以下设置：

- [路径设定](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/path-settings.html)
- [集群名称](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/cluster.name.html)
- [节点名称](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/node.name.html)
- [网络主机](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/network.host.html)
- [发现设置](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/discovery-settings.html)
- [堆大小](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/heap-size.html)
- [堆转储路径](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/heap-dump-path.html)
- [GC记录](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/gc-logging.html)
- [临时目录](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/es-tmpdir.html)

## path.data和path.logs

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

## cluster.name

当`cluster.name`节点与集群中的所有其他节点共享节点时，该节点只能加入集群。默认名称为`elasticsearch`，但您应将其更改为描述群集用途的适当名称。

```yaml
cluster.name: logging-prod
```

确保不要在不同的环境中重复使用相同的集群名称，否则最终可能会导致节点加入错误的集群。

## node.name

Elasticsearch `node.name`用作Elasticsearch 特定实例的人类可读标识符，因此它包含在许多API的响应中。它默认为计算机在Elasticsearch启动时具有的主机名，但可以`elasticsearch.yml`按以下方式显式配置 ：

```yaml
node.name: prod-data-2
```

## network.host

默认情况下，Elasticsearch仅绑定到环回地址（例如`127.0.0.1` 和）`[::1]`。这足以在服务器上运行单个开发节点。

> 实际上，可以从`$ES_HOME` 单个节点上的相同位置启动多个节点。这对于测试Elasticsearch形成集群的能力很有用，但不是建议用于生产的配置。

为了与其他服务器上的节点形成集群，您的节点将需要绑定到非环回地址。尽管[网络设置](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/modules-network.html)很多 ，通常您需要配置的是 `network.host`：

```yaml
network.host: 192.168.1.10
```



该`network.host`设置也了解一些特殊的值，比如 `_local_`，`_site_`，`_global_`和喜欢修饰`:ip4`和`:ip6`，其中的细节中可以找到[特殊值`network.host`](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/modules-network.html#network-interface-values)。

> 一旦为提供了自定义设置`network.host`，Elasticsearch就会假设您正在从开发模式转换为生产模式，并将许多系统启动检查从警告升级为异常。有关更多信息，请参见[开发模式与生产模式](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/system-config.html#dev-vs-prod)。

## 重要的发现和集群的形成设置

在投入生产之前，应该配置两个重要的发现和集群形成设置，以便集群中的节点可以彼此发现并选举一个主节点。

#### discovery.seed_hosts

开箱即用，无需任何网络配置，Elasticsearch将绑定到可用的环回地址，并将扫描本地端口9300至9305，以尝试连接到同一服务器上运行的其他节点。这无需任何配置即可提供自动群集体验。

如果要与其他主机上的节点组成集群，则应使用此`discovery.seed_hosts`设置提供集群中其他主机 的列表，这些主机符合主机要求并且可能处于活动状态且可联系，以便为[发现过程](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/modules-discovery-hosts-providers.html)提供种子。此设置应该是群集中所有符合主机资格的节点的地址的列表。每个地址可以是IP地址，也可以是通过DNS解析为一个或多个IP地址的主机名。

如果符合主机资格的节点没有固定的名称或地址，请使用 [备用主机提供程序](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/modules-discovery-hosts-providers.html#built-in-hosts-providers)动态查找其地址。

#### cluster.initial_master_nodes

首次启动全新的Elasticsearch群集时，会出现一个[群集引导](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/modules-discovery-bootstrap-cluster.html)步骤，该步骤确定了在第一次选举中便对其票数进行计数的主资格节点的集合。在[开发模式下](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/bootstrap-checks.html#dev-vs-prod-mode)（未配置发现设置），此步骤由节点自己自动执行。由于这种自动引导从[本质上来说](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/modules-discovery-quorums.html)是[不安全的](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/modules-discovery-quorums.html)，因此在[生产模式下](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/bootstrap-checks.html#dev-vs-prod-mode)启动全新群集时，必须明确列出符合资格的主机节点，并在第一次选举中对其选票进行计数。使用设置来`cluster.initial_master_nodes`设置此列表 。重新启动集群或将新节点添加到现有集群时，不应使用此设置。

```yaml
discovery.seed_hosts:
   - 192.168.1.10:9300
   - 192.168.1.11 
   - seeds.mydomain.com 
   - [0:0:0:0:0:ffff:c0a8:10c]:9301 
cluster.initial_master_nodes: 
   - master-node-a
   - master-node-b
   - master-node-c
```

1. 端口是可选的，通常默认为`9300`，但是某些设置可以[覆盖](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/modules-discovery-hosts-providers.html#built-in-hosts-providers)此默认值。
2. 如果主机名解析为多个IP地址，则该节点将尝试在所有解析的地址处发现其他节点。
3. IPv6地址必须放在方括号中。
4. 初始主节点应通过其标识 [`node.name`](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/node.name.html)，默认为其主机名。确保中的值完全`cluster.initial_master_nodes`匹配`node.name`。如果您使用完全限定的域名（例如，将其 `master-node-a.example.com`用作节点名称），则必须在此列表中使用完全限定的名称。相反，如果`node.name`是裸机主机名而没有任何尾随限定符，则还必须省略中的尾随限定符`cluster.initial_master_nodes`。

有关更多信息，请参阅[引导集群](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/modules-discovery-bootstrap-cluster.html)以及 [发现和集群形成设置](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/modules-discovery-settings.html)。

## 设置堆大小

默认情况下，Elasticsearch告诉JVM使用最小和最大大小为1 GB的堆。在进入生产阶段时，配置堆大小以确保Elasticsearch有足够的可用堆非常重要。

Elasticsearch将通过（最小堆大小）和（最大堆大小）设置分配[jvm.options中](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/jvm-options.html)指定的整个堆 。您应该将这两个设置设置为彼此相等。`Xms` `Xmx`

这些设置的值取决于服务器上可用的RAM数量：

- 设置`Xmx`和`Xms`你的物理内存不超过50％。Elasticsearch出于JVM堆以外的目的需要内存，因此为此留出空间很重要。例如，Elasticsearch使用堆外缓冲区来进行有效的网络通信，依靠操作系统的文件系统缓存来有效地访问文件，并且JVM本身也需要一些内存。观察Elasticsearch过程使用的内存多于该`Xmx`设置配置的限制，这是正常的。

- 组`Xmx`和`Xms`不超过所述阈值，对压缩对象的指针的JVM用途（压缩糟糕）; 确切的阈值有所不同，但接近32 GB。您可以通过在日志中查找如下一行来验证您是否处于阈值以下：

    ```
    heap size [1.9gb], compressed ordinary obje
    ```

- 理想地设置`Xmx`并`Xms`以不超过该阈值对于基于零的压缩糟糕; 确切的阈值有所不同，但是在大多数系统上26 GB是安全的，但是在某些系统上可以达到30 GB。您可以通过使用JVM选项启动Elasticsearch `-XX:+UnlockDiagnosticVMOptions -XX:+PrintCompressedOopsMode`并查找类似于以下内容的行来验证您是否处于此阈值 以下：

    ```
    heap address: 0x000000011be00000, size: 27648 MB, zero based Compressed Oops
    ```

    显示启用了从零开始的压缩oop。如果未启用从零开始的压缩oop，那么您将看到类似于以下内容的行：

    ```
    heap address: 0x0000000118400000, size: 28672 MB, Compressed Oops with base: 0x00000001183ff000
    ```

Elasticsearch可用的堆越多，它可用于其内部缓存的内存就越多，但可供操作系统用于文件系统缓存的内存就越少。同样，较大的堆可能导致较长的垃圾回收暂停。

这是如何通过`jvm.options.d/`文件设置堆大小的示例：

```txt
- Xms2g 
- Xmx2g 
```

也可以通过环境变量设置堆大小。可以通过以下方法设置这些值`ES_JAVA_OPTS`：

```sh
ES_JAVA_OPTS="-Xms2g -Xmx2g" ./bin/elasticsearch 
ES_JAVA_OPTS="-Xms4000m -Xmx4000m" ./bin/elasticsearch 
```

[Windows服务](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/zip-windows.html#windows-service)的堆配置与上述不同。可以为Windows服务初始填充的值可以如上所述配置，但在安装服务后会有所不同。有关其他详细信息，请查阅[Windows服务文档](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/zip-windows.html#windows-service)。

## JVM堆转储路径

默认情况下，Elasticsearch配置JVM在内存不足或异常的时候dump到默认数据目录中（这是 `/var/lib/elasticsearch`为[RPM](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/rpm.html)和[Debian](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/deb.html)的包分布，并且`data`所述Elasticsearch安装在根目录下的[tar](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/targz.html)和[zip](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/zip-windows.html)归档文件发行） 。如果这个路径是不适合接受堆转储，您应该修改的条目`-XX:HeapDumpPath=...`在 [`jvm.options`](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/jvm-options.html)。如果指定目录，那么JVM将基于正在运行的实例的PID为堆转储生成文件名。如果指定固定文件名而不是目录，则当JVM需要在内存不足异常时执行堆转储时，该文件必须不存在，否则堆转储将失败。

## GC日志

默认情况下，Elasticsearch启用GC日志。它们[`jvm.options`](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/jvm-options.html)在Elasticsearch日志中配置 并输出到相同的默认位置。默认配置每64 MB轮换一次日志，最多可消耗2 GB磁盘空间。

您可以使用[JEP 158：Unified JVM Logging中](https://openjdk.java.net/jeps/158)描述的命令行选项来重新配置JVM日志 [记录](https://openjdk.java.net/jeps/158)。除非您`jvm.options`直接更改默认文件，否则除了您自己的设置外，还将应用Elasticsearch默认配置。要禁用默认配置，请首先通过提供`-Xlog:disable`选项来禁用日志记录 ，然后提供您自己的命令行选项。这将禁用*所有* JVM日志记录，因此请确保检查可用选项并启用所需的所有功能。

要查看原始JEP中未包含的其他选项，请参阅 [使用JVM Unified Logging Framework启用日志记录](https://docs.oracle.com/en/java/javase/13/docs/specs/man/java.html#enable-logging-with-the-jvm-unified-logging-framework)。

### 范例

- 通过创建`$ES_HOME/config/jvm.options.d/gc.options`一些示例选项，将默认GC日志输出位置更改为`/opt/my-app/gc.log`：

    ```shell
    # Turn off all previous logging configuratons
    -Xlog:disable
    
    # Default settings from JEP 158, but with `utctime` instead of `uptime` to match the next line
    -Xlog:all=warning:stderr:utctime,level,tags
    
    # Enable GC logging to a custom location with a variety of options
    -Xlog:gc*,gc+age=trace,safepoint:file=/opt/my-app/gc.log:utctime,pid,tags:filecount=32,filesize=64m
    ```

- 配置Elasticsearch [Docker容器](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/docker.html)以将GC调试日志发送到标准错误（`stderr`）。这使容器协调器可以处理输出。如果使用`ES_JAVA_OPTS`环境变量，请指定：

    ```sh
    MY_OPTS="-Xlog:disable -Xlog:all=warning:stderr:utctime,level,tags -Xlog:gc=debug:stderr:utctime"
    docker run -e ES_JAVA_OPTS="$MY_OPTS" # etc
    ```

