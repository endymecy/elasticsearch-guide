# 计数API

计数API允许开发者简单的执行一个查询，返回和查询条件相匹配的文档的总数。它可以跨多个索引以及跨多个类型执行。

```java
import static org.elasticsearch.index.query.xcontent.FilterBuilders.*;
import static org.elasticsearch.index.query.xcontent.QueryBuilders.*;

CountResponse response = client.prepareCount("test")
        .setQuery(termQuery("_type", "type1"))
        .execute()
        .actionGet();
```

## 操作线程

它有三种线程模式。`NO_THREADS`模式表明操作在当前线程中执行；`SINGLE_THREAD`模式表明操作在一个独立的不同线程中执行，所有的分片共用这个线程；`THREAD_PER_SHARD`模式表明操作在一个独立的不同
线程中执行，并且一个分片一个线程。默认的模式是`SINGLE_THREAD`。