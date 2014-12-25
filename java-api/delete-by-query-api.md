# 基于查询的删除API

基于查询的删除API允许开发者基于查询删除一个或者多个索引、一个或者多个类型。下面是一个例子。

```java
import static org.elasticsearch.index.query.FilterBuilders.*;
import static org.elasticsearch.index.query.QueryBuilders.*;

DeleteByQueryResponse response = client.prepareDeleteByQuery("test")
        .setQuery(termQuery("_type", "type1"))
        .execute()
        .actionGet();
```