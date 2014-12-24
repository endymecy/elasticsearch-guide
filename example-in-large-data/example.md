# 实验实例

## 测试环境

两台4核8G内存的机器组成的集群。网络带宽为100m。JVM配置最大内存为2g。elasticsearch有10个分片，1个副本。通过导入mysql的数据来进行实验。

## 导入数据

将ODS的top_user表中所有数据导入到elastic中，用来进行测试。
```
curl -XPUT 'localhost:9200/_river/my_jdbc_river/_meta' -d '{
"type" : "jdbc",
    "jdbc" : {
        "url" : "jdbc:mysql://host:3306/rds_ods_01",
        "user" : "*",
        "password" : "*",
        "index" : "top_user",
        "type" : " top_user",
        "sql" : "select * from top_user"
}
}'
```
Top_user表中总共有18个字段，类型包括bigint，int，varchar，datetime，总共有15292729条数据，数据总量为4.2G。

## 基本操作

### 查看节点信息

```
curl 'localhost:9200/_cat/nodes?v'
```

### 查看索引信息

```
curl 'localhost:9200/_cat/indices?v'
```

### 删除索引信息

```
curl -XDELETE 'localhost:9200/top_user?pretty'
curl -XDELETE 'localhost:9200/top_trade?pretty'
curl -XDELETE 'localhost:9200/_river?pretty'
```

## 例子

这些例子的运行时间以第一次运行为准。以后运行的运行时间比第一次运行的运行时间会大幅缩短。

### 查看所有数据（默认返回10条）

```
curl -XPOST 'localhost:9200/top_user/_search?pretty' -d '
{
"query": { "match_all": {} }
}'
```
运行时间180ms

等价于的sql语句是：`select * from top_user;`

### 分页查询（20-40）

```
curl -XPOST 'localhost:9200/top_user/_search?pretty' -d '
{
"query": { "match_all": {} },
"from": 20,
"size": 40
}'
```
运行时间67ms

等价于的sql语句是：`select * from top_user limit 20,40;`

### 按照字段排序（默认返回10条）

```
curl -XPOST 'localhost:9200/top_user/_search?pretty' -d '
{
"query": { "match_all": {} },
"sort": { "buyer_nick": { "order": "asc" } }
}'
```
运行时间5152ms

等价于的sql语句是：`select * from top_user order by buyer_nick asc;`

```
curl -XPOST 'localhost:9200/top_user/_search?pretty' -d '
{
"query": { "match_all": {} },
"sort": { "buyer_nick": { "order": "asc" } },
"from": 20,
"size": 40
}'
```

运行时间5091ms

等价于的sql语句是：`select * from top_user order by buyer_nick asc limit 20,40;`

### 指定_source字段

```
curl -XPOST 'localhost:9200/top_user/_search?pretty' -d '
{
    "query": { "match_all": {} },
    "_source": ["buyer_nick", "sex"]
}'
```
运行时间22ms

等价于的sql语句是：select buyer_nick from top_user;

### match查询

```
curl -XPOST 'localhost:9200/top_user/_search?pretty' -d '
{
"query": { "match": { "city": "上海" } }
}'
```
运行时间358ms

等价于的sql语句是：`select * from top_user where city="上海";`

```
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
运行时间427ms

等价于的sql语句是：`select * from top_user where city="上海" or city="北京";`

### 通过过滤器查询

```
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

运行时间649ms

等价于的sql语句是：`select * from top_user where score>200 and score<2000;`

### 聚合查询

```
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
运行时间1079ms

等价于的sql语句是: `select count(*) from top_user group by city order by count(*) desc;`

一下得到每个level的用户的平均得分，并按照得分排序

```
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

运行时间1846ms

等价于的sql语句是: `select max(score) as avg_score from top_user group by level order by avg_score desc;`

