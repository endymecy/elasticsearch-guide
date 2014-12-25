# 搜索API

搜索API允许开发者执行一个搜索查询，返回满足查询条件的搜索信息。它能够跨索引以及跨类型执行。查询既可以用[Java查询API](query-dsl-queries.md)也可以用[Java过滤API](query-dsl-filters.md)。
查询的请求体由`SearchSourceBuilder`构建。

```java
import org.elasticsearch.action.search.SearchResponse;
import org.elasticsearch.action.search.SearchType;
import org.elasticsearch.index.query.FilterBuilders.*;
import org.elasticsearch.index.query.QueryBuilders.*;

SearchResponse response = client.prepareSearch("index1", "index2")
        .setTypes("type1", "type2")
        .setSearchType(SearchType.DFS_QUERY_THEN_FETCH)
        .setQuery(QueryBuilders.termQuery("multi", "test"))             // Query
        .setPostFilter(FilterBuilders.rangeFilter("age").from(12).to(18))   // Filter
        .setFrom(0).setSize(60).setExplain(true)
        .execute()
        .actionGet();
```

注意，所有的参数都是可选的。下面是最简洁的形式。

```java
// MatchAll on the whole cluster with all default options
SearchResponse response = client.prepareSearch().execute().actionGet();
```

## 在Java中使用scrolls

```java
import static org.elasticsearch.index.query.FilterBuilders.*;
import static org.elasticsearch.index.query.QueryBuilders.*;

QueryBuilder qb = termQuery("multi", "test");

SearchResponse scrollResp = client.prepareSearch(test)
        .setSearchType(SearchType.SCAN)
        .setScroll(new TimeValue(60000))
        .setQuery(qb)
        .setSize(100).execute().actionGet(); //100 hits per shard will be returned for each scroll
//Scroll until no hits are returned
while (true) {
    for (SearchHit hit : scrollResp.getHits()) {
        //Handle the hit...
    }
    scrollResp = client.prepareSearchScroll(scrollResp.getScrollId()).setScroll(new TimeValue(600000)).execute().actionGet();
    //Break condition: No hits are returned
    if (scrollResp.getHits().getHits().length == 0) {
        break;
    }
}
```

## 多搜索API

```java
SearchRequestBuilder srb1 = node.client()
    .prepareSearch().setQuery(QueryBuilders.queryString("elasticsearch")).setSize(1);
SearchRequestBuilder srb2 = node.client()
    .prepareSearch().setQuery(QueryBuilders.matchQuery("name", "kimchy")).setSize(1);

MultiSearchResponse sr = node.client().prepareMultiSearch()
        .add(srb1)
        .add(srb2)
        .execute().actionGet();

// You will get all individual responses from MultiSearchResponse#getResponses()
long nbHits = 0;
for (MultiSearchResponse.Item item : sr.getResponses()) {
    SearchResponse response = item.getResponse();
    nbHits += response.getHits().getTotalHits();
}
```

## 使用聚合

下面的例子显示怎样添加两个聚合到你的搜索中。

```java
SearchResponse sr = node.client().prepareSearch()
    .setQuery(QueryBuilders.matchAllQuery())
    .addAggregation(
            AggregationBuilders.terms("agg1").field("field")
    )
    .addAggregation(
            AggregationBuilders.dateHistogram("agg2")
                    .field("birth")
                    .interval(DateHistogram.Interval.YEAR)
    )
    .execute().actionGet();

// Get your facet results
Terms agg1 = sr.getAggregations().get("agg1");
DateHistogram agg2 = sr.getAggregations().get("agg2");
```

## 使用搜索模板

定义你的模板参数为`Map<String,String>`

```java
Map<String, String> template_params = new HashMap<>();
template_params.put("param_gender", "male");
```

你可以用你保存在`config/scripts`目录中的模板。例如，你拥有如下的文件`config/scripts/template_gender.mustache`

```
{
    "template" : {
        "query" : {
            "match" : {
                "gender" : "{{param_gender}}"
            }
        }
    }
}
```

可以通过如下方式执行：

```java
SearchResponse sr = client.prepareSearch()
        .setTemplateName("template_gender")
        .setTemplateType(ScriptService.ScriptType.FILE)
        .setTemplateParams(template_params)
        .get();
```

你也可以将模板存储在一个专门的索引中，这个索引名为`.scripts`

```java
client.preparePutIndexedScript("mustache", "template_gender",
        "{\n" +
        "    \"template\" : {\n" +
        "        \"query\" : {\n" +
        "            \"match\" : {\n" +
        "                \"gender\" : \"{{param_gender}}\"\n" +
        "            }\n" +
        "        }\n" +
        "    }\n" +
        "}").get();
```

为了用这个被索引的模板，需要用到`ScriptService.ScriptType.INDEXED`:

```java
SearchResponse sr = client.prepareSearch()
        .setTemplateName("template_gender")
        .setTemplateType(ScriptService.ScriptType.INDEXED)
        .setTemplateParams(template_params)
        .get();
```