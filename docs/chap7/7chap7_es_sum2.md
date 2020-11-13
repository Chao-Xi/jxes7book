# **第八节 第二部分总结回顾**

## **1、回顾总结:搜索与算分**

### **1-1 结构化搜索与⾮结构化搜索**

* Term 查询和基于全⽂本 Match 搜索的区别
* 对于需要做精确匹配的字段，需要做聚合分析的字段，字段类型设置为 Keyword

### **1-2 Query Context v.s Filter Context**

* Filter Context 可以避免算分，并且利用缓存
* Bool 查询中 Filter 和 Must Not 都属于 Filter Context

### **1-3 搜索的算分**

* TF-IDF / 字段 Boosting

### **1-4 单字符串多字段查询:`multi-match`**

*  `Best_Field` / `Most_Fields` / `Cross_Field`

### **1-5 提⾼搜索的相关性**

* 多语⾔:设置⼦字段和不同的分词器提升搜索的效果
* Search Template 分离代码逻辑和搜索 DSL
*  多测试，监控及分析⽤户的搜索语句和搜索效果

## **2、聚合 / 分⻚**

### **2-1 聚合**

Bucket / Metric / Pipeline

### **2-2 分⻚**

* From & Size / Search After / Scroll API
* 要避免深度分⻚，对于数据导出等操作，可以使⽤ Scroll API

## **3、Elasticsearch 的分布式模型**

### **3-1 ⽂档的分布式存储**

⽂档通过 hash 算法， route 并存储到相应的分⽚

### **3-2 分⽚及其内部的⼯作机制**

Segment / Transaction Log / Refresh / Merge

### **3-3 分布式查询和聚合分析的内部机制**

* `Query Then Fetch`： IDF 不是基于全局，⽽是基于分⽚计算，**因此，数据量少的时候，算分不不准**
* 增加 `“shard_size”` 可以提⾼ Terms 聚合的精准度


## **4、 数据建模及重要性**

### **4-1 数据建模**

*  ES 如何处理管理关系 / 数据建模的常⻅步骤 / 建模的最佳实践

### **4-2 建模相关的⼯具**

* Index Template / Dynamic Template / Ingest Node / Update By Query / Reindex / Index Alias

### **4-3 最佳实践**

* 避免过多的字段 / 避免 wildcard 查询 / 在 Mapping 中设置合适的字段

## **5、测试**

### **5-1 match query**

```
POST test/_search
{
  "profile": "true",
  "query": {
    "match": {
      "content": "Hello World"
    }
  }
}

POST test/_search
{
  "profile": "true",
  "query": {
    "match": {
      "content": "hello world"
    }
  }
}
```

***Output:***

```
 "max_score" : 0.5753642,
    "hits" : [
      {
        "_index" : "test",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 0.5753642,
        "_source" : {
          "content" : "Hello World"
        }
      }
    ]
  },
  "profile" : {
    "shards" : [
      {
        "id" : "[lTFD6qBCQ460xdW8XFgaIg][test][0]",
        "searches" : [
          {
            "query" : [
              {
                "type" : "BooleanQuery",
                "description" : "content:hello content:world",
...
```

*  `match query` 对context进行分词，standard 分词器，分成`content:hello content:world` 


### **5-2 content.keyword**

```
POST test/_search
{
  "profile": "true",
  "query": {
    "match": {
      "content.keyword": "Hello World"
    }
  }
}
```

***Output: 200***

```
"max_score" : 0.2876821,
    "hits" : [
      {
        "_index" : "test",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 0.2876821,
        "_source" : {
          "content" : "Hello World"
        }
      }
    ]
  },
  "profile" : {
    "shards" : [
      {
        "id" : "[lTFD6qBCQ460xdW8XFgaIg][test][0]",
        "searches" : [
          {
            "query" : [
              {
                "type" : "TermQuery",
                "description" : "content.keyword:Hello World",
```

**自动转化成 `"type" : "TermQuery"`**

```
POST test/_search
{
  "profile": "true",
  "query": {
    "match": {
      "content.keyword": "hello world"
    }
  }
}
```



**output:  `"value" : 0`**

**原因：TermQuery 不会做分词处理**

```
"hits" : {
    "total" : {
      "value" : 0,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "profile" : {
    "shards" : [
      {
        "id" : "[lTFD6qBCQ460xdW8XFgaIg][test][0]",
        "searches" : [
          {
            "query" : [
              {
                "type" : "TermQuery",
                "description" : "content.keyword:hello world",
```

### **5-3  TermQuery**

```
POST test/_search
{
  "profile": "true",
  "query": {
    "term": {
      "content": "Hello World"
    }
  }
}
```

***Output:***

```
 "hits" : {
    "total" : {
      "value" : 0,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "profile" : {
    "shards" : [
      {
        "id" : "[lTFD6qBCQ460xdW8XFgaIg][test][0]",
        "searches" : [
          {
            "query" : [
              {
                "type" : "TermQuery",
                "description" : "content:Hello World",
```

**output:  `"value" : 0`**

原因：

* TermQuery 不会做分词处理
* **`"description" : "content:Hello World",`**
* **Context 里面是 `"content:hello content:world"`**


```
POST test/_search
{
  "profile": "true",
  "query": {
    "term": {
      "content": "hello world"
    }
  }
}
```

***Output:***

```
  "hits" : [ ]
  },
  "profile" : {
    "shards" : [
      {
        "id" : "[lTFD6qBCQ460xdW8XFgaIg][test][0]",
        "searches" : [
          {
            "query" : [
              {
                "type" : "TermQuery",
                "description" : "content:hello world",
```

**output:  `"value" : 0`**

原因：

* TermQuery 不会做分词处理
* **`"description" : "content:hello world",`**
* **Context 里面是 `"content:hello content:world"`**


### **5-3 `TermQuery content.keyword`**

```
POST test/_search
{
  "profile": "true",
  "query": {
    "term": {
      "content.keyword": "Hello World"
    }
  }
}
```

***Output***

```
"hits" : [
      {
        "_index" : "test",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 0.2876821,
        "_source" : {
          "content" : "Hello World"
        }
      }
    ]
  },
  "profile" : {
    "shards" : [
      {
        "id" : "[lTFD6qBCQ460xdW8XFgaIg][test][0]",
        "searches" : [
          {
            "query" : [
              {
                "type" : "TermQuery",
                "description" : "content.keyword:Hello World",
```

## **6、测试**

1. 判断题:⽣生产环境中，对索引使⽤ Index Alias 是⼀个好的实践 **(Yes)**
2. 在 Terms 聚合分析中，有哪些⽅法可以**提⾼查询的精准度**
	*  **数据量少，主分片设置为1**
	*  **数据量大，查询设置`shard_size`**
3. 如何通过聚合分析知道，每天⽹网站中的访客来⾃多少不同的 IP
	* **Cardinality: 类似 SQL 中的 Distinct**
4. 请描述 `“multi_match`” 查询中 `“best_field”`的⾏为
	* **返回单字符串在多个字段算分最高的那个字段。作为算分的结果返回**
5. 对搜索结果分⻚时，所采⽤的两个参数
	* **From / Size** 
6. 判断题:使⽤  Scroll API 导出数据时，即使中途有新的数据写⼊，这些数据也能被导出 **(No)**
	* **Scroll API 导出数据建立快照，新数据插入不会影响快照**