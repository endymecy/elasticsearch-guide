# uri搜索

一个搜索可以用纯粹的uri来执行查询。在这种模式下使用搜索，并不是所有的选项都是暴露的。它可以方便快速进行`curl 测试`。

```shell
$ curl -XGET 'http://localhost:9200/twitter/tweet/_search?q=user:kimchy'
```

## 参数

Name | Description
--- | ---
`q` | 表示查询字符串
`df` | 在查询中，当没有定义字段的前缀的情况下的默认字段前缀
`analyzer` | 当分析查询字符串时，分析器的名字
`default_operator` | 被用到的默认操作，有`AND`和`OR`两种，默认是`OR`
`explain` | 对于每一个命中(hit)，对怎样得到命中得分的计算给出一个解释
`_source` | 将其设置为false，查询就会放弃检索`_source`字段。你也可以通过设置`_source_include`和`_source_exclude`检索部分文档
`fields` | 命中的文档返回的字段
`sort` | 排序执行。可以以`fieldName`、`fieldName:asc`或者`fieldName:desc`的格式设置。`fieldName`既可以是存在的字段，也可以是`_score`字段。可以有多个sort参数
`track_scores` | 当排序的时候，将其设置为true，可以返回相关度得分
`timeout` | 默认没有timeout
`from` | 默认是0
`size` | 默认是10
`search_type` | 搜索操作执行的类型，有`dfs_query_then_fetch`, `dfs_query_and_fetch`, `query_then_fetch`, `query_and_fetch`, `count`, `scan`几种，默认是`query_then_fetch`
`lowercase_expanded_terms` | terms是否自动小写，默认是true
`analyze_wildcard` | 是否分配通配符和前缀查询，默认是false
`terminate_after` | The maximum number of documents to collect for each shard, upon reaching which the query execution will terminate early. If set, the response will have a boolean field terminated_early to indicate whether the query execution has actually terminated_early. Defaults to no terminate_after.
