# 实验实例

## 测试环境

两台4核8G内存的机器组成的集群。网络带宽为100m。JVM配置最大内存为2g。elasticsearch有10个分片，1个副本。通过导入mysql的数据来进行实验。

## 导入数据

需要下载[相关插件](https://github.com/jprante/elasticsearch-river-jdbc#time-based-selecting)。

将ODS的yourtable表中所有数据导入到Elasticsearch中，用来进行测试。
```shell
curl -XPUT 'localhost:9200/_river/my_jdbc_river/_meta' -d '{
"type" : "jdbc",
    "jdbc" : {
        "url" : "jdbc:mysql://yourhost:3306/yourdb",
        "user" : "*",
        "password" : "*",
        "index" : "top_user",
        "type" : " top_user",
        "sql" : "select * from yourtable"
}
}'
```
yourtable表中总共有18个字段，类型包括bigint，int，varchar，datetime，总共有15292729条数据，数据总量为4.2G。其主要字段如下所示：

Field | Type
--- | ---
buyer_nick | varchar(50)
sex | varchar(10)
level | int(10)
score | int(10)
city | varchar(50)
created | datetime


## 基本操作

### 查看节点信息

```shell
curl 'localhost:9200/_cat/nodes?v'
```

### 查看索引信息

```shell
curl 'localhost:9200/_cat/indices?v'
```

### 删除索引信息

```shell
curl -XDELETE 'localhost:9200/top_user?pretty'
curl -XDELETE 'localhost:9200/top_trade?pretty'
curl -XDELETE 'localhost:9200/_river?pretty'
```

## 例子

这些例子统计了5次运行的时间。有些查询在mysql上查询非常缓慢，所以没有将其运行时间列出。

### 查看所有数据（默认返回10条）

```shell
curl -XPOST 'localhost:9200/top_user/_search?pretty' -d '
{
"query": { "match_all": {} }
}'
```

次数 | 运行时间（ms）
--- | ---
1 | 260
2 | 124
3 | 39
4 | 38
5 | 47

等价于的sql语句是：`select * from top_user limit 0,10;`

### 分页查询（20-40）

```shell
curl -XPOST 'localhost:9200/top_user/_search?pretty' -d '
{
"query": { "match_all": {} },
"from": 20,
"size": 40
}'
```

次数 | 运行时间（ms）
--- | ---
1 | 54
2 | 59
3 | 45
4 | 49
5 | 76

等价于的sql语句是：`select * from top_user limit 20,40;`

### 按照字段排序（默认返回10条）

```shell
curl -XPOST 'localhost:9200/top_user/_search?pretty' -d '
{
"query": { "match_all": {} },
"sort": { "sex": { "order": "asc" } }
}'
```

次数 | 运行时间（ms）
--- | ---
1 | 982
2 | 1087
3 | 208
4 | 147
5 | 114

等价于的sql语句是：`select * from top_user order by sex asc limit 0,10;`


### 指定_source字段

```shell
curl -XPOST 'localhost:9200/top_user/_search?pretty' -d '
{
    "query": { "match_all": {} },
    "_source": ["buyer_nick", "sex"]
}'
```

次数 | 运行时间（ms）
--- | ---
1 | 92
2 | 108
3 | 53
4 | 44
5 | 79

等价于的sql语句是：select buyer_nick from top_user;

### match查询

```shell
curl -XPOST 'localhost:9200/top_user/_search?pretty' -d '
{
"query": {
    "bool": {
        "should": [
            { "match": { "city": "上海" } },
            { "match": { "city": "北京" } }
        ]
     }
 }
}'
```

次数 | 运行时间（ms）
--- | ---
1 | 1410
2 | 108
3 | 57
4 | 159
5 | 79

等价于的sql语句是：`select * from top_user where city="上海" or city="北京" limit 0,10;`

### 通过过滤器查询

```shell
curl -XPOST 'localhost:9200/top_user/_search?pretty' -d '
{
    "query": {
        "filtered": {
            "query": { "match_all": {} },
            "filter": {
                "range": {
                    "score": {
                        "gte": 200,
                        "lte": 2000
                    }
                }
            }
        }
    }
}'
```

次数 | 运行时间（ms）
--- | ---
1 | 1644
2 | 1096
3 | 57
4 | 30
5 | 31

等价于的sql语句是：`select * from top_user where score>200 and score<2000 limit 0,10;`

### 聚合查询

```shell
curl -XPOST 'localhost:9200/top_user/_search?pretty' -d '
{
    "size": 0,
    "aggs": {
        "group_by_city": {
            "terms": {
                "field": "city"
            }
        }
    }
}'
```
次数 | 运行时间（ms）
--- | ---
1 | 2941
2 | 1887
3 | 542
4 | 659
5 | 612

等价于的sql语句是: `select count(*) from top_user group by city order by count(*) desc limit 0,10;`

一下得到每个level的用户的平均得分，并按照得分排序

```shell
curl -XPOST 'localhost:9200/top_user/_search?pretty' -d '
{
    "size": 0,
    "aggs": {
        "group_by_level": {
            "terms": {
                "field": "level",
                "order": {
                    "average_score": "desc"
                }
            },
            "aggs": {
                "average_score": {
                    "avg": {
                        "field": "score"
                    }
                }
            }
        }
    }
}'
```

次数 | 运行时间（ms）
--- | ---
1 | 3332
2 | 2706
3 | 599
4 | 633
5 | 542

等价于的sql语句是: `select max(score) as avg_score from top_user group by level order by avg_score desc limit 0,10;`


## 总结

与mysql数据库查询相对比，可以发现Elasticsearch在聚合查询和非主键排序上效果提升明显。如下面两种情况，mysql很慢，但是Elasticsearch较快。

- select * from top_user order by sex asc limit 20,40;
- select count(*) from top_user group by city order by count(*) desc limit 0,10;