# **ElasticSearch 分布式搜索**

### **1、分⽚与集群的故障转移**

**集群健康状态**

```
GET /_cluster/health
```

### **2、分⽚及其生命周期**

**将 Index buffer 写入 Segment 的过程叫 Refresh。**

**Segment的merge操作**

```
POST index/_forcemerge
```

### **3、剖析分布式查询及相关性算分**

**搜索的URL 中指定参数 `“_search?search_type=dfs_query_then_fetch”`**

```
PUT message
{
  "settings": {
    "number_of_shards": 20
  }
}

POST message/_doc?routing=2
{
  "content":"good morning"
}
```

```
POST message/_search
{
  "explain": true,
  "query": {
    "match_all": {}
  }
}
```

**使⽤ DFS Query Then Fetch**

```
POST message/_search?search_type=dfs_query_then_fetch
{

  "query": {
    "term": {
      "content": {
        "value": "good"
      }
    }
  }
}
```

### **4、排序及Doc Values&Fielddata**

**1、排序**

* Elasticsearch 默认采⽤相关性算分对结果进⾏降序排序
* 可以通过设定 sorting 参数，⾃⾏设定排序
* 如果不指定`_score`，算分为 Null

**单字段排序**

```
#单字段排序
POST /kibana_sample_data_ecommerce/_search
{
  "size": 5,
  "query": {
    "match_all": {

    }
  },
  "sort": [
    {"order_date": {"order": "desc"}}
  ]
}
```

**多字段进行排序**

```
#多字段排序
POST /kibana_sample_data_ecommerce/_search
{
  "size": 5,
  "query": {
    "match_all": {

    }
  },
  "sort": [
    {"order_date": {"order": "desc"}},
    {"_doc":{"order": "asc"}},
    {"_score":{ "order": "desc"}}
  ]
}
```

**对 Text 类型排序**

```
#对 text 字段进行排序。默认会报错，需打开fielddata
POST /kibana_sample_data_ecommerce/_search
{
  "size": 5,
  "query": {
    "match_all": {

    }
  },
  "sort": [
    {"customer_full_name": {"order": "desc"}}
  ]
}
```

* fielddata: 默认关闭



**2、打开 text的 fielddata**

```
PUT kibana_sample_data_ecommerce/_mapping
{
  "properties": {
    "customer_full_name" : {
          "type" : "text",
          "fielddata": true,
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        }
  }
}
```

```
POST /kibana_sample_data_ecommerce/_search
{
  "size": 5,
  "query": {
    "match_all": {

    }
  },
  "sort": [
    {"customer_full_name": {"order": "desc"}}
  ]
}
```


**3、关闭 Doc Values**

* 默认启⽤，可以通过 Mapping 设置关闭
* 如果重新打开，需要重建索引
	* 什么时候需要关闭
		* **明确不不需要做排序及聚合分析**


```
PUT test_keyword/_mapping
{
  "properties": {
    "user_name":{
      "type": "keyword",
      "doc_values":false
    }
  }
}
```

or

```
PUT test_text/_mapping
{
  "properties": {
    "intro":{
      "type": "text",
      "doc_values":true
    }
  }
}
```

**获取 Doc Values & Fielddata 中存储的内容**

```
PUT temp_users/_mapping
{
  "properties": {
    "name":{"type": "text","fielddata": true},
    "desc":{"type": "text","fielddata": true}
  }
}
```

**查看 `docvalue_fields` 数据**


```
POST  temp_users/_search
{
  "docvalue_fields": [
    "name","desc"
    ]
}
```


### **5、分页与遍历 – From，Size，Search After & Scroll API**

**From / Size**

```
POST tmdb/_search
{
  "from": 10000,
  "size": 1,
  "query": {
    "match_all": {

    }
  }
}
```

**Search After 避免深度分页的问题**

```
POST users/_search
{
    "size": 1,
    "query": {
        "match_all": {}
    },
    "search_after":
        [
          13,
          "Z30JVHUBaxLQOLM-KE5P"],
    "sort": [
        {"age": "desc"} ,
        {"_id": "asc"}    
    ]
}
```

**Scroll API**

调⽤ Scroll API, 创建⼀个快照

```
POST /users/_search?scroll=5m
{
    "size": 1,
    "query": {
        "match_all" : {
        }
    }
}
```

```
POST /_search/scroll
{
    "scroll" : "1m",
    "scroll_id" : "FGluY2x1ZGVfY29udGV4dF91dWlkDXF1ZXJ5QW5kRmV0Y2gBFDBuczRWSFVCUzd6ekR6aElZWHN6AAAAAAAADX4WbFRGRDZxQkNRNDYweGRXOFhGZ2FJZw=="
}
```


### **6、处理并发读写操作**


**内部版本控制: `If_seq_no + If_primary_term`**

```
PUT products/_doc/1?if_seq_no=0&if_primary_term=1
{
  "title":"iphone",
  "count":100
}
```

**使⽤外部版本： version + `version_type`=external** (使⽤其他数据库作为主要数据存储)

```
PUT products/_doc/1?version=30000&version_type=external
{
  "title":"iphone",
  "count":100
}
```