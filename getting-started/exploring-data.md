# 操作数据

## 样本数据集

现在我们对于基本的东西已经有了一些认识，现在让我们尝试使用一些更加贴近现实的数据集。我准备了一些假想的客户银行账户信息的JSON文档样本。文档具有以下的模式（schema）：

```
{
    "account_number": 0,
    "balance": 16623,
    "firstname": "Bradshaw",
    "lastname": "Mckenzie",
    "age": 29,
    "gender": "F",
    "address": "244 Columbus Place",
    "employer": "Euron",
    "email": "bradshawmckenzie@euron.com",
    "city": "Hobucken",
    "state": "CO"
}
```
可以通过[www.json-generator.com/](http://www.json-generator.com/)自动生成这些数据。

## 载入样本数据

你可以在[这里](https://github.com/bly2k/files/blob/master/accounts.zip?raw=true)下载样本数据集。将其解压到当前目录下并加载到我们的集群里：

```shell
curl -XPOST 'localhost:9200/bank/account/_bulk?pretty' --data-binary @accounts.json
curl 'localhost:9200/_cat/indices?v'
```

响应是：

```shell
curl 'localhost:9200/_cat/indices?v'
health index pri rep docs.count docs.deleted store.size pri.store.size
yellow bank    5   1       1000            0    424.4kb        424.4kb
```

这意味着我们成功批量索引了1000个文档到银行索引中（在account类型下）。

## 搜索API

现在，让我们以一些简单的搜索来开始学习。有两种基本的方式来运行搜索：一种是在REST请求的URI中发送搜索参数，另一种是将搜索参数发送到REST请求体中。请求体方法的表达能力更好，并且你可以使用更加可读的JSON格式来定义搜索。我们将尝试使用一次请求URI作为例子，但是教程的后面部分，我们将仅仅使用请求体方法。

搜索的 REST API 可以通过_search终点(endpoint)来访问。下面这个例子返回bank索引中的所有的文档：

```shell
curl 'localhost:9200/bank/_search?q=*&pretty'
```
我们仔细研究一下这个查询调用。我们在bank索引中搜索（ `_search`终点），并且`q=*`参数指示Elasticsearch去匹配这个索引中所有的文档。pretty参数仅仅是告诉Elasticsearch返回美观的JSON结果。

以下是响应（部分列出）：

```
curl 'localhost:9200/bank/_search?q=*&pretty'
{
  "took" : 63,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "failed" : 0
  },
  "hits" : {
    "total" : 1000,
    "max_score" : 1.0,
    "hits" : [ {
      "_index" : "bank",
      "_type" : "account",
      "_id" : "1",
      "_score" : 1.0, "_source" : {"account_number":1,"balance":39225,"firstname":"Amber","lastname":"Duke","age":32,"gender":"M","address":"880 Holmes Lane","employer":"Pyrami","email":"amberduke@pyrami.com","city":"Brogan","state":"IL"}
    }, {
      "_index" : "bank",
      "_type" : "account",
      "_id" : "6",
      "_score" : 1.0, "_source" : {"account_number":6,"balance":5686,"firstname":"Hattie","lastname":"Bond","age":36,"gender":"M","address":"671 Bristol Street","employer":"Netagy","email":"hattiebond@netagy.com","city":"Dante","state":"TN"}
    }, {
      "_index" : "bank",
      "_type" : "account",
```

对于这个响应，我们可以看到如下的部分：

- `took`：Elasticsearch 执行这个搜索的耗时，以毫秒为单位
- `timed_out`：指明这个搜索是否超时
- `_shards`：指出多少个分片被搜索了，同时也指出了成功/失败的被搜索的shards 的数量
- `hits`：搜索结果
- `hits.total`：匹配查询条件的文档的总数目
- `hits.hits`：真正的搜索结果数组（默认是前10个文档）
- `_score` 和 `max_score`：现在先忽略这些字段

使用请求体方法的等价搜索是：

```shell
curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
{
  "query": { "match_all": {} }
}'
```

与上面方法不同之处在于，并不是向URI中传递q=*，取而代之的是在_search API的请求体中POST了一个JSON格式的请求体。我们将在下一部分中讨论这个JSON查询。

有一点需要重点理解的是，一旦你取回了搜索结果，Elasticsearch就完成了使命，它不会维护任何服务器端的资源或者在你的结果中打开游标。这是和其它类似SQL的平台的一个鲜明的对比， 在那些平台上，你可以在前面先获取你查询结果的子集，
然后如果你想获取结果的剩余部分，你必须继续返回服务端去取，这个数据集使用了一种有状态的服务器端游标技术。

## 介绍查询语言

Elasticsearch 提供一种JSON风格的特定领域语言，利用它你可以执行查询。这中语言称为DSL。这个查询语言非常全面，最好的学习方法就是以几个基础的例子来开始。

回到上一个例子，我们执行了这个查询：

```
{
  "query": { "match_all": {} }
}
```
分析以上的这个查询，其中的query部分告诉我查询的定义，match_all部分就是我们想要运行的查询的类型。match_all查询，就是简单地查询一个指定索引下的所有的文档。

除了这个query参数之外，我们也可以通过传递其它的参数来影响搜索结果。比如，下面做了一次`match_all`查询并只返回第一个文档：

```shell
curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
{
  "query": { "match_all": {} },
  "size": 1
}'
```
注意，如果没有指定`size`的值，那么它默认就是10。

下面的例子，做了一次`match_all`查询并且返回第11到第20个文档：

```shell
curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
{
  "query": { "match_all": {} },
  "from": 10,
  "size": 10
}'
```
其中的 from 参数指明从哪个文档开始，size参数指明从from参数开始，要返回的文档数。这个特性对于搜索结果分页来说非常有帮助。注意，如果不指定from的值，它默认就是0。

下面这个例子做了一次`match_all`查询并且以账户余额降序排序，最后返前十个文档：

```shell
curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
{
  "query": { "match_all": {} },
  "sort": { "balance": { "order": "desc" } }
}'
```

## 执行搜索

现在我们已经知道了几个基本的参数， 让我们进一步学习查询语言。首先我们看一下返回文档的字段。 默认情况下，是返回完整的JSON文档的。这可以通过`source`来引用（搜索`hits`中的`_sourcei`字段）。如果我们不想返回完整的源文档，我们可以指定返回的几个字段。

下面这个例子说明了从搜索中只返回两个字段`account_number`和`balance`（当然，这两个字段都是指`_source`中的字段），以下是具体的搜索：
```shell
curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
{
  "query": { "match_all": {} },
  "_source": ["account_number", "balance"]
}'
```

注意到上面的例子简化了`_source`字段,它仍将会返回一个叫做`_source`的字段，但是仅仅包含`account_number`和`balance`两个字段。

如果你有SQL背景，上述查询在概念上有些像SQL的SELECT FROM。

现在让我们进入到查询部分。之前，我们学习了`match_all`查询是怎样匹配到所有的文档的。现在我们介绍一种新的查询，叫做`match`查询，这可以看成是一个简单的字段搜索查询（比如对某个或某些特定字段的搜索）

下面这个例子返回账户编号为 20 的文档：

```shell
curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
{
  "query": { "match": { "account_number": 20 } }
}'
```

下面这个例子返回地址中包含词语(term)“mill”的所有账户：

```shell
curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
{
  "query": { "match": { "address": "mill" } }
}'
```

下面这个例子返回地址中包含词语“mill” 或者“lane” 的账户：

```shell
curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
{
  "query": { "match": { "address": "mill lane" } }
}'
```

下面这个例子是`match`的变体（`match_phrase`），它会去匹配短语“mill lane”：

```shell
curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
{
  "query": { "match_phrase": { "address": "mill lane" } }
}'
```

现在，让我们介绍一下布尔查询。布尔查询允许我们利用布尔逻辑将较小的查询组合成较大的查询。

现在这个例子组合了两个`match`查询，这个组合查询返回包含“mill” 和“lane” 的所有的账户

```shell
curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
{
  "query": {
    "bool": {
      "must": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}'
```

在上面的例子中，`bool must`语句指明了，对于一个文档，所有的查询都必须为真，这个文档才能够匹配成功。

相反的， 下面的例子组合了两个`match`查询，它返回的是地址中包含“mill” 或者“lane”的所有的账户:

```shell
curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
{
  "query": {
    "bool": {
      "should": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}'
```

在上面的例子中`bool should`语句指明，对于一个文档，查询列表中，只要有一个查询匹配，那么这个文档就被看成是匹配的。

现在这个例子组合了两个查询，它返回地址中既不包含“mill”，同时也不包含“lane”的所有的账户信息：

```shell
curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
{
  "query": {
    "bool": {
      "must_not": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}'
```

在上面的例子中，`bool must_not`语句指明，对于一个文档，查询列表中的的所有查询都必须都不为真，这个文档才被认为是匹配的。

我们可以在一个bool查询里一起使用must、should、must_not。 此外，我们可以将bool查询放到这样的bool语句中来模拟复杂的、多层级的布尔逻辑。

下面这个例子返回40岁以上并且不生活在ID（aho）的人的账户：

```shell
curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
{
  "query": {
    "bool": {
      "must": [
        { "match": { "age": "40" } }
      ],
      "must_not": [
        { "match": { "state": "ID" } }
      ]
    }
  }
}'
```

## 执行过滤器

在前面的章节中，我们跳过了文档得分的细节（搜索结果中的`_score`字段）。这个得分是指定的搜索查询匹配程度的一个相对度量。得分越高，文档越相关，得分越低文档的相关度越低。

Elasticsearch中的所有的查询都会触发相关度得分的计算。对于那些我们不需要相关度得分的场景下，Elasticsearch以过滤器的形式提供了另一种查询功能。过滤器在概念上类似于查询，但是它们有非常快的执行速度，这种快的执行速度主要有以下两个原因：

- 过滤器不会计算相关度的得分，所以它们在计算上更快一些
- 过滤器可以被缓存到内存中，这使得在重复的搜索查询上，其要比相应的查询快出许多。

为了理解过滤器，我们先来介绍“被过滤” 的查询，这使得你可以将一个查询（如`match_all`,`match`，`bool`等）和一个过滤器结合起来。作为一个例子，我们介绍一下范围过滤器，它允许我们通过一个区间的值来过滤文档。这通常被用在数字和日期的过滤上。

这个例子使用一个被过滤的查询，其返回值是存款在20000到30000之间（闭区间)的所有账户。换句话说，我们想要找到存款大于等于20000并且小于等于30000的账户。

```scala
curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
{
  "query": {
    "filtered": {
      "query": { "match_all": {} },
      "filter": {
        "range": {
          "balance": {
            "gte": 20000,
            "lte": 30000
          }
        }
      }
    }
  }
}'
```

分析上面的例子，被过滤的查询包含一个`match_all`查询（查询部分）和一个过滤器（`filter`部分）。我们可以在查询部分中放入其他查询，在`filter`部分放入其它过滤器。 在上面的应用场景中，由于所有的在这个范围之内的文档都是平等的（或者说相关度都是一样的），
没有一个文档比另一个文档更相关，所以这个时候使用范围过滤器就非常合适了

通常情况下，要决定是使用过滤器还是使用查询，你就需要问自己是否需要相关度得分。如果相关度是不重要的，使用过滤器，否则使用查询。如果你有SQL背景，查询和过滤器
在概念上类似于`SELECT WHERE`语句，一般情况下过滤器比查询用得更多。

除了`match_all`, `match`, `bool`,`filtered`和`range`查询，还有很多其它类型的查询/过滤器，我们这里不会涉及。由于我们已经对它们的工作原理有了基本的理解，将其应用到其它类型的查询、过滤器上也不是件难事。

## 执行聚合

聚合提供了分组并统计数据的能力。理解聚合的最简单的方式是将其粗略地等同为SQL的GROUP BY和SQL聚合函数。在Elasticsearch中，你可以在一个响应中同时返回命中的数据和聚合结果。你可以使用简单的API同时运行查询和多个聚合并一次返回，这避免了来回的网络通信，是非常强大和高效的。

作为开始的一个例子，我们按照state分组，并按照州名的计数倒序排序：

```shell
curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state"
      }
    }
  }
}'
```
在SQL中，上面的聚合在概念上类似于：

```sql
SELECT COUNT(*) from bank GROUP BY state ORDER BY COUNT(*) DESC
```

响应（其中一部分）是：

```
 "hits" : {
    "total" : 1000,
    "max_score" : 0.0,
    "hits" : [ ]
  },
  "aggregations" : {
    "group_by_state" : {
      "buckets" : [ {
        "key" : "al",
        "doc_count" : 21
      }, {
        "key" : "tx",
        "doc_count" : 17
      }, {
        "key" : "id",
        "doc_count" : 15
      }, {
        "key" : "ma",
        "doc_count" : 15
      }, {
        "key" : "md",
        "doc_count" : 15
      }, {
        "key" : "pa",
        "doc_count" : 15
      }, {
        "key" : "dc",
        "doc_count" : 14
      }, {
        "key" : "me",
        "doc_count" : 14
      }, {
        "key" : "mo",
        "doc_count" : 14
      }, {
        "key" : "nd",
        "doc_count" : 14
      } ]
    }
  }
}
```

我们可以看到AL（abama）有21个账户，TX有17 个账户，ID（aho）有15个账户，依此类推。

注意我们将`size`设置成 0，这样我们就可以只看到聚合结果了，而不会显示命中的结果。

在先前聚合的基础上，现在这个例子计算了每个州的账户的平均存款（还是按照账户数量倒序排序的前10个州）：

```shell
curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state"
      },
      "aggs": {
        "average_balance": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}'
```

注意， 我们把`average_balance`聚合嵌套在了`group_by_state`聚合之中。这是所有聚合的一个常用模式。你可以在任意的聚合之中嵌套聚合，这样就可以从你的数据中抽取出想要的结果。

在前面的聚合的基础上，现在让我们按照平均余额进行排序：

```shell
curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state",
        "order": {
          "average_balance": "desc"
        }
      },
      "aggs": {
        "average_balance": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}'
```

下面的例子显示了如何使用年龄段（20-29，30-39，40-49）分组，然后再用性别分组，最后为每一个年龄段的每组性别计算平均账户余额。

```shell
curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
{
  "size": 0,
  "aggs": {
    "group_by_age": {
      "range": {
        "field": "age",
        "ranges": [
          {
            "from": 20,
            "to": 30
          },
          {
            "from": 30,
            "to": 40
          },
          {
            "from": 40,
            "to": 50
          }
        ]
      },
      "aggs": {
        "group_by_gender": {
          "terms": {
            "field": "gender"
          },
          "aggs": {
            "average_balance": {
              "avg": {
                "field": "balance"
              }
            }
          }
        }
      }
    }
  }
}'
```




