# **深入聚合分析**

### **1、Bucket & Metric 聚合分析及嵌套聚合**

**Metric Aggregation**

* 单值分析:只输出⼀个分析结果
	* min, max, avg, sum
	* Cardinality (类似 distinct Count)
* 多值分析:输出多个分析结果
	*  stats, extended stats
	*  percentile, percentile rank
	*  top hits (排在前⾯的示例)

```
# Metric 聚合，找到最低的工资
POST employees/_search
{
  "size": 0,
  "aggs": {
    "min_salary": {
      "min": {
        "field":"salary"
      }
    }
  }
}
```
**Metric 聚合，找到最高的工资**

```
# Metric 聚合，找到最高的工资
POST employees/_search
{
  "size": 0,
  "aggs": {
    "max_salary": {
      "max": {
        "field":"salary"
      }
    }
  }
}
```

**多个 Metric 聚合，找到最低最高和平均工资**

```
POST employees/_search
{
  "size": 0,
  "aggs": {
    "max_salary": {
      "max": {
        "field": "salary"
      }
    },
    "min_salary": {
      "min": {
        "field": "salary"
      }
    },
    "avg_salary": {
      "avg": {
        "field": "salary"
      }
    }
  }
}
```

**一个聚合，输出多值**

```
POST employees/_search
{
  "size": 0,
  "aggs": {
    "stats_salary": {
      "stats": {
        "field":"salary"
      }
    }
  }
}
```

**Bucket**

* Terms
* 数字类型
	*  Range / Data Range
	*  Histogram / Date Histogram

**Terms Aggregation**

* 字段需要打开 fielddata，才能进行 Terms Aggregation
* `Keyword` 默认支持 `doc_values`
* Text 需要在 Mapping 中 enable。会按照分词后的结果进⾏分类

```
POST employees/_search
{
  "size": 0,
  "aggs": {
    "jobs": {
      "terms": {
        "field":"job.keyword"
      }
    }
  }
}
```

**对 Text 字段打开 fielddata，支持terms aggregation**

```
PUT employees/_mapping
{
  "properties" : {
    "job":{
       "type":     "text",
       "fielddata": true
    }
  }
}
```

```
POST employees/_search
{
  "size": 0,
  "aggs": {
    "jobs": {
      "terms": {
        "field":"job"
      }
    }
  }
 }
```

**对`job.keyword` 和 `job` 进行 `terms` 聚合，分桶的总数并不一样**

```
POST employees/_search
{
  "size": 0,
  "aggs": {
    "jobs": {
      "terms": {
        "field":"job.keyword"
      }
    }
  }
}
```

**Cardinality**

类似 SQL 中的 Distinct

```
POST employees/_search
{
  "size": 0,
  "aggs": {
    "cardinate": {
      "cardinality": {
        "field": "job"
      }
    }
  }
}
```

```
POST employees/_search
{
  "size": 0,
  "aggs": {
    "cardinate": {
      "cardinality": {
        "field": "job.keyword"
      }
    }
  }
}
```

**Bucket Size & Top Hits Demo**

```
# 对 性别的 keyword 进行聚合
POST employees/_search
{
  "size": 0,
  "aggs": {
    "gender": {
      "terms": {
        "field":"gender"
      }
    }
  }
}
```

**指定 bucket 的 size**

```
POST employees/_search
{
  "size": 0,
  "aggs": {
    "ages_5": {
      "terms": {
        "field":"age",
        "size":3
      }
    }
  }
}
```

**Range & Histogram 聚合**

```
#Salary Ranges 分桶，可以自己定义 key
POST employees/_search
{
  "size": 0,
  "aggs": {
    "salary_range": {
      "range": {
        "field":"salary",
        "ranges":[
          {
            "to":10000
          },
          {
            "from":10000,
            "to":20000
          },
          {
            "key":">20000",
            "from":20000
          }
        ]
      }
    }
  }
}
```

**Bucket + Metric Aggregation**

```
# 嵌套聚合1，按照工作类型分桶，并统计工资信息

 POST employees/_search
{
  "size": 0,
  "aggs": {
    "Job_salary_stats": {
      "terms": {
        "field": "job.keyword"
      },
      "aggs": {
        "salary": {
          "stats": {
            "field": "salary"
          }
        }
      }
    }
  }
}
```

### **2、Pipeline 聚合分析**

`Pipeline aggregation` —— 对分组+聚合继续再做分组+聚合。

**Pipeline**

`Pipeline:min_bucket`

* 管道的概念: ⽀持聚合分析的结果，再次进⾏聚合分析
	* Pipeline 的分析结果会输出到原结果中，根据位置的不同，分为两类
	* Sibling - 结果和现有分析结果同级
	* Max，min，Avg & Sum Bucket
	* Stats，Extended Status Bucket
	* Percentiles Bucket
* Parent - 结果内嵌到现有的聚合分析结果之中
	* Derivative (求导)
	* Cumultive Sum (累计求和)
	* Moving Function (滑动窗⼝)


```
# 平均工资最低的工作类型
POST employees/_search
{
  "size": 0,
  "aggs": {
    "jobs": {
      "terms": {
        "field": "job.keyword",
        "size": 10
      },
      "aggs": {
        "avg_salary": {
          "avg": {
            "field": "salary"
          }
        }
      }
    },
    "min_salary_by_job":{
      "min_bucket": {
        "buckets_path": "jobs>avg_salary"
      }
    }
  }
}
```

**`max_bucket`**

```
# 平均工资最高的工作类型
POST employees/_search
{
  "size": 0,
  "aggs": {
    "jobs": {
      "terms": {
        "field": "job.keyword",
        "size": 10
      },
      "aggs": {
        "avg_salary": {
          "avg": {
            "field": "salary"
          }
        }
      }
    },
    "max_salary_by_job":{
      "max_bucket": {
        "buckets_path": "jobs>avg_salary"
      }
    }
  }
}
```

**`stats_bucket`**

```
# 平均工资的统计分析
POST employees/_search
{
  "size": 0,
  "aggs": {
    "jobs": {
      "terms": {
        "field": "job.keyword",
        "size": 10
      },
      "aggs": {
        "avg_salary": {
          "avg": {
            "field": "salary"
          }
        }
      }
    },
    "stats_salary_by_job":{
      "stats_bucket": {
        "buckets_path": "jobs>avg_salary"
      }
    }
  }
}
```

**`percentiles_bucket`**

```
# 平均工资的百分位数
POST employees/_search
{
  "size": 0,
  "aggs": {
    "jobs": {
      "terms": {
        "field": "job.keyword",
        "size": 10
      },
      "aggs": {
        "avg_salary": {
          "avg": {
            "field": "salary"
          }
        }
      }
    },
    "percentiles_salary_by_job":{
      "percentiles_bucket": {
        "buckets_path": "jobs>avg_salary"
      }
    }
  }
}
```


**Parent Pipeline: Derivative**

* Cumulative Sum
* Moving Function

```
#按照年龄对平均工资求导
POST employees/_search
{
  "size": 0,
  "aggs": {
    "age": {
      "histogram": {
        "field": "age",
        "min_doc_count": 0,
        "interval": 1
      },
      "aggs": {
        "avg_salary": {
          "avg": {
            "field": "salary"
          }
        },
        "derivative_avg_salary":{
          "derivative": {
            "buckets_path": "avg_salary"
          }
        }
      }
    }
  }
}
```

**`Cumulative_sum`**

```
#Cumulative_sum
POST employees/_search
{
  "size": 0,
  "aggs": {
    "age": {
      "histogram": {
        "field": "age",
        "min_doc_count": 0,
        "interval": 1
      },
      "aggs": {
        "avg_salary": {
          "avg": {
            "field": "salary"
          }
        },
        "cumulative_salary":{
          "cumulative_sum": {
            "buckets_path": "avg_salary"
          }
        }
      }
    }
  }
}
```

**`moving_fn`**

```
#Moving Function
POST employees/_search
{
  "size": 0,
  "aggs": {
    "age": {
      "histogram": {
        "field": "age",
        "min_doc_count": 0,
        "interval": 1
      },
      "aggs": {
        "avg_salary": {
          "avg": {
            "field": "salary"
          }
        },
        "moving_avg_salary":{
          "moving_fn": {
            "buckets_path": "avg_salary",
            "window":10,
            "script": "MovingFunctions.min(values)"
          }
        }
      }
    }
  }
}
```

### **3、聚合的作⽤用范围及排序**

**同时 ES 还⽀持以下⽅式改变聚合的作⽤范围**

* Filter
* `Post_Filter`
* Global

```
# Query
POST employees/_search
{
  "size": 0,
  "query": {
    "range": {
      "age": {
        "gte": 20
      }
    }
  },
  "aggs": {
    "jobs": {
      "terms": {
        "field":"job.keyword"

      }
    }
  }
}
```

**Filter**

```
#Filter
POST employees/_search
{
  "size": 0,
  "aggs": {
    "older_person": {
      "filter":{
        "range":{
          "age":{
            "from":35
          }
        }
      },
      "aggs":{
         "jobs":{
           "terms": {
        "field":"job.keyword"
      }
      }
    }},
    "all_jobs": {
      "terms": {
        "field":"job.keyword"

      }
    }
  }
}
```

**`Post_Filter`**

是对聚合分析后的⽂档进⾏再次过滤

```
#Post field. 一条语句，找出所有的job类型。还能找到聚合后符合条件的结果
POST employees/_search
{
  "aggs": {
    "jobs": {
      "terms": {
        "field": "job.keyword"
      }
    }
  },
  "post_filter": {
    "match": {
      "job.keyword": "Dev Manager"
    }
  }
}
```

**global**

```
#global
POST employees/_search
{
  "size": 0,
  "query": {
    "range": {
      "age": {
        "gte": 40
      }
    }
  },
  "aggs": {
    "jobs": {
      "terms": {
        "field":"job.keyword"

      }
    },

    "all":{
      "global":{},
      "aggs":{
        "salary_avg":{
          "avg":{
            "field":"salary"
          }
        }
      }
    }
  }
}
```

**排序**

* 指定 order， 按照 count 和 key 进⾏排序
	* 默认情况，按照 count 降序排序
	* 指定 size，就能返回相应的桶

```
#排序 order
#count and key
POST employees/_search
{
  "size": 0,
  "query": {
    "range": {
      "age": {
        "gte": 20
      }
    }
  },
  "aggs": {
    "jobs": {
      "terms": {
        "field":"job.keyword",
        "order":[
          {"_count":"asc"},
          {"_key":"desc"}
          ]

      }
    }
  }
}
```

**基于⼦聚合的值排序**

```
#排序 order
#count and key
POST employees/_search
{
  "size": 0,
  "aggs": {
    "jobs": {
      "terms": {
        "field":"job.keyword",
        "order":[  {
            "stats_salary.min":"desc"
          }]


      },
    "aggs": {
      "stats_salary": {
        "stats": {
          "field":"salary"
        }
      }
    }
    }
  }
}
```

### **4、 聚合的精准度问题**

**如何解决 Terms 不准的问题:提升 shard_size 的参数**

*  聚合分析不准的原因，数据分散在多个分片上， Coordinating Node ⽆法获取数据全
	* 解决⽅案 1:当数据量不大时，设置 Primary Shard 为 1; 实现准确性
	* 解决方案 2:在分布式数据上，设置 `shard_size` 参数，提⾼精确度


**打开 `show_term_doc_count_error`**

调整 shard size ⼤小，降低 `doc_count_error_upper_bound `来提升准确度

**增加整体计算量量，提⾼了准确度，但会降低相应时间**

Shard Size 默认⼤小设定: **`shard size = size *1.5 +10`**

**因为从ES7开始，默认的primary shard 为一，所以不会出现 Terms 不准的问题**



**提高 shard size ⼤小，降低 `doc_count_error_upper_bound` 来提升准确度**

**"shard_size":1**

```
GET my_flights/_search
{
  "size": 0,
  "aggs": {
    "weather": {
      "terms": {
        "field":"OriginWeather",
        "size":1,
        "shard_size":1,
        "show_term_doc_count_error":true
      }
    }
  }
}
```

```
GET my_flights/_search
{
  "size": 0,
  "aggs": {
    "weather": {
      "terms": {
        "field":"OriginWeather",
        "size":1,
        "shard_size":10,
        "show_term_doc_count_error":true
      }
    }
  }
}
```

* size是最终返回多少个buckt的数量。
* `shard_size`是每个bucket在一个shard上取回的bucket的总数。 然后，每个shard上的结果，会在coordinate节点上在做一次汇总，返回总数。
