# 设置Elasticsearch

本部分包括有关如何设置Elasticsearch并使它运行的信息，包括：

- Downloading
- Installing
- Starting
- Configuring

## 支持的平台

可在此处获得官方支持的操作系统和JVM的列表： [Support Matirx](https://www.elastic.co/support/matrix)。Elasticsearch在列出的平台上进行了测试，但有可能也可以在其他平台上使用。

## Java（JVM）版本

Elasticsearch是使用Java构建的，并且在每个发行版中都包含来自JDK维护者（GPLv2 + CE）的捆绑版本的 [OpenJDK](http://openjdk.java.net/)。捆绑的JVM是推荐的JVM，位于`jdk`Elasticsearch主目录的目录内。

要使用自己的Java版本，请设置`JAVA_HOME`环境变量。如果必须使用与捆绑的JVM不同的Java版本，建议使用[受支持](https://www.elastic.co/support/matrix) [的Java LTS版本](http://www.oracle.com/technetwork/java/eol-135779.html)。如果使用已知的Java错误版本，Elasticsearch将拒绝启动。使用您自己的JVM时，可以删除捆绑的JVM目录。