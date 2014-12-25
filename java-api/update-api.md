# 更新API

你能够创建一个`UpdateRequest`,然后将其发送给client。

```java
UpdateRequest updateRequest = new UpdateRequest();
updateRequest.index("index");
updateRequest.type("type");
updateRequest.id("1");
updateRequest.doc(jsonBuilder()
        .startObject()
            .field("gender", "male")
        .endObject());
client.update(updateRequest).get();
```

或者你也可以利用`prepareUpdate`方法

```java
1 client.prepareUpdate("ttl", "doc", "1")
2        .setScript("ctx._source.gender = \"male\""  , ScriptService.ScriptType.INLINE)
3        .get();

5 client.prepareUpdate("ttl", "doc", "1")
6        .setDoc(jsonBuilder()
7            .startObject()
8                .field("gender", "male")
9            .endObject())
10        .get();
```
1-3行用脚本来更新索引，5-10行用doc来更新索引。

当然，java API也支持使用`upsert`。如果文档还不存在，会根据`upsert`内容创建一个新的索引。

```java
IndexRequest indexRequest = new IndexRequest("index", "type", "1")
        .source(jsonBuilder()
            .startObject()
                .field("name", "Joe Smith")
                .field("gender", "male")
            .endObject());
UpdateRequest updateRequest = new UpdateRequest("index", "type", "1")
        .doc(jsonBuilder()
            .startObject()
                .field("gender", "male")
            .endObject())
        .upsert(indexRequest);
client.update(updateRequest).get();
```

如果文档`index/type/1`已经存在，那么在更新操作完成之后，文档为：

```
{
    "name"  : "Joe Dalton",
    "gender": "male"
}
```

否则，文档为：

```
{
    "name" : "Joe Smith",
    "gender": "male"
}
```

