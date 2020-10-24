# **第九节 Elasticsearch 入门总结 & 测试**

## **1、产品与使用场景**

* Elasticsearch是一个开源的分布式搜索与分析引擎，**提供了近实时搜索和聚合两大功能**
* Elastic Stack包括Elasticsearch, Kibana, Logstash, Beats等一系列产品。
* Elasticsearch是核心引擎，提供了海量数据存储，搜索和聚合的能力。Beats是轻量的 数据采集器，Logstash用来做数据转换，Kibana则提供了丰富的可视化展现与分析的功 能。
* Elastic Stack主要被广泛使用于：**搜索，日志管理，安全分析，指标分析，业务分析，应用性能监控等多个领域** 
* Elastic Stack开源了X-Pack在内的相关代码。作为商业解决方案，X-Pack的部分功能需要收费。Elastic公司从6.8和7.1开始，Security功能也可以免费使用 
* **相比关系型数据库，Elasticsearch提供了如模糊查询，搜索条件的算分第等关系型数据库所不擅长的功能，但是在事务性等方面，也不如关系型数据库来的强大。因此，在实际生产环境中，需要考虑具体业务要求，综合使用**

## **2、基本概念**

* 一个Elasticsearch集群可以运行在单节点上，也支持运行在多个服务器上，实现数据和服务的水平扩展 
* **从逻辑角度看，索引是一些具有相似结构的文档的集合** 
* 物理角度看，**分片是一个Lucene的实例**。分片存储了索引的具体数据，分片可以分布在不同的节点之上。副本分片除了提高数据的可靠性，还能一定程度提升集群查询的性能 
* **Elasticsearch的文档可以是任意的JSON格式的数据** 
* **将文档写进Elasticseach的过程叫索引(indexing)**
* **Elasticsearch提供了REST API和Transport API两种方式，建议使用REST API** 


## 3、**搜索和Aggregation** 

* Precision 指除了相关的结果，还返回了多少不相关的结果 
* Recall —— 衡量有多少相关的结果，实际上并没有返回 
* **精确值包括：数字，日期和某些具体的字符串** 
* **全文本：是需要被检索的非结构文本(Analyzer)** 
* Analyis：是将文本转换成倒排索引中的Terms的过程 
* Elasticsearch的`Analyzer`是`Char_filter -> Tokenizer -> Token Filter`的过程
* 要善于利用`_analyze API`去测试`Analyzer` 
* `Elasticsearch`搜索支持`URI Search`和`REST Body`两种方式 
* `Elasticsearch`提供了`Bucket/Metric/Pipeline/Matrix`四种方式的聚合 


## **4、文档CRUD与Index Mapping** 

* 除了CRUD操作外，Elasticsearch还提供了`bulk`, `mget`和`mseach`等操作。从性能的角度上说，建议使用，以提升性能。但是，单次操作的数据量不要过大，以免引发性能问题 
* 每个索引都有一个`Mapping`定义。包含文档的字段及类型，字段的`Analyzer`的相关配置
* `Mapping`可以被动态的创建，为了避免一些错误的类型推算或者满足你特定的需求，可以显示 的定义`Mapping` 
* `Mapping`可以动态创建，也可以显示定义。你可以在`Mapping`中定制`Analyzer` 
* 你可以为字段指定定制化的`analyzer`，也可以为查询字符串指定`search_analyzer`
* `Index Template`可以定义`Mapping`和`Settings`，并自动的应用到新创建的索引之上，建议要合理的使用Index Template 
* Dynamic Template支持在具体的索引上指定规则，为新增加的字段指定相应的Mappings 


## **5、测试**

* 1.判断题：ES支持使用HTTP PUT写入新文档，并通过Elasticsearch生成文档Id 

> 1．错，需要用POST命令创建。 

* 2.判断题：Update一个文档，需要使用HTTP PUT

> 错，Update文档，使用Pos下，PUT只能用来做Index或者Create 

* 3.判断题：Index一个已存在的文档，旧的文档会先被删除，新的文档再被写入，同时版本号加1 

> 对

* 4．尝试描述创建一个新的文档到一个不存在的索引中，背后会发生一些什么？ 

> 默认情况下，会创建相应的索引，并且自己设置Mapping，当然，实际情况还是要看是否有合适的Index Template  

* 5.ES7中的合法的type是什么？ 

> _doc 

* 6.精确值和全文的本质区别是什么？ 

> 精确值不会被Analyzer分词，全文本会 

* 7.Analyzer由哪几个部分组成？ 

> 三部分: Character Filter + Tokenizer + Token Filter


* 8.尝试描述`match`和`match_phrase`的区别 

> `Match`中的`terms`之间是`or`的关系，`Match Phrase`的`terms`之间是`and`的关系 
并且`term`之间位置关系也影响搜索的结果

* 9.如果你希望`match_phrase`匹配到更多结果，你应该配置查询中什么参数？

> slop 

* 10.如果`Mapping`的`dynamic`设置成`“strict"`，索引一个包含新增字段的文档时会发生什么？ 

> 直接报错 

* 11.如果`Mapping`的`dynamic`设置成`“false"`，索引一个包含新增字段的文档时会发生什么？ 

> 文档被索引，新的字段在`_source`中可见。但是该字段无法被搜索

* 12.判断：可以把一个字段的类型从`“integer”`改成`“long"`，因为这两个类型是兼容的 

> 错。字段类型修改，需要重新reindex 

* 13．判断：你可以在`Mapping`文件中为`indexing`和`searching`指定不同的`analyzer` 

> 对。可以在`Mapping`中为`index`和`search`指定不同的`analyizer`

* 14．判断：字段类型为Text的字段，一定可以被全文搜索 

> 错。可以通过为text类型的字段指定`Not Indexed`，使其无法被搜索 




 