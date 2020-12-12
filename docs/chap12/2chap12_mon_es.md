# **第二节 监控Elasticsearch集群**

## **1、Elastcsearch Stats相关的API**

**Elasticsearch 提供7多个监控相关的API**

* **Node Stats:** `_nodes/stats` 
* **Cluster Stats:** `_cluster/stats`
* **Index Stats:** `index_name/_stats` 


## **2、Elasticsearch Task API**

* 查看Task相关的API
	* **Pending Cluster Tasks API:** `GET _cluster/pending_tasks`
	* **Task Management API** `GET _tasks`（可以用来Cancel一个Task) 

* **监控Thread Pools** 
	* `GET _nodes/thread_pool` 
	* `GET _nodes/stats/thread_pool` 
	* `GET _cat/thread_pool?v` 
	* `GET _nodes/hot_threads`

 
## **3、The Index & Query Slow Log**

* **支持将分片上，Search 和 Fetch 阶段的慢查询写入文件** 
* **支持为 Query 和 Fetch 分别定义阈值** 
* **索引级的动态设置，可以按需设置，或者通过 Index Template 统一设定** 
* Slow log文件通过`log4j2.properties`配置 
 
![Alt Image Text](../images/chap12_2_1.png "Body image")

### **3-1 stats**

**Node Stats：**

```
# Node Stats：
GET _nodes/stats

{
  "_nodes" : {
    "total" : 2,
    "successful" : 2,
    "failed" : 0
  },
  "cluster_name" : "jx",
  "nodes" : {
    "y6Kvg6woTbCuDNQJjubBHw" : {
      "timestamp" : 1606292261685,
      "name" : "node2",
      "transport_address" : "172.16.72.169:9301",
      "host" : "172.16.72.169",
      "ip" : "172.16.72.169:9301",
      "roles" : [
        "data",
        "ingest",
        "master",
        "ml",
        "remote_cluster_client",
        "transform"
      ],
      "attributes" : {
        "ml.machine_memory" : "3973804032",
        "ml.max_open_jobs" : "20",
        "xpack.installed" : "true",
        "my_rack_id" : "rack1",
        "transform.node" : "true"
      },
...
```

**Cluster Stats:**

```
#Cluster Stats:
GET _cluster/stats

"indices" : {
    "count" : 15,
    "shards" : {
      "total" : 28,
      "primaries" : 16,
      "replication" : 0.75,
      "index" : {
        "shards" : {
          "min" : 1,
          "max" : 2,
          "avg" : 1.8666666666666667
        },
        "primaries" : {
          "min" : 1,
          "max" : 2,
          "avg" : 1.0666666666666667
        },
        "replication" : {
          "min" : 0.0,
          "max" : 1.0,
          "avg" : 0.8
        }
      }
    },
```

**Index Stats:**

```
#Index Stats:
GET kibana_sample_data_ecommerce/_stats
```

### **3-1 Tasks**

**Pending Cluster Tasks API**

```
#Pending Cluster Tasks API:
GET _cluster/pending_tasks

{
  "tasks" : [ ]
}
```

**查看所有的 tasks，也支持 cancel task**

```
# 查看所有的 tasks，也支持 cancel task
GET _tasks

{
  "nodes" : {
    "y6Kvg6woTbCuDNQJjubBHw" : {
      "name" : "node2",
      "transport_address" : "172.16.72.169:9301",
      "host" : "172.16.72.169",
      "ip" : "172.16.72.169:9301",
      "roles" : [
        "data",
        "ingest",
        "master",
        "ml",
        "remote_cluster_client",
        "transform"
      ],
      "attributes" : {
        "ml.machine_memory" : "3973804032",
        "ml.max_open_jobs" : "20",
        "xpack.installed" : "true",
        "my_rack_id" : "rack1",
        "transform.node" : "true"
      },
 ...
```

### **3-3 `thread_pool`**

```
GET _nodes/thread_pool
GET _nodes/stats/thread_pool
```

```
GET _cat/thread_pool?v

node_name name                active queue rejected
node2     analyze                  0     0        0
node2     ccr                      0     0        0
node2     fetch_shard_started      0     0        0
node2     fetch_shard_store        0     0        0
node2     flush                    0     0        0
node2     force_merge              0     0        0
node2     generic                  0     0        0
...
```

```
GET _nodes/hot_threads

::: {node2}{y6Kvg6woTbCuDNQJjubBHw}{uhHvQ8l2RV2m_NATQOkMRA}{172.16.72.169}{172.16.72.169:9301}{dilmrt}{ml.machine_memory=3973804032, ml.max_open_jobs=20, xpack.installed=true, my_rack_id=rack1, transform.node=true}
   Hot threads at 2020-11-25T08:35:08.208Z, interval=500ms, busiestThreads=3, ignoreIdleThreads=true:

::: {node1}{V1WpUCM4QzKEF0CrRG2BYA}{g6rlxUB-RduMoha2VcRdvg}{172.16.72.169}{172.16.72.169:9300}{dilmrt}{ml.machine_memory=3973804032, xpack.installed=true, transform.node=true, ml.max_open_jobs=20, my_rack_id=rack1}
   Hot threads at 2020-11-25T08:35:08.206Z, interval=500ms, busiestThreads=3, ignoreIdleThreads=true:
```

```
GET _nodes/stats/thread_pool
```

### **3-4 Index Slowlogs**

* 设置 Index Slowlogs
* The first 1000 characters of the doc's source will be logged

```
PUT my_index

PUT my_index/_settings
{
  "index.indexing.slowlog":{
    "threshold.index":{
      "warn":"10s",
      "info": "4s",
      "debug":"2s",
      "trace":"0s"
    },
    "level":"trace",
    "source":1000  
  }
}
```

### **3-5 设置查询**

```
# 设置查询
DELETE my_index

# "0" logs all queries
PUT my_index/
{
  "settings": {
    "index.search.slowlog.threshold": {
      "query.warn": "10s",
      "query.info": "3s",
      "query.debug": "2s",
      "query.trace": "0s",
      "fetch.warn": "1s",
      "fetch.info": "600ms",
      "fetch.debug": "400ms",
      "fetch.trace": "0s"
    }
  }
}
```

```
GET my_index

{
  "my_index" : {
    "aliases" : { },
    "mappings" : { },
    "settings" : {
      "index" : {
        "search" : {
          "slowlog" : {
            "threshold" : {
              "fetch" : {
                "warn" : "1s",
                "trace" : "0s",
                "debug" : "400ms",
                "info" : "600ms"
              },
              "query" : {
                "warn" : "10s",
                "trace" : "0s",
                "debug" : "2s",
                "info" : "3s"
              }
            }
          }
        },
        "number_of_shards" : "1",
        "provided_name" : "my_index",
        "creation_date" : "1606293512411",
        "number_of_replicas" : "1",
        "uuid" : "qq59K9bGQomixy36uXYHnA",
        "version" : {
          "created" : "7090299"
        }
      }
    }
  }
}
```
