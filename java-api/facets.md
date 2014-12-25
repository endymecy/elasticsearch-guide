# facets

Elasticsearch提供完整的java API用来支持facets。在查询的过程中，将需要计数的facets添加到`FacetBuilders`中。然后将该`FacetBuilders`条件到查询请求中。

```java
SearchResponse sr = node.client().prepareSearch()
        .setQuery( /* your query */ )
        .addFacet( /* add a facet */ )
        .execute().actionGet();
```

为了构建facets请求，需要用到`FacetBuilders`帮助类。你只需要在你的程序中导入它即可。

```java
import org.elasticsearch.search.facet.FacetBuilders.*;
```

## terms facet

### 准备一个facet请求

下面的例子新建一个facet请求

```java
FacetBuilders.termsFacet("f")
    .field("brand")
    .size(10);
```
### 利用facet响应

```java
import org.elasticsearch.search.facet.terms.*;

// sr is here your SearchResponse object
TermsFacet f = (TermsFacet) sr.getFacets().facetsAsMap().get("f");

f.getTotalCount();      // Total terms doc count
f.getOtherCount();      // Not shown terms doc count
f.getMissingCount();    // Without term doc count

// For each entry
for (TermsFacet.Entry entry : f) {
    entry.getTerm();    // Term
    entry.getCount();   // Doc count
}
```

## 范围facet

### 准备一个facet请求

下面的例子新建一个facet请求

```java
FacetBuilders.rangeFacet("f")
    .field("price")         // Field to compute on
    .addUnboundedFrom(3)    // from -infinity to 3 (excluded)
    .addRange(3, 6)         // from 3 to 6 (excluded)
    .addUnboundedTo(6);     // from 6 to +infinity
```
### 利用facet响应

```java
import org.elasticsearch.search.facet.range.*;

// sr is here your SearchResponse object
RangeFacet f = (RangeFacet) sr.getFacets().facetsAsMap().get("f");

// For each entry
for (RangeFacet.Entry entry : f) {
    entry.getFrom();    // Range from requested
    entry.getTo();      // Range to requested
    entry.getCount();   // Doc count
    entry.getMin();     // Min value
    entry.getMax();     // Max value
    entry.getMean();    // Mean
    entry.getTotal();   // Sum of values
}
```

## 直方图(Histogram) Facet

### 准备一个facet请求

下面的例子新建一个facet请求
```java
HistogramFacetBuilder facet = FacetBuilders.histogramFacet("f")
    .field("price")
    .interval(1);
```
### 利用facet响应

```java
import org.elasticsearch.search.facet.histogram.*;
// sr is here your SearchResponse object
HistogramFacet f = (HistogramFacet) sr.getFacets().facetsAsMap().get("f");

// For each entry
for (HistogramFacet.Entry entry : f) {
    entry.getKey();     // Key (X-Axis)
    entry.getCount();   // Doc count (Y-Axis)
}
```

## 日期直方图(Histogram) Facet

### 准备一个facet请求

下面的例子新建一个facet请求

```java
FacetBuilders.dateHistogramFacet("f")
    .field("date")      // Your date field
    .interval("year");  // You can also use "quarter", "month", "week", "day",
                        // "hour" and "minute" or notation like "1.5h" or "2w"
```

### 利用facet响应

```java
import org.elasticsearch.search.facet.datehistogram.*;
// sr is here your SearchResponse object
DateHistogramFacet f = (DateHistogramFacet) sr.getFacets().facetsAsMap().get("f");

// For each entry
for (DateHistogramFacet.Entry entry : f) {
    entry.getTime();    // Date in ms since epoch (X-Axis)
    entry.getCount();   // Doc count (Y-Axis)
}
```

## 过滤facet(不是facet过滤)

### 准备一个facet请求

下面的例子新建一个facet请求

```java
FacetBuilders.filterFacet("f",
    FilterBuilders.termFilter("brand", "heineken"));    // Your Filter here
```

### 利用facet响应

```java
import org.elasticsearch.search.facet.filter.*;

// sr is here your SearchResponse object
FilterFacet f = (FilterFacet) sr.getFacets().facetsAsMap().get("f");
f.getCount();   // Number of docs that matched
```

## 查询facet

### 准备一个facet请求

下面的例子新建一个facet请求

```java
FacetBuilders.queryFacet("f",
    QueryBuilders.matchQuery("brand", "heineken"));
```

### 利用facet响应

```java
import org.elasticsearch.search.facet.query.*;
// sr is here your SearchResponse object
QueryFacet f = (QueryFacet) sr.getFacets().facetsAsMap().get("f");

f.getCount();   // Number of docs that matched
```

## 统计


### 准备一个facet请求

下面的例子新建一个facet请求

```java
FacetBuilders.statisticalFacet("f")
   .field("price");
```

### 利用facet响应

```java
import org.elasticsearch.search.facet.statistical.*;
// sr is here your SearchResponse object
StatisticalFacet f = (StatisticalFacet) sr.getFacets().facetsAsMap().get("f");

f.getCount();           // Doc count
f.getMin();             // Min value
f.getMax();             // Max value
f.getMean();            // Mean
f.getTotal();           // Sum of values
f.getStdDeviation();    // Standard Deviation
f.getSumOfSquares();    // Sum of Squares
f.getVariance();        // Variance
```

## Terms Stats Facet

### 准备一个facet请求

下面的例子新建一个facet请求

```java
FacetBuilders.termsStatsFacet("f")
    .keyField("brand")
    .valueField("price");
```
### 利用facet响应

```java
// sr is here your SearchResponse object
TermsStatsFacet f = (TermsStatsFacet) sr.getFacets().facetsAsMap().get("f");
f.getTotalCount();      // Total terms doc count
f.getOtherCount();      // Not shown terms doc count
f.getMissingCount();    // Without term doc count

// For each entry
for (TermsStatsFacet.Entry entry : f) {
    entry.getTerm();            // Term
    entry.getCount();           // Doc count
    entry.getMin();             // Min value
    entry.getMax();             // Max value
    entry.getMean();            // Mean
    entry.getTotal();           // Sum of values
}
```

## 地理距离Facet

### 准备一个facet请求

下面的例子新建一个facet请求

```java
FacetBuilders.geoDistanceFacet("f")
    .field("pin.location")              // Field containing coordinates we want to compare with
    .point(40, -70)                     // Point from where we start (0)
    .addUnboundedFrom(10)               // 0 to 10 km (excluded)
    .addRange(10, 20)                   // 10 to 20 km (excluded)
    .addRange(20, 100)                  // 20 to 100 km (excluded)
    .addUnboundedTo(100)                // from 100 km to infinity (and beyond ;-) )
    .unit(DistanceUnit.KILOMETERS);     // All distances are in kilometers. Can be MILES
```

### 利用facet响应

```java
// sr is here your SearchResponse object
GeoDistanceFacet f = (GeoDistanceFacet) sr.getFacets().facetsAsMap().get("f");

// For each entry
for (GeoDistanceFacet.Entry entry : f) {
    entry.getFrom();            // Distance from requested
    entry.getTo();              // Distance to requested
    entry.getCount();           // Doc count
    entry.getMin();             // Min value
    entry.getMax();             // Max value
    entry.getTotal();           // Sum of values
    entry.getMean();            // Mean
}
```

# facet过滤器（不是过滤facet）

默认情况下，不管过滤器存在与否，facet都是作用在查询的结果集上。如果你需要计数带有过滤器的facet，你能够通过`AbstractFacetBuilder#facetFilter(FilterBuilder)`添加
过滤器到任何facet上。

```java
FacetBuilders
    .termsFacet("f").field("brand") // Your facet
    .facetFilter( // Your filter here
        FilterBuilders.termFilter("colour", "pale")
    );
```

例如，你可以在你的查询中重用创建的过滤器

```java
// A common filter
FilterBuilder filter = FilterBuilders.termFilter("colour", "pale");

TermsFacetBuilder facet = FacetBuilders.termsFacet("f")
    .field("brand")
    .facetFilter(filter);                           // We apply it to the facet

SearchResponse sr = node.client().prepareSearch()
    .setQuery(QueryBuilders.matchAllQuery())
    .setFilter(filter)                              // We apply it to the query
    .addFacet(facet)
    .execute().actionGet();
```

## 作用域

默认情况下，facet作用在查询的结果集上。但是，不管是哪个查询，你可以用`global`参数去计算来自于索引中的所有文档的facet。

```java
TermsFacetBuilder facet = FacetBuilders.termsFacet("f")
    .field("brand")
    .global(true);
```










