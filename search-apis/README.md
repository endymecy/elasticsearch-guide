# 搜索APIS

## 路由

当执行一个搜索，它将会广播到所有的索引/索引分片。被搜索的分片可以通过提供`routing`参数来控制。例如当索引推文时，用户名就可以作为路由的值。

```shell
$ curl -XPOST 'http://localhost:9200/twitter/tweet?routing=kimchy' -d '{
    "user" : "kimchy",
    "postDate" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
'
```

在这种情况下，如果你仅仅想查询特定用户的推文，我们可以将其指定为路由，从而在搜索时只命中相关的分片。

```shell
$ curl -XGET 'http://localhost:9200/twitter/tweet/_search?routing=kimchy' -d '{
    "query": {
        "filtered" : {
            "query" : {
                "query_string" : {
                    "query" : "some query string here"
                }
            },
            "filter" : {
                "term" : { "user" : "kimchy" }
            }
        }
    }
}
'
```

路由参数可以由逗号分隔的多个字符串表示，这将使查询命中与路由参数值相匹配的相关分片。

## 统计组

一个搜索可以与统计组相关联，每一个组都包含一个统计集合，它可以通过索引统计API在以后重新获取。例如，下面是一个与两个分组相关联的搜索的搜索体。

```
{
    "query" : {
        "match_all" : {}
    },
    "stats" : ["group1", "group2"]
}
```

* [搜索](search.md)
* [URI搜索](uri-search.md)
* [请求体(request body)搜索](request-body-search.md)
* [搜索模板](search-template.md)
* [搜索分片API](search-shards-api.md)
* [聚合(aggregations)](aggregations.md)
* [facets](facets.md)
* [启发者(suggesters)](suggesters.md)
* [多搜索API](multi-search-api.md)
* [计数API](count-api.md)
* [搜索存在(search exist)API](search-exits-api.md)
* [验证API](validate-api.md)
* [解释API](explain.md)
* [过滤器(percolator)](percolator.md)
* [more like this api](more-like-this-api.md)