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

