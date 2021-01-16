# **ElasticSearch Operation and Management**

## **1、数据保护**

### **1-1 集群身份认证与用户鉴权**

使用 Security API 创建用户

```
POST /_security/user/jacob
{
	"password" : "paas@word",
	"roles" : ["admin","other_role1"],
	"full_name" : "Jacob Ace",
	"email" : "jacobace@xxx.com",
	"metadata" : {
		"intelligence" : 7
	} 
}
```

**开启并配置 X-Pack 的认证与鉴权**

修改配置文件，打开认证与授权

```
bin/elasticsearch -E node.name=node0 -E cluster.name=geektime -E path.data=node0_data -E http.port=9200 -E xpack.security.enabled=true
```

新版本需要打开SSL： `xpack.security.transport.ssl.enabled=true`

```
elasticsearch -E node.name=node0 -E cluster.name=jx -E path.data=node0_data -E http.port=9200 -E xpack.security.enabled=true -E xpack.security.transport.ssl.enabled=true 
```

* `xpack.security.transport.ssl.enabled=true`
* `xpack.security.enabled=true`

创建默认的用户和分组

```
bin/elasticsearch-setup-passwords interactive
```

### **1-2、集群内部间的安全通信**

生成节点证书

* `bin/elasticsearch-certutil ca`
* `bin/elasticsearch-certutil cert --ca elastic-stack-ca.p12`


将证书拷贝到 `/etc/elasticsearch/certs/` 目录下

```
cd /usr/share/elasticsearch
mkdir /etc/elasticsearch/certs/
cp elastic-stack-ca.p12 elastic-certificates.p12 /etc/elasticsearch/certs/
```

`elasticsearch.yml ` 配置

```
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate

xpack.security.transport.ssl.keystore.path: certs/elastic-certificates.p12
xpack.security.transport.ssl.truststore.path: certs/elastic-certificates.p12
```

```
# ES 启用 https
elasticsearch -E node.name=node0 -E cluster.name=jx -E path.data=node0_data -E http.port=9200 -E xpack.security.enabled=true -E xpack.security.transport.ssl.enabled=true -E xpack.security.transport.ssl.verification_mode=certificate -E xpack.security.transport.ssl.keystore.path=certs/elastic-certificates.p12 -E xpack.security.http.ssl.enabled=true -E xpack.security.http.ssl.keystore.path=certs/elastic-certificates.p12 -E xpack.security.http.ssl.truststore.path=certs/elastic-certificates.p12

elasticsearch -E node.name=node1 -E cluster.name=jx -E path.data=node0_data -E http.port=9201 -E xpack.security.enabled=true -E xpack.security.transport.ssl.enabled=true -E xpack.security.transport.ssl.verification_mode=certificate -E xpack.security.transport.ssl.keystore.path=certs/elastic-certificates.p12 -E xpack.security.http.ssl.enabled=true -E xpack.security.http.ssl.keystore.path=certs/elastic-certificates.p12 -E xpack.security.http.ssl.truststore.path=certs/elastic-certificates.p12
```

* xpack.security.enabled=true
* xpack.security.transport.ssl.enabled=true
* xpack.security.transport.ssl.verification_mode=certificate 
* xpack.security.transport.ssl.keystore.path=certs/elastic-certificates.p12
* `xpack.security.http.ssl.enabled=true -E xpack.security.http.ssl.keystore.path=certs/elastic-certificates.p12`
* `xpack.security.http.ssl.truststore.path=certs/elastic-certificates.p12`


### **1-3 集群与外部间的安全通信**

配置 Elasticsearch for HTTPS

```
elasticsearch -E node.name=node0 -E cluster.name=jx -E path.data=node0_data -E http.port=9200 -E xpack.security.enabled=true -E xpack.security.transport.ssl.enabled=true -E xpack.security.transport.ssl.verification_mode=certificate -E xpack.security.transport.ssl.keystore.path=certs/elastic-certificates.p12 -E xpack.security.http.ssl.enabled=true -E xpack.security.http.ssl.keystore.path=certs/elastic-certificates.p12 -E xpack.security.http.ssl.truststore.path=certs/elastic-certificates.p12 -E xpack.security.authc.anonymous.username=true
```

* xpack.security.authc.anonymous.username=true

配置 Kibana 连接 ES HTTPS

```
sudo vim /usr/share/kibana/config/kibana.yml


elasticsearch.hosts: ["https://localhost:9200"]
elasticsearch.ssl.certificateAuthorities: [ "/usr/share/kibana/config/certs/elastic-ca.pem" ]
elasticsearch.ssl.verificationMode: certificate
```

配置使用 HTTPS 访问 Kibana

```
# 为 Kibna 配置 HTTPS
# 生成后解压，包含了instance.crt 和 instance.key
bin/elasticsearch-certutil ca --pem
sudo cp instance.crt instance.key /usr/share/kibana/config/certs/

sudo vim /usr/share/kibana/config/kibana.yml


server.ssl.enabled: true
server.ssl.certificate: config/certs/instance.crt
server.ssl.key: config/certs/instance.key
```

## **2、水平扩展Elasticsearch集群**

### **2-1 常见的集群部署方式**

**1、节点类型**

* Master eligible / Data / Ingest / Coordinating / Machine Learning
* 建议设置单一角色的节点(dedicated node)


### **2-2 Hot & Warm 架构与 Shard Filtering**

* **Hot 节点(通常使用 SSD):索引有不断有新文档写入。通常使用 SSD**
* **Warm 节点(通常使用 HDD):索引不存在新数据的写入;同时也不存在大量的数据查询**

**标记节点**

```
GET /_cat/nodeattrs?
```

**需要通过 “node.attr” 来标记一个节点**

```
elasticsearch  -E node.name=hotnode -E cluster.name=jx -E path.data=node0_data -E node.attr.my_node_type=hot

elasticsearch  -E node.name=warmnode -E cluster.name=jx -E path.data=node1_data -E node.attr.my_node_type=warm
```

* `node.attr.my_node_type=hot`
* `node.attr.my_node_type=warm`

**配置 Hot 数据**

```
# 配置到 Hot节点
PUT log-2020-11-02
{
  "settings":{
    "number_of_shards":2,
    "number_of_replicas":0,
    "index.routing.allocation.require.my_node_type":"hot"
  }
}
```

```
GET _cat/shards?v

log-2020-11-02                 1     p      STARTED     0    208b 172.16.72.169 hotnode
log-2020-11-02                 0     p      STARTED     0    208b 172.16.72.169 hotnode
```

**旧数据移动到 Warm 节点**

`Index.routing.allocation` 是一个索引级的`dynamic setting`，可以通过 API 在后期进行设定


```
# 配置到 warm 节点
PUT log-2020-11-02/_settings
{  
  "index.routing.allocation.require.my_node_type":"warm"
}

```


**Rack Awareness**

* 当一个机架断电，可能会同时丢失几个节点
* 如果一个索引相同的主分片和副本分片，同时在这个机架上，就有可能导致数据的丢失
* 通过 Rack Awareness 的机制， 就可以尽可能避免将同一个索引的主副分片同时分配在一个机架的节点上


**标记 Rack 节点 + 配置集群**

标记一个 rack 1, rack2

```
elasticsearch  -E node.name=node1 -E cluster.name=jx -E path.data=node1_data -E node.attr.my_rack_id=rack1

elasticsearch  -E node.name=node2 -E cluster.name=jx -E path.data=node2_data -E node.attr.my_rack_id=rack2

PUT _cluster/settings
{
  "persistent": {
    "cluster.routing.allocation.awareness.attributes": "my_rack_id"
  }
}
```

```
PUT my_index1
{
  "settings":{
    "number_of_shards":2,
    "number_of_replicas":1
  }
}

# 分片分到node1, node2
GET _cat/shards?v
my_index1                      1     r      STARTED     0    208b 172.16.72.169 node2
my_index1                      1     p      STARTED     0    208b 172.16.72.169 node1
my_index1                      0     p      STARTED     0    208b 172.16.72.169 node2
my_index1                      0     r      STARTED     0    208b 172.16.72.169 node1
```

**Rack Awareness**

```
# Force awareness
# 标记一个 rack 1
elasticsearch  -E node.name=node1 -E cluster.name=jx -E path.data=node1_data -E node.attr.my_rack_id=rack1

# 标记一个 rack 1
elasticsearch  -E node.name=node2 -E cluster.name=jx -E path.data=node2_data -E node.attr.my_rack_id=rack1
```

* `node.attr.my_rack_id=rack1`

```
PUT _cluster/settings
{
  "persistent": {
    "cluster.routing.allocation.awareness.attributes": "my_rack_id",
    "cluster.routing.allocation.awareness.force.my_rack_id.values": "rack1,rack2"
  }
}
```

```
GET _cluster/settings
```

### **2-3 分片设定及管理**

**1、单个分片**

* 7.0 开始，新创建一个索引时，默认只有一个主分片
* 单个索引，单个分片时候，集群无法实现水平扩展

**2、两个分片**


集群增加一个节点后，Elasticsearch 会自动进行分片的移动，也叫 **Shard Rebalancing**

**3、如何设计分片数**

* 多分片的好处:一个索引如果分布在不同的节点，多个节点可以并行执行

**4、分片过多所带来的副作用**

* Shard 是 Elasticsearch 实现集群水平扩展的最小单位
* 过多设置分片数会带来一些潜在的问题
	* 每个分片是一个 Lucene 的 索引，会使用机器的资源。**过多的分片会导致额外的性能开销**
	* 分片的 Meta 信息由 Master 节点维护。过多，会增加管理的负担。**经验值，控制分片总数在 10 W 以内**

**5、如何确定主分片数**

* 日志类应用，单个分片不要大于 50 GB
* 搜索类应用，单个分片不要超过20 GB

**具体设置几个主分片需要做容量规划，主分片一旦设定，则不能随意修改，除非做reindex，主分片数是文档路由的关键参数，所以，一旦变化必然需要reindex**

### **2-4 如何对集群进行容量规划**

* 如果需要考虑可靠性高可用，建议部署 3 台 dedicated 的 Master 节点
* 如果有复杂的查询和聚合，建议设置 Coordinating 节点

**创建基于时间序列的索引**

```
# POST /<logs-{now/d}/_search
POST /%3Clogs-%7Bnow%2Fd%7D%3E/_search
```

**写入时间序列的数据 – 基于 Index Alias**

```
POST _aliases
{
  "actions": [
    {
      "add": {
        "index": "log-2020.11.24",
        "alias": "logs_write"
      }
    },
    {
      "remove": {
        "index": "log-2020.11.23",
        "alias": "logs_write"
      }
    }
  ]
}
```

## **3、生产环境中的集群运维**

### **3-1 生产环境常用配置和上线清单**

* 将 内存 Xms 和 Xmx 设置成一样，避免 heap resize 时引发停顿
* Xmx 设置不要超过物理内存的 50%;单个节点上，最大内存建议不要超过 32 G 内存
* 生产环境，JVM 必须使用 Server 模式
* 关闭 JVM Swapping

集群的 API 设定

* Transient 在集群重启后会丢失
* Persistent 在集群中重启后不会丢失

内存设定计算实例

* 搜索类的比例建议: 1:16
* 日志类: 1:48 - 1:96 之间

集群设置:Throttles 限流

为 Relocation 和 Recovery 设置限流，避免过多任务对集群产生性能影响

* Recovery
	* `Cluster.routing.allocation.node_concurrent_recoveries: 2`
* Relocation	
	* `Cluster.routing.allocation.cluster_concurrent_rebalance: 2`

**集群设置:关闭 Dynamic Indexes**

* 可以考虑关闭动态索引创建的功能

```
PUT _cluster/settings
{
	"persistent" : {
		"action.auto_create_index": false
	}
}
```

* 或者通过模版设置白名单

```
PUT _cluster/settings
{
	"persistent" : {
		"action.auto_create_index": "logstash-*,.kibana*"
	}
}
```

### **3-2 监控Elasticsearch集群**

**1、Elastcsearch Stats相关的API**

* Node Stats: `_nodes/stats`
* Cluster Stats: `_cluster/stats`
* Index Stats: `index_name/_stats`

**2、Elasticsearch Task API**

* 查看Task相关的API
	* **Pending Cluster Tasks API: `GET _cluster/pending_tasks`**
	* Task Management API: `GET _tasks`（可以用来Cancel一个Task)

* 监控Thread Pools
	
	* `GET _nodes/thread_pool`
	* `GET _nodes/stats/thread_pool`
	* `GET _cat/thread_pool?v`
	* `GET _nodes/hot_threads`


**3、The Index & Query Slow Log**

* 支持将分片上，Search 和 Fetch 阶段的慢查询写入文件
* 支持为 Query 和 Fetch 分别定义阈值
* 索引级的动态设置，可以按需设置，或者通过 Index Template 统一设定
* Slow log文件通过`log4j2.properties`配置

```
PUT /my_inde/_settings
{
 	"settings" : {
 		"index.indexing.slowlog.threshold" : {
 			"query:warn" : "10s",
 			"query:info" : "3s",
 			"query:debug" : "2s",
 			"query:trace" : "0s",
 			"fetch:warn" : "1s",
 			"fetch:info" : "600ms",
 			"fetch:debug" : "400ms",
 			"fetch:trace" : "0s",
 		}
 	}
}
```

**stats**

```
# Node Stats：
GET _nodes/stats


#Cluster Stats:
GET _cluster/stats


# Index Stats
GET kibana_sample_data_ecommerce/_stats
```

**Pending Cluster Tasks API**

```
#Pending Cluster Tasks API:
GET _cluster/pending_tasks

{
  "tasks" : [ ]
}
```

查看所有的 tasks，也支持 cancel task

```
# 查看所有的 tasks，也支持 cancel task
GET _tasks
```

`thread_pool`

```
GET _nodes/thread_pool
GET _nodes/stats/thread_pool

GET _nodes/hot_threads
```

**Index Slowlogs**

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

### **3-3 解决集群 Yellow 与 Red 的问题**

**集群健康度**

* `GET _cluster/health`: 集群的状态(检查节点数量)
* `GET _cluster/health?level=indices`: 所有索引的健康状态 (查看有问题的索引)
* `GET _cluster/health/my_index`: 单个索引的健康状态(查看具体的索引)
* `GET _cluster/health?level=shards:` 分片级的索引
* `GET _cluster/allocation/explain`: 返回第一个未分配 Shard 的原因

**Explain 变红的原因**

```
# Explain 变红的原因
GET /_cluster/allocation/explain
```

**Explain 看 hot 上的 explain**

```
PUT mytest
{
  "settings":{
    "number_of_shards":2,
    "number_of_replicas":1,
    "index.routing.allocation.require.box_type":"hot"
  }
}
```

可以指定 Move 或者 Reallocate 分片

```
POST _cluster/reroute
{
	"commands" : {
		"move" : {
			"index": "index_name",
			"shard": 0,
			"from_node": "node_name_1",
			"to_node": "node_name_2",
		}
	}
}


POST _cluster/reroute?explain
{
	"commands" : {
		"allocate" : {
			"index": "index_name",
			"shard": 0,
			"node": "nodename",
		}
	}
}
```

### **3-4 集群写性能优化**

一个索引设定的例子

```
PUT myindex
{
  "settings": {
    "index": {
      "refresh_interval": "30s",  # 30s 一次的refresh
      "number_of_shards": "2"
    },
    "routing": {
      "allocation": {
        "total_shards_per_node": "3" 	#控制分片，避免数据热点
      }
    },
    "translog": {
      "sync_interval": "30s",			# 降低 translog 落盘
      "durability": "async"
    },
    "number_of_replicas": 0
  },
  "mappings": {        		# 避免不必要的字段索引。 必要事时可以通过 update by query索引必要的字段
    "dynamic": false,
    "properties": {}
  }
}
```

### **3-5 段合并优化及注意事项**

```
POST geonames/_forcemerge?max_num_segments=1
get _cat/segements/geonames?v
```

## **4、缓存及使用 Circuit Breaker 限制内存使用**

```
PUT /my_index
{
	"settings" : {
		"index.request.cache.enable" : false
	}
}

GET /my_index/_search?request_cache=true
{
	"size": 0,
	"aggs": {
		"popular_color": {
			"terms": {
				"field": "colors"
			}
		}
	}
}
```

**诊断内存状况**

查看各个节点的内存状况

* `GET _cat/nodes?v`
* `GET _nodes/stats/indices?pretty`
* `GET _cat/nodes?v&h=name,queryCacheMemory,queryCacheEvictions,requestCacheMemory,reques tCacheHitCount,request_cache.miss_count`
* `GET _cat/nodes?h=name,port,segments.memory,segments.index_writer_memory,fielddata.memo ry_size,query_cache.memory_size,request_cache.memory_size&v`

**Circuit Breaker 统计信息**

```
GET /_nodes/stats/breaker?
```

```
PUT /_cluster/settings
{
    "persistent" : {
       "indices.breaker.request.limit" : "90%"
    }
}

```

### **5、一些运维相关的建议**

**Full Restart 的步骤**

* 停止索引数据，同时备份集群
* Disable Shard Allocation (Persistent)
* 执行 Synced Flush
* 关闭并更新所有节点
* 先运行所有 Master 节点 / 再运行其他节点
* 等集群变黄后打开 Shard Allocation

```
curl -XGET 'http://ip:9200/_cluster/health?pretty=true'
curl -XGET 'http://ip:9200/_cat/indices?v'
```

Disable Shard Allocation (Persistent)

```
curl -X PUT "http://ip:9200/_cluster/settings" -H 'Content-Type: application/json' -d'

{

   "transient":{

      "cluster.routing.allocation.enable":"none"

   }

}
'
```

执行 Synced Flush

```
curl -X POST "http://ip:9200/_flush/synced"
```

等集群变黄后打开 Shard Allocation

```
 curl -X PUT "http://ip:9200/_cluster/settings" -H 'Content-Type: application/json' -d'

{

   "transient":{

      "cluster.routing.allocation.enable":"all"

   }

}

'
```

**运维 Cheat Sheet**

```
# 移动一个分片从一个节点到另外一个节点

POST _cluster/reroute
{
  "commands": [
    {
      "move": {
        "index": "index_name",
        "shard": 0,
        "from_node": "node_name_1",
        "to_node": "node_name_2"
      }
    }
  ]
}


# Fore the allocation of an unassinged shard with a reason

POST _cluster/reroute?explain
{
  "commands": [
    {
      "allocate": {
        "index": "index_name",
        "shard": 0,
        "node": "nodename"
      }
    }
  ]
}


# remove the nodes from cluster 
PUT _cluster/settings
{
  "transient": {
    "cluster.routing.allocation.exclude._ip":"the_IP_of_your_node"
  }
}

# Force a synced flush
POST _flush/synced


# change the number of moving shards to balance the cluster
PUT /_cluster/settings
{
  "transient": {"cluster.routing.allocation.cluster_concurrent_rebalance":2}
}

# change the number of shards being recovered simultanceously per node
PUT _cluster/settings
{
  "transient": {"cluster.routing.allocation.node_concurrent_recoveries":5}
}

# Change the recovery speed
PUT /_cluster/settings
{
  "transient": {"indices.recovery.max_bytes_per_sec": "80mb"}
}

# Change the number of concurrent streams for a recovery on a single node
PUT _cluster/settings
{
  "transient": {"indices.recovery.concurrent_streams":6}
}


# Change the sinze of the search queue
PUT _cluster/settings
{
  "transient": {
    "threadpool.search.queue_size":2000
  }
}

# Clear the cache on a node
POST _cache/clear


#Adjust the circuit breakers
PUT _cluster/settings
{
  "persistent": {
    "indices.breaker.total.limit":"40%"
  }
}
```

## **4、索引生命周期管理**

### **4-1 使用 Shrink 与 Rollover API 管理索引**

* `Open / Close Index`: 索引关闭后无法进行读写，但是索引数据不会被删除
* `Shrink Index`:可以将索引的**主分片数收缩到较小的值**
* `Split Index`:可以扩大主分片个数
* `Rollover Index`:类似 `**Log4J**` 记录日志的方式，**索引尺寸或者时间超过一定值后，创建新的**
* `Rollup Index`:对数据进行处理后，重新写入，减少数据量

```
#关闭索引
POST /test/_close
```

```
#打开索引
POST /test/_open

POST test/_search
{
  "query": {
    "match_all": {}
  }
}
```

```
POST test/_count
```

**Shrink API**

```
PUT my_source_index
{
 "settings": {
   "number_of_shards": 4,
   "number_of_replicas": 0
 }
}

GET _cat/shards/my_source_index

# 分片数3，会失败
POST my_source_index/_shrink/my_target_index
{
  "settings": {
    "index.number_of_replicas": 0,
    "index.number_of_shards": 3,
    "index.codec": "best_compression"
  },
  "aliases": {
    "my_search_indices": {}
  }
}

# 报错，因为没有置成 readonly
POST my_source_index/_shrink/my_target_index
{
  "settings": {
    "index.number_of_replicas": 0,
    "index.number_of_shards": 2,
    "index.codec": "best_compression"
  },
  "aliases": {
    "my_search_indices": {}
  }
}

# 将 my_source_index 设置为只读
PUT /my_source_index/_settings
{
  "settings": {
    "index.blocks.write": true
  }
}


# 报错，必须都在一个节点
POST my_source_index/_shrink/my_target_index
{
  "settings": {
    "index.number_of_replicas": 0,
    "index.number_of_shards": 2,
    "index.codec": "best_compression"
  },
  "aliases": {
    "my_search_indices": {}
  }
}
```

```
## 确保分片都在 hot
PUT my_source_index
{
 "settings": {
   "number_of_shards": 4,
   "number_of_replicas": 0,
   "index.routing.allocation.include.box_type":"hot"
 }
}

GET _cat/shards/my_source_index

#设置为只读
PUT /my_source_index/_settings
{
  "settings": {
    "index.blocks.write": true
  }
}

POST my_source_index/_shrink/my_target_index
{
  "settings": {
    "index.number_of_replicas": 0,
    "index.number_of_shards": 2,
    "index.codec": "best_compression"
  },
  "aliases": {
    "my_search_indices": {}
  }
}
```
```
GET _cat/shards/my_target_index
```

**Split API**

```
PUT my_source_index
{
 "settings": {
   "number_of_shards": 4,
   "number_of_replicas": 0
 }
}

# 必须是倍数
POST my_source_index/_split/my_target
{
  "settings": {
    "index.number_of_shards": 10
  }
}

# 必须是只读
POST my_source_index/_split/my_target
{
  "settings": {
    "index.number_of_shards": 8
  }
}
```

```
#设置为只读
PUT /my_source_index/_settings
{
  "settings": {
    "index.blocks.write": true
  }
}


POST my_source_index/_split/my_target_index
{
  "settings": {
    "index.number_of_shards": 8,
    "index.number_of_replicas":0
  }
}
```

**Rollover API**

**当满足一系列的条件，Rollover API 支持将一个 Alias 指向一个新的索引**

* 存活的时间
* 最大文档数
* 最大的文件尺寸

一般需要和 Index Lifecycle Management Policies 结合使用

* 只有调用 Rollover API 时，才会去做相应的检测。ES 并不会自动去监控这些索引

```
# 不设定 is_write_true
# 名字符合命名规范
PUT /nginx-logs-000001
{
  "aliases": {
    "nginx_logs_write": {}
  }
}

# 多次写入文档 > 5
POST nginx_logs_write/_doc
{
  "log":"something"
}

POST /nginx_logs_write/_rollover
{
  "conditions": {
    "max_age":   "1d",
    "max_docs":  5,
    "max_size":  "5gb"
  }
}
```

**Rollover API `is_write_index`**

```
# 设置 is_write_index
PUT apache-logs1
{
  "aliases": {
    "apache_logs": {
      "is_write_index":true
    }
  }
}
```

```
# 需要指定 target 的名字
POST /apache_logs/_rollover/apache-logs8xxxx
{
  "conditions": {
    "max_age":   "1d",
    "max_docs":  1,
    "max_size":  "5gb"
  }
}
```

### **4-2 索引全生命周期管理及工具介绍**

**Index Lifecycle Management**

ILM概念

* Policy
* Phase
* Action

```
# 设置 1秒刷新1次，生产环境10分种刷新一次
PUT _cluster/settings
{
  "persistent": {
    "indices.lifecycle.poll_interval":"1s"
  }
}

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

## **5、集群 Backup & Restore**

```
cd /etc/elasticsearch
vim elasticsearch.yml

path.repo: ["/home/vagrant/bak"]

# 创建一个 repositoty
PUT /_snapshot/my_fs_backup
{
    "type": "fs",
    "settings": {
        "location": "/home/vagrant/bak",
        "compress": true
    }
}

# 创建一个snapshot
PUT /_snapshot/my_fs_backup/snapshot_1?wait_for_completion=true

# 指定索引创建快照
PUT /_snapshot/my_fs_backup/snapshot_2?wait_for_completion=true
{
  "indices": "my_index",
  "ignore_unavailable": true,
  "include_global_state": false,
  "metadata": {
    "taken_by": "jacob",
    "taken_because": "backup before upgrading"
  }
}

# 查看所有的快照
GET /_snapshot/my_fs_backup/_all

#  删除快照
DELETE /_snapshot/my_fs_backup/snapshot_2

#  回复快照
DELETE my_index

POST /_snapshot/my_fs_backup/snapshot_1/_restore
{
  "indices": "my_index",
  "index_settings": {
    "index.number_of_replicas": 5
  },
  "ignore_index_settings": [
    "index.refresh_interval"
  ]
}
```

