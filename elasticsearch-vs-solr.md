# elasticsearch优点

elasticsearch相比于solr拥有一些重要特征：

- ElasticSearch是分布式的。不需要其他组件，分发是实时的，被叫做"Push replication"。
- ElasticSearch 完全支持Apache Lucene的接近实时的搜索。
- ElasticSearch 采用Gateway概念，使得全备份更简单。
- 支持更多的客户端库，如PHP, Ruby, Perl, Scala, Python, .NET, Javascript, Erlang, Clojure
- 自包含集群
- 自动化插件安装
- 集合脚本语言查询
- 导入性能更好，查询性能与solr持平

# elasticsearch vs solr

[比较1](http://stackoverflow.com/questions/10213009/solr-vs-elasticsearch)
[比较2](http://solr-vs-elasticsearch.com/)

# elasticsearch社区版和商业版

社区版和商业版几乎没有区别，但是商用版提供了marvel工具，用于监控集群的状态。

# 下一步的工作

这里要特别提出权限验证的问题。对于商用产品权限验证是必须的。两者都不支持权限验证。需要用户自己增加这一块的功能。然而就目前来看，两者在增加该功能的难易程度来说，elasticsearch是相对容易很多。原因为elasticsearch内部节点之间使用自定义协议；
而Solr4.x内部节点之间使用了和外部一样的http协议，并且节点间的通信关系混乱，所以对目前的Solr4.x增加权限验证以及ACL将很难寻找到一个优雅的解决方案，但并不是不可以。


现在，elasticsearch正在致力于开发shield项目，在不久的将来就会实现。但是并不清楚它是否会收费。

#相关资料

[elasticsearch权威指南](http://fuxiaopang.gitbooks.io/learnelasticsearch/)
[elasticsearch下载国内镜像](http://pan.baidu.com/s/1bnEjYkZ)
[Kibana 中文指南](http://chenryn.gitbooks.io/kibana-guide-cn/)

