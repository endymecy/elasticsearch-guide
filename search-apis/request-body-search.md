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












