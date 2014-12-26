# 请求体搜索

有搜索DSL的搜索请求可以被执行。这些DSL包含在请求的请求体中。

```shell
$ curl -XGET 'http://localhost:9200/twitter/tweet/_search' -d '{
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
'
```

## 参数

Name | Description
--- | ---
`timeout` | 默认没有timeout
`from` | 默认是0
`size` | 默认是10
`search_type` | 搜索操作执行的类型，有`dfs_query_then_fetch`, `dfs_query_and_fetch`, `query_then_fetch`, `query_and_fetch`, `count`, `scan`几种，默认是`query_then_fetch`
`query_cache` | 当`?search_type=count`时，查询结果是否缓存
`terminate_after` | The maximum number of documents to collect for each shard, upon reaching which the query execution will terminate early. If set, the response will have a boolean field terminated_early to indicate whether the query execution has actually terminated_early. Defaults to no terminate_after.

`search_type`和`query_cache`必须通过查询参数字符串传递。

`HTTP GET`和`HTTP POST`都可以用来执行带有请求体的搜索。