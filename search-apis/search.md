# 搜索API

搜索API允许开发者执行搜索查询，返回匹配查询的搜索结果。这既可以通过查询字符串也可以通过查询体实现。

## 多索引多类型

所有的搜索API都可以跨多个类型使用，也可以通过多索引语法跨索引使用。例如，我们可以搜索twitter索引的跨类型的所有文档。

```shell
$ curl -XGET 'http://localhost:9200/twitter/_search?q=user:kimchy'
```

我们也可以带上特定的类型:

```shell
$ curl -XGET 'http://localhost:9200/twitter/tweet,user/_search?q=user:kimchy'
```

我们也可以搜索跨多个索引的所有文档

```shell
$ curl -XGET 'http://localhost:9200/kimchy,elasticsearch/tweet/_search?q=tag:wow'
```

或者我们也可以用`_all`占位符表示搜索所有可用的索引的所有推特。

```shell
$ curl -XGET 'http://localhost:9200/_all/tweet/_search?q=tag:wow'
```

或者搜索跨所有可用索引和所有可用类型的推特

```shell
$ curl -XGET 'http://localhost:9200/_search?q=tag:wow'
```