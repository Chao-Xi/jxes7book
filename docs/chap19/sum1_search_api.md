# **ElasticSearch URI 常用语句总结篇**

### **1、ElasticSearch 基本概念**

**1-1 索引**

```
GET movies/_doc/1

GET movies/_settings
# Setting定义不同的数据分布

GET movies/_mapping
# Mapping定义文档字段的类型
```

**1-2 CAT Index API**

```
#查看索引的文档总数
GET kibana_sample_data_ecommerce/_count

#查看前10条文档，了解文档格式
POST kibana_sample_data_ecommerce/_search
{
}

#_cat indices API
#查看indices
GET /_cat/indices/kibana*?v&s=index

#查看状态为绿的索引
GET /_cat/indices?v&health=green

#按照文档个数排序
GET /_cat/indices?v&s=docs.count:desc

#查看具体的字段
GET /_cat/indices/kibana*?pri&v&h=health,index,pri,rep,docs.count,mt

#How much memory is used per index?
GET /_cat/indices?v&h=i,tm&s=tm:desc
```

**1-2 CAT Cluster/Shards API**

```
GET _cluster/health

GET _cat/nodes?v
GET /_nodes/es79,es7_02
GET /_cat/nodes?v
GET /_cat/nodes?v&h=id,ip,port,v,m

GET _cluster/health
GET _cluster/health?level=shards
GET /_cluster/health/kibana_sample_data_ecommerce,kibana_sample_data_flights
GET /_cluster/health/kibana_sample_data_flights?level=shards

#### cluster state The cluster state API allows access to metadata representing the state of the whole cluster. This includes information such as
GET /_cluster/state

#cluster get settings
GET /_cluster/settings
GET /_cluster/settings?include_defaults=true

GET _cat/shards
GET _cat/shards?h=index,shard,prirep,state,unassigned.reason
```

### **2、文档的基本CRUD与批量操作 & 读取＆Bulk API**

**2-1 Create 一个文档**

```
POST users/_doc
{
    "user" : "Mike",
    "post_date" : "2019-04-15T14:12:12",
    "message" : "trying out Kibana"
}

# create document. 指定Id。如果id已经存在，报错

PUT users/_doc/1?op_type=create
{
    "user" : "Jack",
    "post_date" : "2019-05-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
# sucess 

# create document. 指定 ID 如果已经存在，就报错
PUT users/_create/1
{
     "user" : "Jack",
    "post_date" : "2019-05-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```

**2-2 Get 一个文档**

```
GET users/_doc/1
```

**2-3 Index文档**

```
#Update 指定 ID  (先删除，在写入)
PUT users/_doc/1
{
    "user" : "Mike"
}
```

**2-4 Update 文档**

```
# Updete方法不会删除原来的文档而是实现真正的数据更新
POST users/_update/1/
{
    "doc":{
        "post_date" : "2019-05-15T14:12:12",
        "message" : "trying out Elasticsearch"
    }
}
```

**2-5 删除文档**

```
### Delete by Id

DELETE users/_doc/1
```


**2-6  Bulk API**

```
#执行第1次
POST _bulk
{ "index" : { "_index" : "test", "_id" : "1" } }
{ "field1" : "value1" }
{ "delete" : { "_index" : "test", "_id" : "2" } }
{ "create" : { "_index" : "test2", "_id" : "3" } }
{ "field1" : "value3" }
{ "update" : {"_id" : "1", "_index" : "test"} }
{ "doc" : {"field2" : "value2"} }
```

**2-7  批量读取一mget**

```
GET /_mget
{
    "docs" : [
        {
            "_index" : "test",
            "_id" : "1"
        },
        {
            "_index" : "test",
            "_id" : "2"
        }
    ]
}
```

**2-8 批量查询一msearch**

```
GET /_mget
{
    "docs" : [
        {
            "_index" : "test",
            "_id" : "1",
            "_source" : false
        },
        {
            "_index" : "test",
            "_id" : "2",
            "_source" : ["field3", "field4"]
        },
        {
            "_index" : "test",
            "_id" : "3",
            "_source" : {
                "include": ["user"],
                "exclude": ["user.location"]
            }
        }
    ]
}
```

```
POST kibana_sample_data_ecommerce/_msearch
{}
{"query" : {"match_all" : {}},"size":1}
{"index" : "kibana_sample_data_flights"}
{"query" : {"match_all" : {}},"size":2}
```

###  **``**

**3-1 使用`_analyzer API`**

```
GET _analyze
{
  "analyzer": "standard",
  "text": "2 running Quick brown-foxes leap over lazy dogs in the summer evening."
}
```

**3-2 Simple Analyzer**

```
#simpe
GET _analyze
{
  "analyzer": "simple",
  "text": "2 running Quick brown-foxes leap over lazy dogs in the summer evening."
}
```

**3-3 White Analyzer**

```
#whitespace
GET _analyze
{
  "analyzer": "whitespace",
  "text": "2 running Quick brown-foxes leap over lazy dogs in the summer evening."
}
```

**3-4 Stop Analyzer**

```
GET _analyze
{
  "analyzer": "stop",
  "text": "2 running Quick brown-foxes leap over lazy dogs in the summer evening."
}
```

**3-5 Keyword Analyzer**

```
#keyword
GET _analyze
{
  "analyzer": "keyword",
  "text": "2 running Quick brown-foxes leap over lazy dogs in the summer evening."
}
```

**3-6 Pattern Analyzer**

```
GET _analyze
{
  "analyzer": "pattern",
  "text": "2 running Quick brown-foxes leap over lazy dogs in the summer evening."
}
```


**3-7 Language Analyzer**

```
#english
GET _analyze
{
  "analyzer": "english",
  "text": "2 running Quick brown-foxes leap over lazy dogs in the summer evening."
}
```


###  **4、ElasticSearch Search语句**

```
GET kibana_sample_data_ecommerce/_search?q=customer_first_name:Eddie'

GET /_all/_search?q=customer_first_name:Eddie
```

**4-1 Request Body**

```
#REQUEST Body
POST kibana_sample_data_ecommerce/_search
{
    "profile": true,
    "query": {
        "match_all": {}
    }
}
```

**4-2 URI Search详解**

```
#基本查询
GET /movies/_search?q=2012&df=title&sort=year:desc&from=0&size=10&timeout=1s
```

```
GET /movies/_search?q=2012&df=title
{
    "profile":"true"
}
```

```
#泛查询，正对_all,所有字段
GET /movies/_search?q=2012
{
    "profile":"true"
}
```

```
#指定字段
GET /movies/_search?q=title:2012&sort=year:desc&from=0&size=10&timeout=1s
{
    "profile":"true"
}
```

```
# 泛查询
GET /movies/_search?q=title:2012
{
    "profile":"true"
}
```

**Term vs Phrase**

* `Beautifual Mind`等效于`Beautiful OR Mind`
* `"Beautifual Mind"`等效于`Beautiful AND Mind`, Phrase查询还要求前后顺序保待一致

```
# 查找美丽心灵, Mind为泛查询
GET /movies/_search?q=title:Beautiful Mind
{
    "profile":"true"
}
```

**分组与引号**

* `title:(Beautiful AND Mind)`
* `title="Beautifual Mind"`

```
#使用引号，Phrase查询
GET /movies/_search?q=title:"Beautiful Mind"
{
    "profile":"true"
}
```

```
#分组，Bool查询
GET /movies/_search?q=title:(Beautiful Mind)
{
    "profile":"true"
}
```

**布尔操作**

* `AND/OR/NOT` 或者 `&& / || / !`
* 必须大写
* `title:(matrix NOT reloaded)`

```
#布尔操作符
GET /movies/_search?q=title:(Beautiful AND Mind)
{
    "profile":"true"
}

# 查找美丽心灵
GET /movies/_search?q=title:(Beautiful NOT Mind)
{
    "profile":"true"
}

# 查找美丽心灵
GET /movies/_search?q=title:(Beautiful %2BMind)
{
    "profile":"true"
}
```

**分组**

* 表示must
* 表示`must_not`
* `title:(+matrix -reloaded)`

**范围查询**

* 区间表示：`[] 闭区间,{}开区间`
* 算数符号

```
#范围查询 ,区间写法
GET /movies/_search?q=title:beautiful AND year:[2002 TO 2018%7D
{
    "profile":"true"
}

```
**通配符查询（通配符查询效率低, 占用内存大,不建议使用。特别是放在最前面）**

* ?代表1个宇符，＊代表0或多个宇符

```
#通配符查询
GET /movies/_search?q=title:b*
{
    "profile":"true"
}
```

* 正则表达: `title:[bt]oy`
* 模糊匹配与近似查询
	* `title:befutifl~1`
	* `title:"lord rings"~2`

```
//模糊匹配&近似度匹配
GET /movies/_search?q=title:beautifl~1
{
    "profile":"true"
}
```

```
GET /movies/_search?q=title:"Lord Rings"~2
{
    "profile":"true"
}
```

**4-3 Request Body & Query DSL介绍**

**Request Body Search:**

```
#查询movies分页
POST /movies,404_idx/_search?ignore_unavailable=true
{
  "profile": true,
    "query": {
        "match_all": {}
    }
}
```

**分页:**

```
POST /kibana_sample_data_ecommerce/_search
{
  "from":10,
  "size":20,
  "query":{
    "match_all": {}
  }
}
```

**排序**

```
#对日期排序
POST kibana_sample_data_ecommerce/_search
{
  "sort":[{"order_date":"desc"}],
  "query":{
    "match_all": {}
  }

}
```

**`_source filerting`**

* 如果 `_source`没有存储 ，那就只返回匹配的文档的元数据
* `_source`支持使用通配符， `_source["name*","desc*"]`

```
#source filtering
POST kibana_sample_data_ecommerce/_search
{
  "_source":["order_date"],
  "query":{
    "match_all": {}
  }
}
```

**脚本字段**

```
#脚本字段
GET kibana_sample_data_ecommerce/_search
{
  "script_fields": {
    "new_field": {
      "script": {
        "lang": "painless",
        "source": "doc['order_date'].value+'hello'"
      }
    }
  },
  "query": {
    "match_all": {}
  }
}
```

**使用查询表达式一Match**

```
POST movies/_search
{
  "query": {
    "match": {
      "title": {
        "query": "last christmas",
        "operator": "and"
      }
    }
  }
}
```

```
POST movies/_search
{
  "query": {
    "match_phrase": {
      "title":{
        "query": "one love"

      }
    }
  }
}
```

**4-4 Query String&Simple Query String查询**

* query 功能强大易出错
* `query_string` 功能简单灵活性低
* `simple_query_string` 功能简单灵活性低

**Query String Query: 类似URI Query**

```
POST users/_search
{
  "query": {
    "query_string": {
      "default_field": "name",
      "query": "Xi AND Jacob"
    }
  }
}
```

```
POST users/_search
{
  "query": {
    "query_string": {
      "fields":["name","about"],
      "query": "(Xi AND Jacob) OR (Java AND Elasticsearch)"
    }
  }
}
```

**Simple Query String Query**

* 类似`Query String`，但是会忽略错误的语法， 同时只支持部分查询语法
* 不支持`AND OR NOT`，会当作字符串处理
* `Term`之间默认的关系是`OR`，可以指定 `Operator`

```
#Simple Query 默认的operator是 Or
POST users/_search
{
  "query": {
    "simple_query_string": {
      "query": "Xi AND Jacob",
      "fields": ["name"]
    }
  }
}
```

```
POST users/_search
{
  "query": {
    "simple_query_string": {
      "query": "Xi Jacob",
      "fields": ["name"],
      "default_operator": "AND"
    }
  }
}
```

**Single field Query String**

```
GET /movies/_search
{
    "profile": true,
    "query":{
        "query_string":{
            "default_field": "title",
            "query": "Beafiful AND Mind"
        }
    }
}
```

**Multiple Simple Query String**

```
# 多fields
GET /movies/_search
{
    "profile": true,
    "query":{
        "query_string":{
            "fields":[
                "title",
                "year"
            ],
            "query": "2012"
        }
    }
}
```

**Simple Query String**

```
GET /movies/_search
{
    "profile":true,
    "query":{
        "simple_query_string":{
            "query":"Beautiful +mind",
            "fields":["title"]
        }
    }
}
```

### **5、ElasticSearch Mapping 介绍**

```
#写入文档，查看 Mapping
PUT mapping_test/_doc/1
{
  "firstName":"Chan",
  "lastName": "Jackie",
  "loginDate":"2018-07-24T10:29:48.103Z"
}
```

```
GET mapping_test/_mapping

DELETE mapping_test

# dynamic mapping，推断字段的类型
PUT mapping_test/_doc/1
{
    "uid" : "123",
    "isVip" : false,
    "isAdmin": "true",
    "age":19,
    "heigh":180
}
```

**1、控制Dynamic Mappings**

```
# 修改为false

PUT movies
{
	"mappings": {
		"_doc": {
			"dynamic" : "false"
		}
	}
}

# 新增 anotherField

PUT dynamic_mapping_test/_doc/10
{
  "anotherField":"someValue"
}

# "dynamic": false: 可以插入，无法被搜索到

POST dynamic_mapping_test/_search
{
  "query":{
    "match":{
      "anotherField":"someValue"
    }
  }
}

# 修改为strict
PUT dynamic_mapping_test/_mapping
{
  "dynamic": "strict"
}


```

**2、显式Mapping设置与常见参数介绍**

```
#设置 Mobile index 为 false
DELETE users


PUT users
{
    "mappings" : {
      "properties" : {
        "firstName" : {
          "type" : "text"
        },
        "lastName" : {
          "type" : "text"
        },
        "mobile" : {
          "type" : "text",
          "index": false
        }
      }
    }
}
```

*  "index": false


**3、Index Options**

* docs——记录doc id
* freqs——记录doc id和term frequencies
* positions一记录doc id / term frequencies / term position
* offsets 一 doc id/term frequencies/term posistion/character offects

**4、`null_values`**

```
"null_value": "NULL"
```

**5、`copy_to` 设置**

* `copy_to`将字段的数值拷贝到目标字段，实现类似`_all`的作用

```
PUT users
{
  "mappings": {
    "properties": {
      "firstName":{
        "type": "text",
        "copy_to": "fullName"
      },
      "lastName":{
        "type": "text",
        "copy_to": "fullName"
      }
    }
  }
}
```

```
PUT users/_doc/1
{
  "firstName":"Jacob",
  "lastName": "Xi"
}
```

Check:

```
GET users/_search?q=fullName:(Jacob Xi)

POST users/_search
{
  "query": {
    "match": {
       "fullName":{
        "query": "Jacob Xi",
        "operator": "and"
      }
    }
  }
}
```

**6、数组类型**

```
PUT users/_doc/1
{
  "name":"twobirds",
  "interests":["reading","music"]
}
```

```
POST users/_search
{
  "query": {
        "match_all": {}
    }
}
```

**多字段特性及Mapping中配置自定义Analyzer**

**1、多字段类型**

* 增加一个keyword字段

**2、Exact Values不需要被分词**

**3、自定义分词**

* Character Filter
* Tokenizer
* Token Filter

**4、 Character Filters**

在Tokenizer之前对文本进行处理，例如增加删除及替换字符。可以配置多个Character Filters。会影响Tokenizer的`position`和`offset`信息

* HTML strip —— 去除html标签

```
POST _analyze
{
  "tokenizer":"keyword",
  "char_filter":["html_strip"],
  "text": "<b>hello world</b>"
}
```

* Mapping —— 字符串替换

```
#使用char filter进行替换
POST _analyze
{
  "tokenizer": "standard",
  "char_filter": [
      {
        "type" : "mapping",
        "mappings" : [ "- => _"]
      }
    ],
  "text": "123-456, I-test! test-990 650-555-1234"
}
```

* Pattern replace —— 正则匹配替换

```
//正则表达式
GET _analyze
{
  "tokenizer": "standard",
  "char_filter": [
      {
        "type" : "pattern_replace",
        "pattern" : "http://(.*)",
        "replacement" : "$1"
      }
    ],
    "text" : "http://www.elastic.co"
}
```

**5、Tokenizer**

* 将原始的文本按照一定的规则，切分为词（term or token)
* Elasticsearch内置的Tokenizers
	* whitespace/standard/`uax_url_email`/pattern/keyword/path hierarchy

```
// white space and snowball
GET _analyze
{
  "tokenizer": "whitespace",
  "filter": ["stop","snowball"],
  "text": ["The gilrs in China are playing this game!"]
}
```

```
// whitespace与stop
GET _analyze
{
  "tokenizer": "whitespace",
  "filter": ["stop","snowball"],
  "text": ["The rain in Spain falls mainly on the plain."]
}
```

```
// path hierarchy

POST _analyze
{
  "tokenizer":"path_hierarchy",
  "text":"/user/ymruan/a/b/c/d/e"
}

```

**6、Token Filters**

* 将Tokenizer输出的单词（term)，进行增加，修改，删除
* 自带的Token Filters
	* Lowercase/stop/synonym（添加近义词）

```
//remove 加入lowercase后，The被当成 stopword删除
GET _analyze
{
  "tokenizer": "whitespace",
  "filter": ["lowercase","stop","snowball"],
  "text": ["The gilrs in China are playing this game!"]
}
```

### **6、Index Template和Dynamic Template**

* Index Templates —— 帮助你设定Mappings和Settings，并按照一定的规则自动匹配到新创建的索引之上
* 模版仅在一个索引被新创建时，才会产生作用。修改模版不会影响已创建的索引

```
PUT template/_doc/1
{
    "someNumber":"1",
    "someDate":"2019/01/01"
}
```

**设定Mappings和Settings**

```
#Create a default template
PUT _template/template_default
{
  "index_patterns": ["*"],
  "order" : 0,
  "version": 1,
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas":1
  }
}
```
```
PUT /_template/template_test
{
    "index_patterns" : ["test*"],
    "order" : 1,
    "settings" : {
        "number_of_shards": 1,
        "number_of_replicas" : 2
    },
    "mappings" : {
        "date_detection": false,
        "numeric_detection": true
    }
}
```

*  `"date_detection": false,`
*   `"numeric_detection": true`

查看template信息

```
#查看template信息
GET /_template/template_default
GET /_template/temp*
```

```
PUT testtemplate/_doc/1
{
    "someNumber":"1",
    "someDate":"2019/01/01"
}
```

**Dynamic Template**

根据Elasticsearch识别的数据类型，结合字段名称，来动态设定字段类型

* 所有的字符串类型都设定成Keyword，或者关闭keyword字段
* `is`开头的字段都设置成`boolean`
* `long_`开头的都设置成`long`类型

* Dynamic Tempate

	* Dynamic Tempate是定义在在某个索引的Mapping中
	* Template有一个名称
	* 匹配规则是一个数组
	* 为匹配到字段设置Mapping

*  匹配规则参数

	* `match_mapping_type`：匹配自动识别的字段类型 如string, boolean等
	* `match`, `unmatch`：匹配字段名
	* `path_match`,`path_unmatch`

```
PUT my_index
{
  "mappings": {
    "dynamic_templates": [
            {
        "strings_as_boolean": {
          "match_mapping_type":   "string",
          "match":"is*",
          "mapping": {
            "type": "boolean"
          }
        }
      },
      {
        "strings_as_keywords": {
          "match_mapping_type":   "string",
          "mapping": {
            "type": "keyword"
          }
        }
      }
    ]
  }
}
```

* match, unmatch：匹配字段名

```
#结合路径
PUT my_index
{
  "mappings": {
    "dynamic_templates": [
      {
        "full_name": {
          "path_match":   "name.*",
          "path_unmatch": "*.middle",
          "mapping": {
            "type":       "text",
            "copy_to":    "full_name"
          }
        }
      }
    ]
  }
}
```

```
PUT my_index/_doc/1
{
  "name": {
    "first":  "John",
    "middle": "Winston",
    "last":   "Lennon"
  }
}
```

```
GET my_index/_search?q=full_name:John
```

### **7、Elasticsearch Aggregations聚合分析简介**

* Bucket

```
#按照目的地进行分桶统计
GET kibana_sample_data_flights/_search
{
    "size": 0,
    "aggs":{
        "flight_dest":{
            "terms":{
                "field":"DestCountry"
            }
        }
    }
}
```

* Metrics

```
#查看航班目的地的统计信息，增加平均，最高最低价格
GET kibana_sample_data_flights/_search
{
    "size": 0,
    "aggs":{
        "flight_dest":{
            "terms":{
                "field":"DestCountry"
            },
            "aggs":{
                "avg_price":{
                    "avg":{
                        "field":"AvgTicketPrice"
                    }
                },
                "max_price":{
                    "max":{
                        "field":"AvgTicketPrice"
                    }
                },
                "min_price":{
                    "min":{
                        "field":"AvgTicketPrice"
                    }
                }
            }
        }
    }
}
```

* 嵌套

```
#价格统计信息+天气信息
GET kibana_sample_data_flights/_search
{
    "size": 0,
    "aggs":{
        "flight_dest":{
            "terms":{
                "field":"DestCountry"
            },
            "aggs":{
                "stats_price":{
                    "stats":{
                        "field":"AvgTicketPrice"
                    }
                },
                "wather":{
                  "terms": {
                    "field": "DestWeather",
                    "size": 5
                  }
                }

            }
        }
    }
}
```





