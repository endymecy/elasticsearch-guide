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

## 1 参数

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

## 2 查询

在搜索请求体中的查询元素允许用查询DSL定义一个查询

```
{
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

## 3 from / size

可以用from和size参数对结果进行分页。from表示你想获得的第一个结果的偏移量，size表示你想获得的结果的个数。from默认是0，size默认是10.

```
{
    "from" : 0, "size" : 10,
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

## 4 排序

一个特定的字段运行添加一个或者多个排序。排序定义在字段级别，特定的字段名`_score`是通过得分排序。

```
{
    "sort" : [
        { "post_date" : {"order" : "asc"}},
        "user",
        { "name" : "desc" },
        { "age" : "desc" },
        "_score"
    ],
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

### 4.1 排序值

#### 4.1.1 排序选项

elasticsearch支持通过数组或者多值字段排序。`mode`选项控制获取排序文档的什么数组值来进行排序。`mode`选项有如下几种

- min：挑选最低的值
- max: 挑选最高的值
- sum：挑选所有值的和作为排序的值，仅用于数字
- avg：挑选所有值得平均作为排序的值，仅用于数字

#### 4.1.2 例子

下面的例子按照文档的平均价格的升序进行排列
```shell
curl -XPOST 'localhost:9200/_search' -d '{
   "query" : {
    ...
   },
   "sort" : [
      {"price" : {"order" : "asc", "mode" : "avg"}}
   ]
}'
```

### 4.2 带有嵌套对象的排序

Elasticsearch支持字段中带有嵌套对象的排序，嵌套的字段排序在其它排序选项存在的基础上支持下面的参数

- nested_path：Defines the on what nested object to sort. The actual sort field must be a direct field inside this nested object. The default is to use the most immediate inherited nested object from the sort field.
- nested_filter：A filter the inner objects inside the nested path should match with in order for its field values to be taken into account by sorting. Common case is to repeat the query / filter inside the nested filter or query. By default no nested_filter is active.

#### 4.2.1 例子

在下面的例子中，`offer`是一个嵌套类型的字段。

```shell
curl -XPOST 'localhost:9200/_search' -d '{
   "query" : {
    ...
   },
   "sort" : [
       {
          "offer.price" : {
             "mode" :  "avg",
             "order" : "asc",
             "nested_filter" : {
                "term" : { "offer.color" : "blue" }
             }
          }
       }
    ]
}'
```

### 4.3 missing值

`missing`参数指定缺失字段的文档的处理方式：将`missing`值设置为`_last`, `_first`或者自定义值

```
{
    "sort" : [
        { "price" : {"missing" : "_last"} },
    ],
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

### 4.4 ignoring unmapped字段

默认情况下，如果没有与字段相关联的映射，搜索请求将会失败。`unmapped_type`选项允许忽略没有映射的字段，不用它们排序。这个参数的值指定哪些排序值可以忽略。

```shell
{
    "sort" : [
        { "price" : {"unmapped_type" : "long"} },
    ],
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```
如果任何索引都没有price字段的映射，那么elasticsearch将会处理它，就好像有一个long类型的映射一样。

### 4.5 地理距离排序

通过`_geo_distance`排序。

```
{
    "sort" : [
        {
            "_geo_distance" : {
                "pin.location" : [-70, 40],
                "order" : "asc",
                "unit" : "km",
                "mode" : "min",
                "distance_type" : "sloppy_arc"
            }
        }
    ],
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```
- distance_type：怎样计算距离可以有`sloppy_arc`(默认),`arc`（更精确但是显著变慢）,`plane`(最快)

地理距离排序支持的排序`mode`有`max`，`min`和`avg`。

### 4.6 基于脚本的排序

```
{
    "query" : {
        ....
    },
    "sort" : {
        "_script" : {
            "script" : "doc['field_name'].value * factor",
            "type" : "number",
            "params" : {
                "factor" : 1.1
            },
            "order" : "asc"
        }
    }
}
```

### 4.7 追踪得分

当基于一个字段进行排序是，默认不计算score，通过设置track_scores为true，可以计算得分并且追踪。

```
{
    "track_scores": true,
    "sort" : [
        { "post_date" : {"reverse" : true} },
        { "name" : "desc" },
        { "age" : "desc" }
    ],
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

## 5 source过滤

用于控制`_source`字段的返回。默认情况下，操作返回`_source`字段的内容，除非你用到了`fields`参数，或者`_source`被禁用了。你能够通过`_source`参数关掉`_source`检索。

```
{
    "_source": false,
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```
`_source`也接受一个或者多个通配符模式控制返回值。

```shell
{
    "_source": "obj.*",
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
or
{
    "_source": [ "obj1.*", "obj2.*" ],
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```
最后，对于完整的控制，我们可以包含include和exclude。

```shell
{
    "_source": {
        "include": [ "obj1.*", "obj2.*" ],
        "exclude": [ "*.description" ],
    }
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

## 6 字段

允许选择性地加载文档特定的存储字段。

```shell
{
    "fields" : ["user", "postDate"],
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

`*`可以被用来加载文档中的所有字段。

如果`fields`数组为空，那么就只会返回`_id`和`_type`字段。

```shell
{
    "fields" : [],
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

为了向后兼容，如果`fields`参数指定的字段在文档中不存在，它将会加载`_source`，并从中抽取这个字段。这个功能已经被source过滤器替代。

从文档中获取的字段值总以数组的方式返回。但是源数据字段如`_routing`和`_parent`却从不以数组的方式返回。

只有叶子字段可以通过`field`选项返回。所以对象字段不能被返回，这样的操作会报错。

### partial

从`_source`加载数据时，partial字段可以用来使用通配符来控制哪部分的`_source`将被加载。例如

```shell
{
    "query" : {
        "match_all" : {}
    },
    "partial_fields" : {
        "partial1" : {
            "include" : "obj1.obj2.*",
        }
    }
}
```

或者

```shell
{
    "query" : {
        "match_all" : {}
    },
    "partial_fields" : {
        "partial1" : {
            "include" : "obj1.obj2.*",
            "exclude" : "obj1.obj3.*"
        }
    }
}
```

include和exclude都支持多模式

```shell
{
    "query" : {
        "match_all" : {}
    },
    "partial_fields" : {
        "partial1" : {
            "include" : ["obj1.obj2.*", "obj1.obj4.*"],
            "exclude" : "obj1.obj3.*"
        }
    }
}
```

## script fields

```shell
{
    "query" : {
        ...
    },
    "script_fields" : {
        "test1" : {
            "script" : "doc['my_field_name'].value * 2"
        },
        "test2" : {
            "script" : "doc['my_field_name'].value * factor",
            "params" : {
                "factor"  : 2.0
            }
        }
    }
}
```
script fields可以在没有保存的字段（如例子中的`my_field_name`）上工作，返回自定义的值（脚本算出的值）。

脚本也可以访问文档的`_source`字段，并抽取特定的元素（是一个对象类型）返回。

```shell
{
        "query" : {
            ...
        },
        "script_fields" : {
            "test1" : {
                "script" : "_source.obj1.obj2"
            }
        }
    }
```

了解`doc['my_field'].value`和`_source.my_field`之间的不同是很重要的。首先，使用doc关键字，会使相应的字段加载到内存，执行速度更快但是更耗费内存。第二，`doc[...]`符号
仅允许简单的值字段，只在基于字段的非分析或者单个项上有意义。

另一方面，`_source`加载、分析source，然后仅仅返回相关部分的json。

## field data fields

返回一个字段的字段数据表示，如下例

```shell
{
    "query" : {
        ...
    },
    "fielddata_fields" : ["test1", "test2"]
}
```

field data fields可以用于没有保存的字段。利用`fielddata_fields`参数会导致该字段的项加载到内存，增加内存的消耗。

## post filter

`post_filter`在搜索查询的最后，在聚合操作已经被计算后，应用于搜索的`hits`。下面用一个例子来说明。

假设你正在卖衬衣，用户指定了两个过滤器：`color:red`和`brand:gucci`。一般情况下，你可以用到`filtered query`

```shell
curl -XGET localhost:9200/shirts/_search -d '
{
  "query": {
    "filtered": {
      "filter": {
        "bool": {
          "must": [
            { "term": { "color": "red"   }},
            { "term": { "brand": "gucci" }}
          ]
        }
      }
    }
  }
}
'
```

假设你有一个`model`字段允许用户限制它们的搜索结果为red Gucci `t-shirts`或者`dress-shirts`。可以用`terms aggregation`。

```shell
curl -XGET localhost:9200/shirts/_search -d '
{
  "query": {
    "filtered": {
      "filter": {
        "bool": {
          "must": [
            { "term": { "color": "red"   }},
            { "term": { "brand": "gucci" }}
          ]
        }
      }
    }
  },
  "aggs": {
    "models": {
      "terms": { "field": "model" }
    }
  }
}
'
```

但是，也许你可能要告诉用户有多少其它颜色的Gucci shirts可以购买。如果你仅仅在`color`字段中加入`terms`聚合，那么你只会返回`red`的值，因为你的查询只返回红色的衬衣。

你想要在聚合中包含所有颜色的衬衣，然后只在搜索结果中应用`colors`过滤。可以用到`post_filter`

```shell
curl -XGET localhost:9200/shirts/_search -d '
{
  "query": {
    "filtered": {
      "filter": {
1        { "term": { "brand": "gucci" }}
      }
    }
  },
  "aggs": {
    "colors": {
2      "terms": { "field": "color" },
    },
    "color_red": {
      "filter": {
3        "term": { "color": "red" }
      },
      "aggs": {
        "models": {
4          "terms": { "field": "model" }
        }
      }
    }
  },
5  "post_filter": {
    "term": { "color": "red" },
  }
}
'
```

第1点，查询所有的衬衣，不管它是什么颜色

第2点，`colors`聚合返回流行颜色的衬衣

第3、4点，`color_red`聚合利用子聚合`models`限制red Gucci shirt

第5点，`post_filter`删除除了红色的其它颜色的结果

## search type

- query and fetch：参数是`query_and_fetch`,它在所有相关的分片上执行查询，返回结果。每个分片返回`size`个结果。因为每个分片返回`size`个结果，所以这个类型实际返回
`size`乘以分片个数的结果。
-  query then fetch：参数是`query_then_fetch`,查询也依赖于所有分片，但是只返回足够的信息(不是文档内容)。基于这个结果进行分类和排名，之后才访问相关分片的实际文档内容。
这个类型返回结果的实际个数是`size`。这是默认的类型，你不必指定一个特定的`search_type`。
- dfs query and fetch：参数是`dfs_query_and_fetch`,和`query_and_fetch`相似。除了初始scatter的阶段，这个阶段为了更精确的得分，计算分布式项频率。
- dfs_query_then_fetch：参数是`dfs_query_then_fetch`,和`query_then_fetch`相似。除了初始scatter的阶段，这个阶段为了更精确的得分，计算分布式项频率。
- count：参数是`count`,返回满足查询条件的`hits`的数量。
- scan：参数是`scan`,`scan`查询类型禁用排序，允许通过大型结果集非常有效的滚动（scrolling）。

## scroll

一个`search`查询返回一“页”的结果，`scroll` API可以用来检索大数量的结果(甚至所有结果)。它类似关系型数据库中的游标。

Scrolling并不用来作实时的用户查询，而是处理大数量的数据。

为了利用`scrolling`，初始的搜索请求应该在查询字符串中指定`scroll`参数，告诉Elasticsearch，需要保持`搜索上下文`存活多长时间。

```shell
curl -XGET 'localhost:9200/twitter/tweet/_search?scroll=1m' -d '
{
    "query": {
        "match" : {
            "title" : "elasticsearch"
        }
    }
}
'
```

上面的请求结果中包含一个`scroll_id`,它应该传递给`scroll` API去检索下一批数据。

```shell
curl -XGET  'localhost:9200/_search/scroll?scroll=1m'   \
     -d       'c2Nhbjs2OzM0NDg1ODpzRlBLc0FXNlNyNm5JWUc1'
```

### 保持查询上下文存活

`scroll`参数告诉Elasticsearch需要保持`搜索上下文`存活多长时间。它的值不需要存活足够长时间处理所有的数据，它只需要足够的时间处理前面的批结果数据。每一个`scroll`请求
设置了一个新的过期时间。

### clear scroll api

```shell
curl -XDELETE localhost:9200/_search/scroll \
     -d 'c2Nhbjs2OzM0NDg1ODpzRlBLc0FXNlNyNm5JWUc1'
```

```shell
curl -XDELETE localhost:9200/_search/scroll \
     -d 'c2Nhbjs2OzM0NDg1ODpzRlBLc0FXNlNyNm5JWUc1,aGVuRmV0Y2g7NTsxOnkxaDZ'
```

```shell
curl -XDELETE localhost:9200/_search/scroll/_all
```

##  min_score

```shell
{
    "min_score": 0.5,
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

返回的文档的得分小于`min_score`。










