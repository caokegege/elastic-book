## 启动Elasticsearch

启动Elasticsearch的方法因您的安装方式而异。

### 存档包（`.tar.gz`）

如果您使用`.tar.gz`软件包安装了Elasticsearch ，则可以从命令行启动Elasticsearch。

#### 在命令行中运行Elasticsearch

可以从命令行启动Elasticsearch，如下所示：

```sh
./bin/elasticsearch
```

如果您已经用密码保护了Elasticsearch密钥库，那么将提示您输入密钥库的密码。有关更多详细信息，请参见[安全设置](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/secure-settings.html)。

默认情况下，Elasticsearch在前台运行，将其日志打印到标准输出（`stdout`），可以通过按停止`Ctrl-C`。

所有的Elasticsearch打包脚本需要一个支持数组的Bash版本，并假定Bash在以下位置可用`/bin/bash`。因此，Bash应该直接或通过符号链接在此路径上可用。

#### 作为守护程序运行

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

[RPM](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/rpm.html)和[Debian](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/deb.html) 软件包中提供的启动脚本会为您启动和停止Elasticsearch进程。

### Docker镜像

如果安装了Docker映像，则可以从命令行启动Elasticsearch。根据您使用的是开发模式还是生产模式，可以使用不同的方法。请参阅[Docker Run](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/docker-cli-run.html)。

## 停止Elasticsearch

Elasticsearch的有序关闭可确保Elasticsearch有机会清理和关闭未使用的资源。例如，以有序方式关闭的节点将其自身从群集中删除，将跨日志同步到磁盘，并执行其他相关的清理活动。您可以通过适当地停止Elasticsearch来确保有序关闭。

如果您将Elasticsearch作为服务运行，则可以通过安装提供的服务管理功能来停止Elasticsearch。

如果您直接运行Elasticsearch，则可以通过在控制台中运行Elasticsearch的方式发送Control-C来停止Elasticsearch，或者通过发送`SIGTERM`至POSIX系统上的Elasticsearch进程来停止。您可以获取PID以通过各种工具（例如`ps`或`jps`）将信号发送至：

```sh
$ jps | grep Elasticsearch
14542 Elasticsearch
 
```

从Elasticsearch启动日志中：

```sh
[2016-07-07 12:26:18,908][INFO ][node                     ] [I8hydUG] version[5.0.0-alpha4], pid[15399], build[3f5b994/2016-06-27T16:23:46.861Z], OS[Mac OS X/10.11.5/x86_64], JVM[Oracle Corporation/Java HotSpot(TM) 64-Bit Server VM/1.8.0_92/25.92-b14]   
```

或者通过指定在启动时将PID文件写入的位置（`-p `）：

```sh
$ ./bin/elasticsearch -p /tmp/elasticsearch-pid -d
$ cat /tmp/elasticsearch-pid && echo
15516
$ kill -SIGTERM 15516
```



### 停止致命错误

在Elasticsearch虚拟机的生命周期内，可能会出现某些致命错误，使虚拟机处于可疑状态。此类致命错误包括内存不足错误，虚拟机内部错误以及严重的I / O错误。

当Elasticsearch检测到虚拟机遇到此类致命错误时，Elasticsearch将尝试记录该错误，然后停止该虚拟机。当Elasticsearch启动此类关闭时，它不会如上所述进行有序关闭。Elasticsearch流程还将返回一个特殊的状态代码，以指示错误的性质。

| JVM internal error            | JVM内部错误      | 128  |
| ----------------------------- | ---------------- | ---- |
| Out of memory error           | 内存不足错误     | 127  |
| Stack overflow error          | 堆栈溢出错误     | 126  |
| Unknown virtual machine error | 未知的虚拟机错误 | 125  |
| Serious I/O error             | 严重的I / O错误  | 124  |
| Unknown fatal error           | 未知的致命错误   | 1个  |
