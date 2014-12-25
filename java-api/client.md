# 客户端

有多个地方需要使用`Java client`:

- 在存在的集群中执行标准的[index](index-api.md), [get](get-api.md), [delete](delete-api.md)和[search](search-api.md)
- 在集群中执行管理任务
- 当你要运行嵌套在你的应用程序中的Elasticsearch的时候或者当你要运行单元测试或者集合测试的时候，启动所有节点

获得一个Client是非常容易的，最通用的步骤如下所示：

- 创建一个嵌套的节点，充当集群的一个节点
- 从这个嵌套的节点请求一个Client

另外一种方式是创建一个`TransportClient`来连接集群。

重要提示：
    客户端和集群端推荐使用相同的版本，如果版本不同，可能会出现一些不兼容的问题。

## 节点客户端

实例化一个节点的客户端是获得客户端的最简单的方式。这个Client可以执行elasticsearch相关的操作。

```java
import static org.elasticsearch.node.NodeBuilder.*;

// on startup

Node node = nodeBuilder().node();
Client client = node.client();

// on shutdown

node.close();
```

当你启动一个`node`,它就加入了elasticsearch集群。你可以通过简单的设置`cluster.name`或者明确地使用`clusterName`方法拥有不同的集群。

你能够在你项目的`/src/main/resources/elasticsearch.yml`文件中定义`cluster.name`。只要`elasticsearch.yml`在classpath目录下面，你就能够用到它来启动你的节点。

```
cluster.name: yourclustername
```
或者通过java：

```java
Node node = nodeBuilder().clusterName("yourclustername").node();
Client client = node.client();
```
利用Client的好处是，操作可以自动地路由到这些操作被执行的节点，而不需要执行双跳(double hop)。例如，索引操作将会在该操作最终存在的分片上执行。

当你启动了一个节点，最重要的决定是是否它将保有数据。大多数情况下，我们仅仅需要用到clients，而不需要分片分配给它们。这可以通过设置`node.data`为false或者设置
`node.client`为true来简单实现。

```java
import static org.elasticsearch.node.NodeBuilder.*;

// on startup

Node node = nodeBuilder().client(true).node();
Client client = node.client();

// on shutdown

node.close();
```

另外一个通用的用处就是启动node，然后利用Client进行单元/集成测试。在这种情况下，我们应该启动一个“本地”node。这只是启动node的一个简单的设置。注意，"本地"表示运行在本地
JVM上。运行在同一个JVM上的两个本地服务可以彼此发现并组成一个集群。

```java
import static org.elasticsearch.node.NodeBuilder.*;

// on startup

Node node = nodeBuilder().local(true).node();
Client client = node.client();

// on shutdown

node.close();
```

## 传输（transport）客户端

`TransportClient`利用transport模块远程连接一个elasticsearch集群。它并不加入到集群中，只是简单的获得一个或者多个初始化的transport地址，并以轮询的方式与这些地址进行通信。

```java
// on startup
Client client = new TransportClient()
        .addTransportAddress(new InetSocketTransportAddress("host1", 9300))
        .addTransportAddress(new InetSocketTransportAddress("host2", 9300));

// on shutdown
client.close();
```

注意，如果你有一个与`elasticsearch`集群不同的集群，你可以设置机器的名字。

```java
Settings settings = ImmutableSettings.settingsBuilder()
        .put("cluster.name", "myClusterName").build();
Client client =    new TransportClient(settings);
//Add transport addresses and do something with the client...
```

你也可以用`elasticsearch.yml`文件来设置。

这个客户端可以嗅到集群的其它部分，并将它们加入到机器列表。为了开启该功能，设置`client.transport.sniff`为true。

```java
Settings settings = ImmutableSettings.settingsBuilder()
        .put("client.transport.sniff", true).build();
TransportClient client = new TransportClient(settings);
```

其它的transport客户端设置有如下几个：

Parameter | Description
--- | ---
client.transport.ignore_cluster_name | true：忽略连接节点的集群名验证
client.transport.ping_timeout | ping一个节点的响应时间，默认是5s
client.transport.nodes_sampler_interval | sample/ping 节点的时间间隔，默认是5s

