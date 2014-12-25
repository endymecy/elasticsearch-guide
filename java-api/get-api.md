# 获取API

获取API允许你通过id从索引中获取类型化的JSON文档，如下例：

```java
GetResponse response = client.prepareGet("twitter", "tweet", "1")
        .execute()
        .actionGet();
```

## 操作线程

The get API allows to set the threading model the operation will be performed when the actual execution of the API is
performed on the same node (the API is executed on a shard that is allocated on the same server).

默认情况下，`operationThreaded`设置为true表示操作执行在不同的线程上面。下面是一个设置为false的例子。

```java
GetResponse response = client.prepareGet("twitter", "tweet", "1")
        .setOperationThreaded(false)
        .execute()
        .actionGet();
```