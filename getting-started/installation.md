# 安装

Elasticsearch需要Java 7。在本文写作的时候，推荐使用Oracle JDK 1.8.0_25 版本。Java的安装，在各个平台上都有差异，所以我们不想在这里涉及太多细节。在你安装Elasticsearch 之前，
你可以通过以下命令来检查你的Java版本（如果有需要，安装或者升级）：

```
java -version
echo $JAVA_HOME
```

一旦我们将 Java 安装完成， 我们就可以下载并安装 Elasticsearch 了。其二进制文件可以从[www.elasticsearch.org/download](http://www.elasticsearch.org/download)这里下载，
你也可以从这里下载以前发布的版本。对于每个版本，你可以在zip、tar、DEB、RPM 类型的包中选择下载。简单起见，我们使用 tar 包。

用下面的命令下载 Elasticsearch 1.4.2 tar包

```
curl -L -O https://download.elasticsearch.org/elasticsearch/elasticsearch/elasticsearch-1.4.2.tar.gz
```

将其解压并进入bin目录，启动你的节点和单节点集群

```
tar -xvf elasticsearch-1.4.2.tar.gz
cd elasticsearch-1.4.2/bin
./elasticsearch
```

我们可以覆盖集群或者节点的名字。我们可以在启动Elasticsearch的时候通过命令行来指定，如下：

```
./elasticsearch --cluster.name my_cluster_name --node.name my_node_name
```

默认情况下，Elasticsearch使用9200来提供对其REST API的访问。如果有必要，这个端口是可以配置的。