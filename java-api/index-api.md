# 索引API

索引API允许开发者索引类型化的JSON文档到一个特定的索引，使其可以被搜索。

## 生成JSON文档

有几种不同的方式生成JSON文档

- 利用`byte[]`或者作为一个`String`手动生成
- 利用一个`Map`将其自动转换为相应的JSON
- 利用第三方库如[Jackson](http://wiki.fasterxml.com/JacksonHome)去序列化你的bean
- 利用内置的帮助函数XContentFactory.jsonBuilder()

### 手动生成

需要注意的是，要通过[Date Format](http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/mapping-date-format.html)编码日期。

```java
String json = "{" +
        "\"user\":\"kimchy\"," +
        "\"postDate\":\"2013-01-30\"," +
        "\"message\":\"trying out Elasticsearch\"" +
    "}";
```

### 使用map

```java
Map<String, Object> json = new HashMap<String, Object>();
json.put("user","kimchy");
json.put("postDate",new Date());
json.put("message","trying out Elasticsearch");
```

### 序列化bean

elasticsearch早就用到了Jackson，把它放在了`org.elasticsearch.common.jackson`下面。你可以在你的`pom.xml`文件里面添加你自己的Jackson版本。

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.1.3</version>
</dependency>
```
这样，你就可以序列化你的bean为JSON。

```java
import com.fasterxml.jackson.databind.*;

// instance a json mapper
ObjectMapper mapper = new ObjectMapper(); // create once, reuse

// generate json
String json = mapper.writeValueAsString(yourbeaninstance);
```

### 利用elasticsearch帮助类

elasticsearch提供了内置的帮助类来将数据转换为JSON

```java
import static org.elasticsearch.common.xcontent.XContentFactory.*;

XContentBuilder builder = jsonBuilder()
    .startObject()
        .field("user", "kimchy")
        .field("postDate", new Date())
        .field("message", "trying out Elasticsearch")
    .endObject()
```

注意，你也可以使用`startArray(String)`和`endArray()`方法添加数组。另外，`field`可以接收任何类型的对象，你可以直接传递数字、时间甚至XContentBuilder对象。

可以用下面的方法查看json。

```java
String json = builder.string();
```

## 索引文档

下面的例子将JSON文档索引为一个名字为“twitter”，类型为“tweet”，id值为1的索引。

```java
import static org.elasticsearch.common.xcontent.XContentFactory.*;

IndexResponse response = client.prepareIndex("twitter", "tweet", "1")
        .setSource(jsonBuilder()
                    .startObject()
                        .field("user", "kimchy")
                        .field("postDate", new Date())
                        .field("message", "trying out Elasticsearch")
                    .endObject()
                  )
        .execute()
        .actionGet();
```

你也可以不提供id:

```java
String json = "{" +
        "\"user\":\"kimchy\"," +
        "\"postDate\":\"2013-01-30\"," +
        "\"message\":\"trying out Elasticsearch\"" +
    "}";

IndexResponse response = client.prepareIndex("twitter", "tweet")
        .setSource(json)
        .execute()
        .actionGet();
```

`IndexResponse`将会提供给你索引信息

```java
// Index name
String _index = response.getIndex();
// Type name
String _type = response.getType();
// Document ID (generated or not)
String _id = response.getId();
// Version (if it's the first time you index this document, you will get: 1)
long _version = response.getVersion();
```

如果你在索引时提供了过滤，那么`IndexResponse`将会提供一个过滤器（percolator ）

```java
IndexResponse response = client.prepareIndex("twitter", "tweet", "1")
        .setSource(json)
        .execute()
        .actionGet();

List<String> matches = response.matches();
```

