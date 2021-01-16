# **数据建模**

### **1、对象及 Nested 对象**

* 关系型数据库，⼀般会考虑 Normalize 数据; 
* 在 Elasticsearch，往往考虑 Denormalize 数据


**Denormalize 的好处: 读的速度变快 / ⽆需表连接 / ⽆需行锁**

Elasticsearch 并不擅⻓处理关联关系。我们⼀般采用以下四种⽅法处理关联

* 对象类型
* 嵌套对象(Nested Object)
* ⽗子关联关系(Parent / Child )
* 应⽤端关联

```
# 查询 Blog 信息
POST blog/_search
{
  "query": {
    "bool": {
      "must": [
        {"match": {"content": "Elasticsearch"}},
        {"match": {"user.username": "Jack"}}
      ]
    }
  }
}
```
```
# 查询电影信息
POST my_movies/_search
{
  "query": {
    "bool": {
      "must": [
        {"match": {"actors.first_name": "Keanu"}},
        {"match": {"actors.last_name": "Hopper"}}
      ]
    }
  }

}
```

**Nested 数据类型: 允许对象数组中的对象被独立索引**

```
# 创建 Nested 对象 Mapping
PUT my_movies
{
      "mappings" : {
      "properties" : {
        "actors" : {
          "type": "nested",
          "properties" : {
            "first_name" : {"type" : "keyword"},
            "last_name" : {"type" : "keyword"}
          }},
        "title" : {
          "type" : "text",
          "fields" : {"keyword":{"type":"keyword","ignore_above":256}}
        }
      }
    }
}
```

**嵌套查询**

```
# Nested 查询
POST my_movies/_search
{
  "query": {
    "bool": {
      "must": [
        {"match": {"title": "Speed"}},
        {
          "nested": {
            "path": "actors",
            "query": {
              "bool": {
                "must": [
                  {"match": {
                    "actors.first_name": "Keanu"
                  }},

                  {"match": {
                    "actors.last_name": "Hopper"
                  }}
                ]
              }
            }
          }
        }
      ]
    }
  }
}
```

**嵌套聚合Nested Aggregation**

```
# Nested Aggregation
POST my_movies/_search
{
  "size": 0,
  "aggs": {
    "actors": {
      "nested": {
        "path": "actors"
      },
      "aggs": {
        "actor_name": {
          "terms": {
            "field": "actors.first_name",
            "size": 10
          }
        }
      }
    }
  }
}
```

### **2、文档的父子关系**

**对象和 Nested 对象的局限性： 每次更新，需要重新索引整个对象(包括根对象和嵌套对象)**

* ES 提供了类似关系型数据库中 Join 的实现。使用 Join 数据类型实现，可以通过维护 Parent / Child 的关系，从⽽分离两个对象
	* ⽗文档和⼦文档是两个独⽴的文档
		* 更新⽗文档无需重新索引子文档。
		* ⼦文档被添加，更新或者删除也不会影响到⽗文档和其他的⼦文档

**定义⽗子关系的⼏个步骤**

* 设置索引的 Mapping
* 索引⽗文档
* 索引⼦文档
* 按需查询⽂档

```
# 设定 Parent/Child Mapping
PUT my_blogs
{
  "settings": {
    "number_of_shards": 2
  },
  "mappings": {
    "properties": {
      "blog_comments_relation": {
        "type": "join",
        "relations": {
          "blog": "comment"
        }
      },
      "content": {
        "type": "text"
      },
      "title": {
        "type": "keyword"
      }
    }
  }
}
```

* ⽗文档: blog
* 子文档：comment


插入索引父文档

```
#索引父文档
PUT my_blogs/_doc/blog1
{
  "title":"Learning Elasticsearch",
  "content":"learning ELK @ geektime",
  "blog_comments_relation":{
    "name":"blog"
  }
}
```
插入索引⼦文档

```
#索引子文档
PUT my_blogs/_doc/comment1?routing=blog1
{
  "comment":"I am learning ELK",
  "username":"Jack",
  "blog_comments_relation":{
    "name":"comment",
    "parent":"blog1"
  }
}
```

查询所有文档

```
# 查询所有文档
POST my_blogs/_search
{

}
```

**Parent / Child 所⽀持的查询**

* 查询所有⽂档
* Parent Id 查询
* Has Child 查询
* Has Parent 查询

```
# Has Child 查询,返回父文档
POST my_blogs/_search
{
  "query": {
    "has_child": {
      "type": "comment",
      "query" : {
                "match": {
                    "username" : "Jack"
                }
            }
    }
  }
```

```
# Has Parent 查询，返回相关的子文档
POST my_blogs/_search
{
  "query": {
    "has_parent": {
      "parent_type": "blog",
      "query" : {
                "match": {
                    "title" : "Learning Hadoop"
                }
            }
    }
  }
}
```

```
# Parent Id 查询
POST my_blogs/_search
{
  "query": {
    "parent_id": {
      "type": "comment",
      "id": "blog2"
    }
  }
}
```

```
# Parent Id 查询
POST my_blogs/_search
{
  "query": {
    "parent_id": {
      "type": "comment",
      "id": "blog2"
    }
  }
}
```
```
#更新子文档
PUT my_blogs/_doc/comment3?routing=blog2
{
    "comment": "Hello Hadoop??",
    "blog_comments_relation": {
      "name": "comment",
      "parent": "blog2"
    }
}
```

### **3、Update By Query & Reindex API**

一般在以下⼏种情况时，我们需要重建索引

* **索引的 Mappings 发生变更: 字段类型更改，分词器及字典更新**
* **索引的 Settings 发生变更:索引的主分⽚数发⽣改变**
* **集群内，集群间需要做数据迁移**  

Elasticsearch 的内置提供的 API

* **Update By Query:在现有索引上重建**
* **Reindex:在其他索引上重建索引**

**案例 1:为索引增加子字段**

```
# 写入文档
PUT blogs/_doc/1
{
  "content":"Hadoop is cool",
  "keyword":"hadoop"
}

# 查看 Mapping
GET blogs/_mapping

# 修改 Mapping，增加子字段，使用英文分词器
PUT blogs/_mapping
{
      "properties" : {
        "content" : {
          "type" : "text",
          "fields" : {
            "english" : {
              "type" : "text",
              "analyzer":"english"
            }
          }
        }
      }
    }


# 写入文档
PUT blogs/_doc/2
{
  "content":"Elasticsearch rocks",
    "keyword":"elasticsearch"
}

# 查询新写入文档
POST blogs/_search
{
  "query": {
    "match": {
      "content.english": "Elasticsearch"
    }
  }
}
```

**执⾏ Update By Query**

```
# Update所有文档
POST blogs/_update_by_query
{

}
```

**案例 2:更改已有字段类型的 Mappings**

* ES 不允许在原有 Mapping 上对字段类型进⾏修改
* 只能创建新的索引，并且设定正确的字段类型，再重新导⼊数据

```
PUT blogs/_mapping
{
        "properties" : {
        "content" : {
          "type" : "text",
          "fields" : {
            "english" : {
              "type" : "text",
              "analyzer" : "english"
            }
          }
        },
        "keyword" : {
          "type" : "keyword"
        }
      }
}
```

**"mapper [keyword] cannot be changed from type [text] to [keyword]"**

**Reindex API**

* Reindex API ⽀持把⽂档从⼀个索引拷⻉到另外⼀个索引
* 使⽤ Reindex API 的⼀些场景
	* 修改索引的主分⽚数
	* 改变字段的 Mapping 中的字段类型
	* 集群内数据迁移 / 跨集群的数据迁移

创建新的索引并且设定新的Mapping

```
# 创建新的索引并且设定新的Mapping
PUT blogs_fix/
{
  "mappings": {
        "properties" : {
        "content" : {
          "type" : "text",
          "fields" : {
            "english" : {
              "type" : "text",
              "analyzer" : "english"
            }
          }
        },
        "keyword" : {
          "type" : "keyword"
        }
      }    
  }
}
```

```
# Reindx API
POST  _reindex
{
  "source": {
    "index": "blogs"
  },
  "dest": {
    "index": "blogs_fix"
  }
}
```

**测试 Term Aggregation**

```
# 测试 Term Aggregation
POST blogs_fix/_search
{
  "size": 0,
  "aggs": {
    "blog_keyword": {
      "terms": {
        "field": "keyword",
        "size": 10
      }
    }
  }
}
```

**OP Type**

* `_reindex` 只会创建不存在的⽂档
* ⽂档如果已经存在，会导致版本冲突

```
# Reindx API，version Type Internal
POST  _reindex
{
  "source": {
    "index": "blogs"
  },
  "dest": {
    "index": "blogs_fix",
    "op_type": "create"
  }
}
```

**查看 Task API**

`GET _tasks?detailed=true&actions=*reindex`

* `Reindx API `⽀持异步操作，执⾏只返回Task Id
* `POST _reindex?wait_for_completion=false`

```
# Reindx API，version Type Internal
POST  _reindex
{
  "source": {
    "index": "blogs"
  },
  "dest": {
    "index": "blogs_fix",
    "version_type": "internal"
  }
}
```
```
# Reindx API，version Type external
POST  _reindex
{
  "source": {
    "index": "blogs"
  },
  "dest": {
    "index": "blogs_fix",
    "version_type": "external"
  }
}
```

Output :409 - Conflict

```
# Reindx API，version Type external
POST  _reindex
{
  "source": {
    "index": "blogs"
  },
  "dest": {
    "index": "blogs_fix",
    "version_type": "external"
  },
  "conflicts": "proceed"
}
```

Output: 200 - Conflict

### **4、Ingest Pipeline 与 Painless Script**

**需求:修复与增强写⼊的数据**

Pipeline & Processor

```
#Blog数据，包含3个字段，tags用逗号间隔
PUT tech_blogs/_doc/1
{
  "title":"Introducing big data......",
  "tags":"hadoop,elasticsearch,spark",
  "content":"You konw, for big data"
}

# 测试split tags
POST _ingest/pipeline/_simulate
{
  "pipeline": {
    "description": "to split blog tags",
    "processors": [
      {
        "split": {
          "field": "tags",
          "separator": ","
        }
      }
    ]
  },
  "docs": [
    {
      "_index": "index",
      "_id": "id",
      "_source": {
        "title": "Introducing big data......",
        "tags": "hadoop,elasticsearch,spark",
        "content": "You konw, for big data"
      }
    },
    {
      "_index": "index",
      "_id": "idxx",
      "_source": {
        "title": "Introducing cloud computering",
        "tags": "openstack,k8s",
        "content": "You konw, for cloud"
      }
    }
  ]
}
```

**为⽂档增加字段**

```
#同时为文档，增加一个字段。blog查看量
POST _ingest/pipeline/_simulate
{
  "pipeline": {
    "description": "to split blog tags",
    "processors": [
      {
        "split": {
          "field": "tags",
          "separator": ","
        }
      },

      {
        "set":{
          "field": "views",
          "value": 0
        }
      }
    ]
  },

  "docs": [
    {
      "_index":"index",
      "_id":"id",
      "_source":{
        "title":"Introducing big data......",
  "tags":"hadoop,elasticsearch,spark",
  "content":"You konw, for big data"
      }
    },


    {
      "_index":"index",
      "_id":"idxx",
      "_source":{
        "title":"Introducing cloud computering",
  "tags":"openstack,k8s",
  "content":"You konw, for cloud"
      }
    }

    ]
}
```

**添加 Pipeline 并测试**

```
# 为ES添加一个 Pipeline
PUT _ingest/pipeline/blog_pipeline
{
  "description": "a blog pipeline",
  "processors": [
      {
        "split": {
          "field": "tags",
          "separator": ","
        }
      },

      {
        "set":{
          "field": "views",
          "value": 0
        }
      }
    ]
}
```

```
#查看Pipleline
GET _ingest/pipeline/blog_pipeline
```

**测试pipeline**

```
#测试pipeline
POST _ingest/pipeline/blog_pipeline/_simulate
{
  "docs": [
    {
      "_source": {
        "title": "Introducing cloud computering",
        "tags": "openstack,k8s",
        "content": "You konw, for cloud"
      }
    }
  ]
}
```

**Index & Update By Query**

```
#不使用pipeline更新数据
PUT tech_blogs/_doc/1
{
  "title":"Introducing big data......",
  "tags":"hadoop,elasticsearch,spark",
  "content":"You konw, for big data"
}

#使用pipeline更新数据
PUT tech_blogs/_doc/2?pipeline=blog_pipeline
{
  "title": "Introducing cloud computering",
  "tags": "openstack,k8s",
  "content": "You konw, for cloud"
}
```

```
#查看两条数据，一条被处理，一条未被处理
POST tech_blogs/_search
{}

#update_by_query 会导致错误
POST tech_blogs/_update_by_query?pipeline=blog_pipeline
{
}
Output: 400 - Bad Request
```

**增加`update_by_query`的条件**

```
#增加update_by_query的条件
POST tech_blogs/_update_by_query?pipeline=blog_pipeline
{
    "query": {
        "bool": {
            "must_not": {
                "exists": {
                    "field": "views"
                }
            }
        }
    }
}

Output: 200-ok
```

**Painless 的用途**

案例 1:Script Processor

```
# 增加一个 Script Prcessor
POST _ingest/pipeline/_simulate
{
  "pipeline": {
    "description": "to split blog tags",
    "processors": [
      {
        "split": {
          "field": "tags",
          "separator": ","
        }
      },
      {
        "script": {
          "source": """
          if(ctx.containsKey("content")){
            ctx.content_length = ctx.content.length();
          }else{
            ctx.content_length=0;
          }


          """
        }
      },

      {
        "set":{
          "field": "views",
          "value": 0
        }
      }
    ]
  },

  "docs": [
    {
      "_index":"index",
      "_id":"id",
      "_source":{
        "title":"Introducing big data......",
  "tags":"hadoop,elasticsearch,spark",
  "content":"You konw, for big data"
      }
    },


    {
      "_index":"index",
      "_id":"idxx",
      "_source":{
        "title":"Introducing cloud computering",
  "tags":"openstack,k8s",
  "content":"You konw, for cloud"
      }
    }

    ]
}
```

案例 2:⽂档更新计数

```
DELETE tech_blogs
PUT tech_blogs/_doc/1
{
  "title":"Introducing big data......",
  "tags":"hadoop,elasticsearch,spark",
  "content":"You konw, for big data",
  "views":0
}

POST tech_blogs/_update/1
{
  "script": {
    "source": "ctx._source.views += params.new_views",
    "params": {
      "new_views":100
    }
  }
}

# 查看views计数
POST tech_blogs/_search
{

}

#保存脚本在 Cluster State
POST _scripts/update_views
{
  "script":{
    "lang": "painless",
    "source": "ctx._source.views += params.new_views"
  }
}
```

案例 3:搜索时的 Script 字段

```
GET tech_blogs/_search
{
  "script_fields": {
    "rnd_views": {
      "script": {
        "lang": "painless",
        "source": """
          java.util.Random rnd = new Random();
          doc['views'].value+rnd.nextInt(1000);
        """
      }
    }
  },
  "query": {
    "match_all": {}
  }
}
```

### **5、Elasticsearch 数据建模实例**

```
DELETE books

PUT books
{
      "mappings" : {
      "properties" : {
        "author" : {"type" : "keyword"},
        "cover_url" : {"type" : "keyword","index": false},
        "description" : {"type" : "text"},
        "public_date" : {"type" : "date"},
        "title" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 100
            }
          }
        }
      }
    }
}
```

Cover URL index 设置成false，无法对该字段进行搜索

```
#Cover URL index 设置成false，无法对该字段进行搜索
POST books/_search
{
  "query": {
    "term": {
      "cover_url": {
        "value": "https://images-na.ssl-images-amazon.com/images/I/51OeaMFxcML.jpg"
      }
    }
  }
}

Output: 400 - Bad Request
```

Cover URL index 设置成false，依然支持聚合分析

```
#Cover URL index 设置成false，依然支持聚合分析
POST books/_search
{
  "aggs": {
    "cover": {
      "terms": {
        "field": "cover_url",
        "size": 10
      }
    }
  }
}
```

查询图书:解决字段过⼤引发的性能问题

```
#搜索，通过store 字段显示数据，同时高亮显示 conent的内容
POST books/_search
{
  "stored_fields": ["title","author","public_date"],
  "query": {
    "match": {
      "content": "searching"
    }
  },

  "highlight": {
    "fields": {
      "content":{}
    }
  }
}
```

### **6、Elasticsearch 数据建模最佳实践**

**解决⽅案:Nested Object & Key Value**

使用 Nested 对象，增加key/value

```
DELETE cookie_service

PUT cookie_service
{
  "mappings": {
    "properties": {
      "cookies": {
        "type": "nested",
        "properties": {
          "name": {
            "type": "keyword"
          },
          "dateValue": {
            "type": "date"
          },
          "keywordValue": {
            "type": "keyword"
          },
          "IntValue": {
            "type": "integer"
          }
        }
      },
      "url": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          }
        }
      }
    }
  }
}
```

写入数据，使用key和合适类型的value字段

```
PUT cookie_service/_doc/1
{
 "url":"www.google.com",
 "cookies":[
    {
      "name":"username",
      "keywordValue":"tom"
    },
    {
       "name":"age",
      "intValue":32

    }

   ]
 }


PUT cookie_service/_doc/2
{
 "url":"www.amazon.com",
 "cookies":[
    {
      "name":"login",
      "dateValue":"2019-01-01"
    },
    {
       "name":"email",
      "IntValue":32

    }

   ]
 }
```

**写⼊ & 查询**

Nested 查询，通过bool查询进行过滤

```
POST cookie_service/_search
{
  "query": {
    "nested": {
      "path": "cookies",
      "query": {
        "bool": {
          "filter": [
            {
            "term": {
              "cookies.name": "age"
            }},
            {
              "range":{
                "cookies.intValue":{
                  "gte":30
                }
              }
            }
          ]
        }
      }
    }
  }
}
```
```
# 在Mapping中加入元信息，便于管理
PUT softwares/
{
  "mappings": {
    "_meta": {
      "software_version_mapping": "1.0"
    }
  }
}
```

```
DELETE softwares

# 优化,使用inner object
PUT softwares/
{
  "mappings": {
    "_meta": {
      "software_version_mapping": "1.1"
    },
    "properties": {
      "version": {
        "properties": {
          "display_name": {
            "type": "keyword"
          },
          "hot_fix": {
            "type": "byte"
          },
          "marjor": {
            "type": "byte"
          },
          "minor": {
            "type": "byte"
          }
        }
      }
    }
  }
}
```

```
#通过 Inner Object 写入多个文档
PUT softwares/_doc/1
{
  "version":{
  "display_name":"7.1.0",
  "marjor":7,
  "minor":1,
  "hot_fix":0  
  }

}

PUT softwares/_doc/2
{
  "version":{
  "display_name":"7.2.0",
  "marjor":7,
  "minor":2,
  "hot_fix":0  
  }
}

PUT softwares/_doc/3
{
  "version":{
  "display_name":"7.2.1",
  "marjor":7,
  "minor":2,
  "hot_fix":1  
  }
}
```

```
# 通过 bool 查询，
POST softwares/_search
{
  "query": {
    "bool": {
      "filter": [
        {
          "match":{
            "version.marjor":7
          }
        },
        {
          "match":{
            "version.minor":2
          }
        }

      ]
    }
  }
}
```

**避免空值引起的聚合不准**

```

PUT ratings/_doc/1
{
 "rating":5
}
PUT ratings/_doc/2
{
 "rating":null
}

POST ratings/_search
{
  "size": 0,
  "aggs": {
    "avg": {
      "avg": {
        "field": "rating"
      }
    }
  }
}
```

**Not Null 解决聚合的问题**

```
DELETE ratings

PUT ratings
{
  "mappings": {
      "properties": {
        "rating": {
          "type": "float",
          "null_value": 1.0
        }
      }
    }
}


PUT ratings/_doc/1
{
 "rating":5
}
PUT ratings/_doc/2
{
 "rating":null
}
```

```
POST ratings/_search
{
 "size": 0,
 "aggs": {
   "avg": {
     "avg": {
       "field": "rating"
     }
   }
 }
}
```
