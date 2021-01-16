# **Elasticsearch 深入搜索总结**

### **1、基于词项和基于全文的搜索**

* **如果不需要算分，可以通过Constant Score，将查询转为Filtering**
* 范围查询和Date Math
* 使用Exist查询处理非空Null值
* 精确值＆多值字段的精确值查找
	* Term查询是包含，不是完全相等。针对多值字段查询要尤其注意



**1-1、基于Term的查询**

* Term是表达语意的最小单位。搜索和利用统计语言模型进行自然语言处理都需要处理Term
* Term Level Query:Term Query／Range Query／Exists Query／Prefix Query/Wildcard Query

```
POST /products/_bulk
{ "index": { "_id": 1 }}
{ "productID" : "XHDK-A-1293-#fJ3","desc":"iPhone" }
{ "index": { "_id": 2 }}
{ "productID" : "KDKE-B-9947-#kL5","desc":"iPad" }
{ "index": { "_id": 3 }}
{ "productID" : "JODL-X-1937-#pV7","desc":"MBP" }
```

```
POST /products/_search
{
  "query": {
    "term": {
      "desc": {
        "value": "iphone"   
      }
    }
  }
}
```

```
POST /products/_search
{
  "query": {
    "term": {
      "productID": {
        "value": "xhdk-a-1293-#fJ3"
      }
    }
  }
}
```

**多字段Mapping和Term查询**

```
POST /products/_search
{
  "query": {
    "term": {
      "productID.keyword": {
        "value": "XHDK-A-1293-#fJ3"
      }
    }
  }
}
```


**复合查询一Constant Score转为Filter**

* 将Query转成Filter，忽略TF-IDF计算，避免相关性算分的开销
* Filter可以有效利用缓存

```
POST /products/_search
{
  "explain": true,
  "query": {
    "constant_score": {
      "filter": {
        "term": {
          "productID.keyword": "XHDK-A-1293-#fJ3"
        }
      }
    }
  }
}
```

**1-2 基于全文的查询**

* 基于全文的查询
	* Match Query/Match Phrase Query/Query String Query

**Match Query Result**

```
POST movies/_search
{
  "query": {
    "match": {
      "title": {
        "query": "Matrix reloaded"
      }
    }
  }
}
```

**Operator**

```
POST movies/_search
{ 
  "profile": true,
  "query": {
    "match": {
      "title": {
        "query": "Matrix reloaded",
        "operator": "AND"
      }
    }
  }
}
```

**`Minimum_should_match`**

```
POST movies/_search
{ 
  "profile": true,
  "query": {
    "match": {
      "title": {
        "query": "Matrix reloaded",
        "minimum_should_match": 2
      }
    }
  }
}
```

**Match Phrase Query**

```
POST movies/_search
{ 
  "profile": true,
  "query": {
    "match_phrase": {
      "title": {
        "query": "Matrix reloaded",
        "slop": 1
      }
    }
  }
}
```


### **2、结构化搜索以及搜索的相关性算分**

**1、ES中的结构化搜索**

* 布尔，时间，日期和数字这类结构化数据：
* 结构化的文本可以做精确匹配或者部分匹配
* 结构化结果只有“是”或“否”两个值


* 对布尔值 match 查询，有算分

```
POST products/_search
{
  "query": {
    "term": {
      "avaliable": {
        "value": "false"
      }
    }
  }
}


POST products/_search
{
  "profile": "true",
  "explain": true,
  "query": {
    "term": {
      "avaliable": true
    }
  }
}
```

对布尔值，通过	`constant score` 转成 `filtering`，没有算分

```
POST products/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "term": {
          "avaliable": "false"
        }
      }
    }
  }
}
```

* 数字Range
	* gt大于
	* It小于
	* gte大于等于
	* lte 小于等于

```
# 数字类型 Term
POST products/_search
{
  "profile": "true",
  "explain": true,
  "query": {
    "term": {
      "price": 30
    }
  }
}
```

*  日期 range

```
POST products/_search
{
    "query" : {
        "constant_score" : {
            "filter" : {
                "range" : {
                    "date" : {
                      "gte" : "now-2y"
                    }
                }
            }
        }
    }
}
```

* 处理空值

```
#exists查询
POST products/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "exists": {
          "field": "date"
        }
      }
    }
  }
}
```

```
# "bool": { "must_not" }

POST products/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "bool": {
          "must_not": {
            "exists": {
              "field": "date"
            }
          }
        }
      }
    }
  }
}
```

* 处理多值字段，term 查询是包含，而不是等于

```
POST movies/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "term": {
          "genre.keyword": "Comedy"
        }
      }
    }
  }
}
```

* 查找多个精确值

```
#数字类型 terms
POST products/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "terms": {
          "price": [
            "20",
            "30"
          ]
        }
      }
    }
  }
}
```

* 字符类型 terms

```
#字符类型 terms
POST products/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "terms": {
          "productID.keyword": [
            "QQPX-R-3956-#aD8",
            "JODL-X-1937-#pV7"
          ]
        }
      }
    }
  }
}
```

*  Term VS Match

```
POST products/_search
{
  "profile": "true",
  "explain": true,
  "query": {
    "constant_score": {
      "filter": {
        "term": {
          "productID.keyword": "XHDK-A-1293-#fJ3"
        }
      }
    }
  }
}

POST products/_search
{
  "profile": "true",
  "explain": true,
  "query": {
    "term": {
      "productID.keyword": "XHDK-A-1293-#fJ3"
    }
  }
}
```

```
POST products/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "term": {
          "avaliable": "false"
        }
      }
    }
  }
}

POST products/_search
{
  "query": {
    "term": {
      "avaliable": {
        "value": "false"
      }
    }
  }
}
```

**2、搜索的相关性算分**

* 相关性和相关性算分
* 词频TF
* 逆文档频率IDF
* TF-IDF 的概念
* Boosting Relvance

```
POST testscore/_search
{
    "query": {
        "boosting" : {
            "positive" : {
                "term" : {
                    "content" : "elasticsearch"
                }
            },
            "negative" : {
                 "term" : {
                     "content" : "like"
                }
            },
            "negative_boost" : 0.2
        }
    }
}
```

### **3、 Query & Filtering与多字符串多字段查询**

Elasticsearch中，有Query和Filter两种不同 Context

* **Query Context：相关性算分**
* **Filter Context: 不需要算分（Yes or No) ，可以利用Cache，获得更好的性能**

* **must / should => count score**
* **`must_not` / filter => not count score**

**bool查询**

```
#基本语法
POST /products/_search
{
  "query": {
    "bool" : {
      "must" : {
        "term" : { "price" : "30" }
      },
      "filter": {
        "term" : { "avaliable" : "true" }
      },
      "must_not" : {
        "range" : {
          "price" : { "lte" : 10 }
        }
      },
      "should" : [
        { "term" : { "productID.keyword" : "JODL-X-1937-#pV7" } },
        { "term" : { "productID.keyword" : "XHDK-A-1293-#fJ3" } }
      ],
      "minimum_should_match" :1
    }
  }
}
```

如何解决结构化查询一“包含而不是相等”的问题

```
POST /movies/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "term": {
            "genre.keyword": "Comedy"
        }
      }
    }
  }
}
```

增加count字段，使用bool查询解决

```
# must，有算分: "_score" : 1.2111092,
POST /newmovies/_search
{
  "query": {
    "bool": {
      "must": [
        {"term": {"genre.keyword": {"value": "Comedy"}}},
        {"term": {"genre_count": {"value": 1}}}

      ]
    }
  }
 }
```

```
#Filter。不参与算分，结果的score是0
POST /newmovies/_search
{
  "query": {
    "bool": {
      "filter": [
        {"term": {"genre.keyword": {"value": "Comedy"}}},
        {"term": {"genre_count": {"value": 1}}}
        ]

    }
  }
}
```

**Filter Context一不影响算分**

```
#Filtering Context
POST _search
{
  "query": {
    "bool" : {

      "filter": {
        "term" : { "avaliable" : "true" }
      },
      "must_not" : {
        "range" : {
          "price" : { "lte" : 10 }
        }
      }
    }
  }
}
```

**Query Context一影响算分**

```
POST /products/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "term": {
            "productID.keyword": {
              "value": "JODL-X-1937-#pV7"}}
        },
        {"term": {"avaliable": {"value": true}}
        }
      ]
    }
  }
}
```

**Bool嵌套 实现了should not的逻辑**

```
POST /products/_search
{
  "query": {
    "bool": {
      "must": {
        "term": {
          "price": "30"
        }
      },
      "should": [
        {
          "bool": {
            "must_not": {
              "term": {
                "avaliable": "false"
              }
            }
          }
        }
      ],
      "minimum_should_match": 1
    }
  }
}
```

**Controll the Precision**

```
POST _search
{
  "query": {
    "bool" : {
      "must" : {
        "term" : { "price" : "30" }
      },
      "filter": {
        "term" : { "avaliable" : "true" }
      },
      "must_not" : {
        "range" : {
          "price" : { "lte" : 10 }
        }
      },
      "should" : [
        { "term" : { "productID.keyword" : "JODL-X-1937-#pV7" } },
        { "term" : { "productID.keyword" : "XHDK-A-1293-#fJ3" } }
      ],
      "minimum_should_match" :2
    }
  }
}
```

**查询语句的结构，会对相关度算分产生影响**

```
POST /animals/_search
{
  "query": {
    "bool": {
      "should": [
        { "term": { "text": "brown" }},
        { "term": { "text": "red" }},
        { "term": { "text": "quick"   }},
        { "term": { "text": "dog"   }}
      ]
    }
  }
}
```

```
POST /animals/_search
{
  "query": {
    "bool": {
      "should": [
        { "term": { "text": "quick" }},
        { "term": { "text": "dog"   }},
        {
          "bool":{
            "should":[
               { "term": { "text": "brown" }},
                 { "term": { "text": "brown" }},
            ]
          }

        }
      ]
    }
  }
```

* 同一层级下的竞争字段，具有有相同的权重
* 通过嵌套bool查询，可以改变对算分的影响

**控制字段的Boosting**

```
POST blogs/_search
{
  "query": {
    "bool": {
      "should": [
        {"match": {
          "title": {
            "query": "apple,ipad",
            "boost": 4
          }
        }},

        {"match": {
          "content": {
            "query": "apple,ipad",
            "boost": 1
          }
        }}
      ]
    }
  }
}
```

**Not Quite Not**

```
POST news/_search
{
  "query": {
    "bool": {
      "must": {
        "match":{"content":"apple"}
      }
    }
  }
}
```

**Boosting Query**

```
POST news/_search
{
  "query": {
    "bool": {
      "must": {
        "match":{"content":"apple"}
      },
      "must_not": {
        "match":{"content":"pie"}
      }
    }
  }
```

**Negative Boost**

```
POST news/_search
{
  "query": {
    "boosting": {
      "positive": {
        "match": {
          "content": "apple"
        }
      },
      "negative": {
        "match": {
          "content": "pie"
        }
      },
      "negative_boost": 0.5
    }
  }
}
```

### **4、单字符串多字段查询**

```
PUT /blogs/_doc/1
{
    "title": "Quick brown rabbits",
    "body":  "Brown rabbits are commonly seen."
}

PUT /blogs/_doc/2
{
    "title": "Keeping pets healthy",
    "body":  "My quick brown fox eats rabbits on a regular basis."
}
```

**单字符串多字段查询 Dis Max Query**

算分过程

* 查询should语句中的两个查询
* 加和两个查询的评分
* 乘以匹配语句的总数
* 除以所有语句的总数

```
POST /blogs/_search
{
    "query": {
        "bool": {
            "should": [
                { "match": { "title": "Brown fox" }},
                { "match": { "body":  "Brown fox" }}
            ]
        }
    }
}
```

**Disjunction Max Query查询**

* 不应该将分数简单全加而是应该找到单个最佳匹配的字段的评分
* 将任何与任一查询匹配的文档作为结果返回。采用字段上最匹配的评分最终评分返回

```
POST blogs/_search
{
    "query": {
        "dis_max": {
            "queries": [
                { "match": { "title": "Brown fox" }},
                { "match": { "body":  "Brown fox" }}
            ]
        }
    }
}
```

**通过Tie Breaker参数调整**

```
POST blogs/_search
{
    "query": {
        "dis_max": {
            "queries": [
                { "match": { "title": "Quick pets" }},
                { "match": { "body":  "Quick pets" }}
            ],
            "tie_breaker": 0.2
        }
    }
}
```

**Tier Breaker是一个介于0-1之间的浮点数。0代表使用最佳匹配；1代表所有语句同等重要.**

**单字符串多字段查询：Multi Match**

三种场景

* 最佳字段（Best Fields)
* 多数字段（Most Fields)
* 混合字段（Cross Field)

**Multi Match Query**

* Best Fields是默认类型可以不用指定
* Minimum should match等参数可以传递到生成的query中

```
POST blogs/_search
{
    "query": {
        "dis_max": {
            "queries": [
                { "match": { "title": "Quick pets" }},
                { "match": { "body":  "Quick pets" }}
            ],
            "tie_breaker": 0.2
        }
    }
}
```

**使用多数字段匹配解决**


用广度匹配字段title包括尽可能多的文档——以提升召回率——同时又使用字段title.std作为信号将相关度更高的文档置于结果顶部。

```
GET /titles/_search
{
   "query": {
        "multi_match": {
            "query":  "barking dogs",
            "type":   "most_fields",
            "fields": [ "title", "title.std" ]
        }
    }
}
```

* 每个字段对于最终评分的贡献可以通过自定义值boost来控制。比如，使title字段更为重要， 这样同时也降低了其他信号字段的作用：

```
GET /titles/_search
{
   "query": {
        "multi_match": {
            "query":  "barking dogs",
            "type":   "most_fields",
            "fields": [ "title^10", "title.std" ]
        }
    }
}
```

**跨字段搜索**

* 无法使用Operator
* 可以用`copy_to`解决，但是需要额外的存储空间

```
post address/_search
{
    "query": {
        "multi_match": {
            "query" :  "Poland Street W1V",
            "type" :   "most_fields",
            "fields" : [ "street", "city", "coutry",  "poscode"]
        }
    }
}
```

**Cross Field**

```
post address/_search
{
    "query": {
        "multi_match": {
            "query" :  "Poland Street W1V",
            "type" :   "cross_fields",
            "operator": "and", 
            "fields" : [ "street", "city", "coutry",  "poscode"]
        }
    }
}
```

**5、多语言及中文分词与检索**

```
POST _analyze
{
  "analyzer": "hanlp_standard",
  "text": ["剑桥分析公司多位高管对卧底记者说，他们确保了唐纳德·特朗普在总统大选中获胜"]
} 
```

```
#Pinyin
PUT /artists/
{
    "settings" : {
        "analysis" : {
            "analyzer" : {
                "user_name_analyzer" : {
                    "tokenizer" : "whitespace",
                    "filter" : "pinyin_first_letter_and_full_pinyin_filter"
                }
            },
            "filter" : {
                "pinyin_first_letter_and_full_pinyin_filter" : {
                    "type" : "pinyin",
                    "keep_first_letter" : true,
                    "keep_full_pinyin" : false,
                    "keep_none_chinese" : true,
                    "keep_original" : false,
                    "limit_first_letter_length" : 16,
                    "lowercase" : true,
                    "trim_whitespace" : true,
                    "keep_none_chinese_in_first_letter" : true
                }
            }
        }
    }
}


GET /artists/_analyze
{
  "text": ["刘德华 张学友 郭富城 黎明 四大天王"],
  "analyzer": "user_name_analyzer"
}
```

### **6、使⽤ Search Template 和 Index Alias**

**Search Template – 解耦程序 & 搜索 DSL**

* 搜索⼯程师：

```
POST _scripts/tmdb
{
  "script": {
    "lang": "mustache",
    "source": {
      "_source": [
        "title","overview"
      ],
      "size": 20,
      "query": {
        "multi_match": {
          "query": "{{q}}",
          "fields": ["title","overview"]
        }
      }
    }
  }
}
```

**使⽤ Search Template 进行查询**

* 性能工程师

```
POST tmdb/_search/template
{
    "id":"tmdb",
    "params": {
        "q": "basketball with cartoon aliens"
    }
}
```

**Index Alias 实现零停机运维**

```
POST _aliases
{
  "actions": [
    {
      "add": {
        "index": "movies-2019",
        "alias": "movies-latest"
      }
    }
  ]
}
```

```
POST movies-latest/_search
{
  "query": {
    "match_all": {}
  }
}
```

**使用 Alias 创建不同查询的视图**

```
POST _aliases
{
  "actions": [
    {
      "add": {
        "index": "movies-2019",
        "alias": "movies-lastest-highrate",
        "filter": {
          "range": {
            "rating": {
              "gte": 4
            }
          }
        }
      }
    }
  ]
}
```

```
POST movies-lastest-highrate/_search
{
  "query": {
    "match_all": {}
  }
}
```

### **7、 综合排序:Function Score Query 优化算分**

**Function Score Query**

提供了⼏种默认的计算分值的函数
 
* **Weight** :为每⼀个⽂档设置⼀个简单而不被规范化的权重
* **Field Value Factor**:使⽤用该数值来修改 `_score`，例例如将 “热度”和“点赞数”作为算分的参考因素
* **Random Score**:为每⼀个用户使⽤⼀个不同的，随机算分结果
* **衰减函数**: 以某个字段的值为标准，距离某个值越近，得分越⾼
* **Script Score**:自定义脚本完全控制所需逻辑 

**按受欢迎度提升权重**

```
PUT /blogs/_doc/1
{
  "title":   "About popularity",
  "content": "In this post we will talk about...",
  "votes":   0
}
```

```
POST /blogs/_search
{
  "query": {
    "function_score": {
      "query": {
        "multi_match": {
          "query":    "popularity",
          "fields": [ "title", "content" ]
        }
      },
      "field_value_factor": {
        "field": "votes"
      }
    }
  }
}
```

**使⽤ Modifier 平滑曲线**

```
POST /blogs/_search
{
  "query": {
    "function_score": {
      "query": {
        "multi_match": {
          "query":    "popularity",
          "fields": [ "title", "content" ]
        }
      },
      "field_value_factor": {
        "field": "votes",
        "modifier": "log1p"
      }
    }
  }
}
```

**引⼊ Factor**

`新的算分=⽼的算分*log(1+factor*投票数)`

```
POST /blogs/_search
{
  "query": {
    "function_score": {
      "query": {
        "multi_match": {
          "query":    "popularity",
          "fields": [ "title", "content" ]
        }
      },
      "field_value_factor": {
        "field": "votes",
        "modifier": "log1p" ,
        "factor": 0.1
      }
    }
  }
}
```

**Boost Mode 和 Max Boost**

```
POST /blogs/_search
{
  "query": {
    "function_score": {
      "query": {
        "multi_match": {
          "query":    "popularity",
          "fields": [ "title", "content" ]
        }
      },
      "field_value_factor": {
        "field": "votes",
        "modifier": "log1p" ,
        "factor": 0.1
      },
      "boost_mode": "sum",
      "max_boost": 3
    }
  }
}
```

**⼀致性随机函数**

```
POST /blogs/_search
{
  "query": {
    "function_score": {
      "random_score": {
        "seed": 911119
      }
    }
  }
}
```

**8、Term & Phrase Suggester**

Term Suggester

* Suggester 就是一种特殊类型的搜索。”text” ⾥是调⽤时候提供的⽂本，通常来⾃于⽤户界面上⽤用户输⼊的内容


```
PUT articles
{
  "mappings": {
    "properties": {
      "title_completion":{
        "type": "completion"
      }
    }
  }
}
```

```
POST articles/_search?pretty
{
  "size": 0,
  "suggest": {
    "article-suggester": {
      "prefix": "elk ",
      "completion": {
        "field": "title_completion"
      }
    }
  }
}
```

**Suggester – Missing Mode**

几种 Suggestion Mode

* Missing – 如索引中已经存在，就不提供建议
* Popular – 推荐出现频率更加⾼的词
* Always – ⽆论是否存在，都提供建议

**Term Suggester – Popular Mode**

Popular – 推荐出现频率更加⾼的词

```
POST /articles/_search
{

  "suggest": {
    "term-suggestion": {
      "text": "lucen rock",
      "term": {
        "suggest_mode": "popular",
        "field": "body"
      }
    }
  }
}
```

**Term Suggester – Missing Mode**

* Missing – 如索引中已经存在，就不提供建议

```
POST /articles/_search
{
  "size": 1,
  "query": {
    "match": {
      "body": "lucen rock"
    }
  },
  "suggest": {
    "term-suggestion": {
      "text": "lucen rock",
      "term": {
        "suggest_mode": "missing",
        "field": "body"
      }
    }
  }
}
```

**Term Suggester – Always Mode**

* always – ⽆论是否存在，都提供建议

```
POST /articles/_search
{
  "suggest": {
    "term-suggestion": {
      "text": "lucen rock",
      "term": {
        "suggest_mode": "always",
        "field": "body"
      }
    }
  }
}
```

**Sorting by Frequency & Prefix Length**

* 默认按照 score 排序，也可以按照 “frequency”
* 默认⾸首字⺟不⼀致就不会匹配推荐，但是如果将 `prefix_length` 设置为 `0`，就会为 `hock` 建议 `rock`

```
POST /articles/_search
{

  "suggest": {
    "term-suggestion": {
      "text": "lucen hocks",
      "term": {
        "suggest_mode": "always",
        "field": "body",
        "prefix_length":0,
        "sort": "frequency"
      }
    }
  }
}
```

**Phrase Suggester**

* Phrase Suggester 在 Term Suggester 上增加了一些额外的逻辑
* 一些参数
	* Suggest Mode :missing, popular, always
	* Max Errors:最多可以拼错的 Terms 数
	* Confidence:限制返回结果数，默认为 1

```
POST /articles/_search
{
  "suggest": {
    "my-suggestion": {
      "text": "lucne and elasticsear rock hello world ",
      "phrase": {
        "field": "body",
        "max_errors":2,
        "confidence":0,
        "direct_generator":[{
          "field":"body",
          "suggest_mode":"always"
        }],
        "highlight": {
          "pre_tag": "<em>",
          "post_tag": "</em>"
        }
      }
    }
  }
}
```

### **9、⾃动补全与基于上下文的提示**

The Completion Suggester

* Completion Suggester 提供了了“⾃动完成” (Auto Complete) 的功能。⽤户每输⼊入⼀个字符，就需要即时发送⼀个查询请求到后段查找匹配项
* 对性能要求比较苛刻。Elasticsearch 采⽤了不同的**数据结构，并⾮通过倒排索引来完成。 而是将 Analyze 的数据编码成 FST 和索引⼀起存放。FST 会被 ES 整个加载进内存**， 速度很快


**使⽤ Completion Suggester 的一些步骤**

* 定义 Mapping，使⽤ "completion" type
* 索引数据
* 运⾏ "suggest" 查询，得到搜索建议


**索引数据**

```
PUT articles
{
  "mappings": {
    "properties": {
      "title_completion":{
        "type": "completion"
      }
    }
  }
}
```

**搜索数据**

```
POST articles/_search?pretty
{
  "size": 0,
  "suggest": {
    "article-suggester": {
      "prefix": "el ",
      "completion": {
        "field": "title_completion"
      }
    }
  }
}
```

**什么是 Context Suggester**

实现 Context Suggester 的具体步骤

* 定制一个 Mapping
* 索引数据，并且为每个⽂档加⼊ Context 信息
* 结合 Context 进⾏ Suggestion 查询


**定义 Mapping**

增加 Contexts

* Type
* name

```
PUT comments/_mapping
{
  "properties": {
    "comment_autocomplete":{
      "type": "completion",
      "contexts":[{
        "type":"category",
        "name":"comment_category"
      }]
    }
  }
}
```

**索引数据**

```
POST comments/_doc
{
  "comment":"I love the star war movies",
  "comment_autocomplete":{
    "input":["star wars"],
    "contexts":{
      "comment_category":"movies"
    }
  }
}

POST comments/_doc
{
  "comment":"Where can I find a Starbucks",
  "comment_autocomplete":{
    "input":["starbucks"],
    "contexts":{
      "comment_category":"coffee"
    }
  }
}
```

**不同的上下⽂，⾃动提示**

```
POST comments/_search
{
  "suggest": {
    "MY_SUGGESTION": {
      "prefix": "sta",
      "completion":{
        "field":"comment_autocomplete",
        "contexts":{
          "comment_category":"coffee"
        }
      }
    }
  }
}
```

```
POST comments/_search
{
  "suggest": {
    "MY_SUGGESTION": {
      "prefix": "sta",
      "completion":{
        "field":"comment_autocomplete",
        "contexts":{
          "comment_category":"movies"
        }
      }
    }
  }
}
```

### **10、跨集群搜索**

启动3个集群:

```
elasticsearch -E node.name=cluster0node -E cluster.name=cluster0 -E path.data=cluster0_data -E discovery.type=single-node -E http.port=9200 -E transport.port=9300
elasticsearch -E node.name=cluster1node -E cluster.name=cluster1 -E path.data=cluster1_data -E discovery.type=single-node -E http.port=9201 -E transport.port=9301
elasticsearch -E node.name=cluster2node -E cluster.name=cluster2 -E path.data=cluster2_data -E discovery.type=single-node -E http.port=9202 -E transport.port=9302
```

在每个集群上设置动态的设置

```
PUT _cluster/settings
{
  "persistent": {
    "cluster": {
      "remote": {
        "cluster0": {
          "seeds": [
            "127.0.0.1:9300"
          ],
          "transport.ping_schedule": "30s"
        },
        "cluster1": {
          "seeds": [
            "127.0.0.1:9301"
          ],
          "transport.compress": true,
          "skip_unavailable": true
        },
        "cluster2": {
          "seeds": [
            "127.0.0.1:9302"
          ]
        }
      }
    }
  }
```


```
GET /users,cluster1:users,cluster2:users/_search
{
  "query": {
    "range": {
      "age": {
        "gte": 20,
        "lte": 40
      }
    }
  }
}
```

