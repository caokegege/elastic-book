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