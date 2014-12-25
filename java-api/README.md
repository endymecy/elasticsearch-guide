# Java API

这节会介绍elasticsearch支持的Java API。所有的elasticsearch操作都使用[Client](client.md)对象执行。本质上，所有的操作都是并行执行的。

另外，Client中的操作有可能累积并通过[Bulk](bulk.md)执行。

## maven

Elasticsearch托管在Maven仓库中。例如，你可以在pom.xml中定义最新的版本。

```xml
<dependency>
    <groupId>org.elasticsearch</groupId>
    <artifactId>elasticsearch</artifactId>
    <version>${es.version}</version>
</dependency>
```
## 在jboss eap6模块中部署

Elasticsearch和Lucene类必须在相同的jboss模块中，你应该像如下方式定义`module.xml`。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<module name="org.elasticsearch">
  <resources>
    <!-- Elasticsearch -->
    <resource-root path="elasticsearch-1.4.1.jar"/>
    <!-- Lucene -->
    <resource-root path="lucene-core-4.10.2.jar"/>
    <resource-root path="lucene-analyzers-common-4.10.2.jar"/>
    <resource-root path="lucene-queries-4.10.2.jar"/>
    <resource-root path="lucene-memory-4.10.2.jar"/>
    <resource-root path="lucene-highlighter-4.10.2.jar"/>
    <resource-root path="lucene-queryparser-4.10.2.jar"/>
    <resource-root path="lucene-sandbox-4.10.2.jar"/>
    <resource-root path="lucene-suggest-4.10.2.jar"/>
    <resource-root path="lucene-misc-4.10.2.jar"/>
    <resource-root path="lucene-join-4.10.2.jar"/>
    <resource-root path="lucene-grouping-4.10.2.jar"/>
    <resource-root path="lucene-spatial-4.10.2.jar"/>
    <resource-root path="lucene-expressions-4.10.2.jar"/>
    <!-- Insert other resources here -->
  </resources>
  <dependencies>
    <module name="sun.jdk" export="true" >
        <imports>
            <include path="sun/misc/Unsafe" />
        </imports>
    </module>
    <module name="org.apache.log4j"/>
    <module name="org.apache.commons.logging"/>
    <module name="javax.api"/>
  </dependencies>
</module>
```

* [客户端](client.md)
* [索引API](index-api.md)
