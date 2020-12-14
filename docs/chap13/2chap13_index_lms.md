# **第二节 索引全生命周期管理及工具介绍**

## **1、时间序列的索引**

### **1-1 特点**

索引中的数据随着时间，持续不断增长

### **1-2 按照时间序列划分索引的好处 & 挑战**

* **按照时间进行划分索引，会使得管理更加简单**。例如，完整删除一个索引，性能比 `delete by query` 好
* **如何进行自动化管理，减少人工操作**
	* **从 Hot 移动到 Warm**
	* **定期关闭或者删除索引**

## **2、索引生命周期常见的阶段**

![Alt Image Text](../images/chap13_2_1.png "Body image")

* **Hot**: 	索引还存在着大量的读写操作
* **Warm**:		**索引不存在写操作**，还有被查询的需要 
* **Cold**:	**数据不存在写操作，读操作也不多**
* **Delete**:		**索引不再需要**，可以被安全删除

## **3、Elasticsearch Curator**

* Elastic 官方推出的工具
	* 基于 python 的命令行工具
* **配置 Actions**
	* 内置 10 多种 Index 相关的操作
	* 每个动作可以顺序执行
* **Filters**
	* 支持各种条件，过滤出需要操作的索引

[https://www.elastic.co/guide/en/elasticsearch/client/curator/current/index.html](https://www.elastic.co/guide/en/elasticsearch/client/curator/current/index.html)

![Alt Image Text](../images/chap13_2_2.png "Body image")

## **4、eBay Lifecycle Management Tool**

![Alt Image Text](../images/chap13_2_3.png "Body image")

* eBay Pronto team 自研图形化工具
	*  支持 Curator 的功能
	*  一个界面，管理多个 ES 集群
	*  支持不同的 ES 版本
* 支持图形化配置
* Job 定时触发
* 系统高可用


## **5、工具比较**

![Alt Image Text](../images/chap13_2_4.png "Body image")

## **6、Index Lifecycle Management**

**Elasticsearch 6.6 推出的新功能**

*  基于 X-Pack Basic License，可免费使用

**ILM概念**

* Policy
* Phase
* Action

## **7、ILM Policy**

* 集群中支持定义多个 Policy
* 每个索引可以使用相同或不相同的 Policy

![Alt Image Text](../images/chap13_2_5.png "Body image")

## **8、Index Lifecycle Policies 图形化界面**

* 通过 Kibana Management 设定
* Hot phase 是必须要的
	* 可以 enable rollover
* 其他 Phase 按需设定
* Watch-history-ilm policy
	* 创建 7 天后自动删除

![Alt Image Text](../images/chap13_2_6.png "Body image")

## **9、Live Demo**

* 将ILM刷新时间设定为1秒，默认10分钟
* 设置Hot/Warm/Cold和Delete四个阶段
	* 超过 5 个文档以后 rollover
	* 10 秒后进入 warm
	* 15 秒后进入 Cold
	* 20 秒后删除索引

```
# 运行三个节点，分片 将box_type设置成 hot，warm和cold
# 具体参考 github下，docker-hot-warm-cold 下的docker-compose 文件
```

[启动hot-warm-cold集群](https://chao-xi.github.io/jxes7book/chap12/4chap12_es_red_yellow/#6-1-hot-warm-cold)

```
# 设置 1秒刷新1次，生产环境10分种刷新一次
PUT _cluster/settings
{
  "persistent": {
    "indices.lifecycle.poll_interval":"1s"
  }
}
```
```
GET _cluster/settings

### output
{
  "persistent" : {
    "indices" : {
      "lifecycle" : {
        "poll_interval" : "1s"
      }
    }
  },
  "transient" : { }
}
```

```
# 设置 Policy
PUT /_ilm/policy/log_ilm_policy
{
  "policy": {
    "phases": {
      "hot": {
        "actions": {
          "rollover": {
            "max_docs": 5
          }
        }
      },
      "warm": {
        "min_age": "10s",
        "actions": {
          "allocate": {
            "include": {
              "box_type": "warm"
            }
          }
        }
      },
      "cold": {
        "min_age": "15s",
        "actions": {
          "allocate": {
            "include": {
              "box_type": "cold"
            }
          }
        }
      },
      "delete": {
        "min_age": "20s",
        "actions": {
          "delete": {}
        }
      }
    }
  }
}
```

```
# 设置索引模版
PUT /_template/log_ilm_template
{
  "index_patterns" : [
      "ilm_index-*"
    ],
    "settings" : {
      "index" : {
        "lifecycle" : {
          "name" : "log_ilm_policy",
          "rollover_alias" : "ilm_alias"
        },
        "routing" : {
          "allocation" : {
            "include" : {
              "box_type" : "hot"
            }
          }
        },
        "number_of_shards" : "1",
        "number_of_replicas" : "0"
      }
    },
    "mappings" : { },
    "aliases" : { }
}
```

```
PUT ilm_index-000001
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0,
    "index.lifecycle.name": "log_ilm_policy",
    "index.lifecycle.rollover_alias": "ilm_alias",
    "index.routing.allocation.include.box_type":"hot"
  },
  "aliases": {
    "ilm_alias": {
      "is_write_index": true
    }
  }
}
```

***Output:***

```
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "ilm_index-000001"
}
```



```
# 对 Alias写入文档
POST  ilm_alias/_doc
{
  "dfd":"dfdsf"
}
```

***Output:***

```
{
  "_index" : "ilm_index-000001",
  "_type" : "_doc",
  "_id" : "9PpPVXYB_xXR0dDRfje1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}
```

```
GET _cat/shards/ilm_index-000001
```

```
GET _cat/shards/ilm_index-000001
```

***Output***

```
...
{
  "error" : {
    "root_cause" : [
      {
        "type" : "index_not_found_exception",
        "reason" : "no such index [ilm_index-000001]",
        "resource.type" : "index_or_alias",
        "resource.id" : "ilm_index-000001",
        "index_uuid" : "_na_",
        "index" : "ilm_index-000001"
      }
    ],
    "type" : "index_not_found_exception",
    "reason" : "no such index [ilm_index-000001]",
    "resource.type" : "index_or_alias",
    "resource.id" : "ilm_index-000001",
    "index_uuid" : "_na_",
    "index" : "ilm_index-000001"
  },
  "status" : 404
}
```


 