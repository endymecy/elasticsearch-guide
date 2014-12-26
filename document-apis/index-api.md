# 索引API

索引API在一个特定的索引中添加或者更新类型化的JSON文档，使它可以被查询。下面的例子插入一个文档到名为"twitter"的索引中

```shell
$ curl -XPUT 'http://localhost:9200/twitter/tweet/1' -d '{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}'
```

索引操作的结果是：

```
{
    "_index" : "twitter",
    "_type" : "tweet",
    "_id" : "1",
    "_version" : 1,
    "created" : true
}
```