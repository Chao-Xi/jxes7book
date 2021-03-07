# **期末考试**


1、根据使用场景，我们可以将Elasticsearch中存储的数据分成以下哪两类？ 

* **Static data**     *
* Indexed data 
* **Time series data**    *
* Big data 

**题目解析** 

一般情况，我们可以将存储在Elasticsearch中的数据分成两大类：Static data和Time series data。 

Static data的增长相对缓慢，如商品分类和库存信息。而Time series data增长日志信息和指标数据。 

2、通过ELK以下哪些组件，我们可以实现对**服务器性能指标和网络流量的监控**？ 

* Heatbeat
* Auditbeat 
* **Metricbeat**  *
* **Package beat** *

答案：CD 

* Heatbeat主要用来搜集网站的uptime和response time; 
* Auditbeat主要用来审计系统用户和进程的活动信息；
* **Metricbeat主要用于搜集服务器的指标信信息**；
* **Packetbeat可以搜集网络流量信息**。 


3、Elasticsearch集群中的节点，相互之间如何进行通信？ 

* 通过9200端口，用REST API进行通信 
* 通过9200端口，用Transport进行通信 
* **通过9300，用Transport API通信**  *
* 通过9300，用REST API通信 

答案：C 

**题目解析**

* Elasticsearh集群内的节点通过9300端口，**以Transport API的方式进行节点间通信**。
* 对外则通过9200端口，**以REST API提供外部程序与用户的访问。** 


4、Elasticsearch查询默认返回多少条结果？ 

* **10**   *
* 20
* 30 
* 40 

答案：A 

**题目解析** 

Elasticsearch的查询，默认使用简单意义上的浅分页。from定义了目标数据的偏移值，size定义当前返回的数目。

**默认from为0, size为10，即所有的查询默认仅仅返回前10条数据。** 


5、Analyzer由哪几个部分构成？ 

* **character filter** *
* index filter 
* **tokenizer** *
* **token filter** *

> 答案：ACD 

题目解析 

* 做全文搜索就需要对文档分析、建索引。 
* 从文档中提取词元（Token)的算法称为分词器（Tokenizer)，在分词前预处理的算法称为字符过滤器（Character Filter)，进一步处理词元的算法称为词元过滤器(Token Filter) , 最后得到词（Term)，整个分析算法称为分析器（Analyzer) 



6、如果将一个索引的`number_of_shards`设置成`4`，同时将`number_of_replica`设置成2，那么这个索引总共会有多少个分片？ 

* 6 
* 8 
* **12** *
* 16 

> 答案：C 
题目解析 

索引总数是**`4+4*2=12`**

7、如果Elasticsearch集群总共有5个节点，而其中有3个`master-eligible`节点，那我们需要将`minimum_master_nodes`设置成多少？ 

* **2**			*
* 3
* 4
* **新版本无需设置** *

答案：AD 

> 题目解析 

在Elasticsearch7之前，为了防止脑裂，一个基本的原则是： 

需要将`minimum_master_nodes`设置成 `N/2+1`。N是集群中`Master-eligible`节点的数量。如果在集群中有3 Master-eligible节点的集群中， `minimum_master_nodes`应该被设为`3/2+1=2`(四舍五入）。

**在新版本中，这个参数已经被废除**


8、默认情况下，Elasticsearch会根据哪个因素，决定将文档存储到具体哪个分片？ 

* 默认根据文档的大小 
* **默认根据文档的ID** 				*
* 默认会随机分配 
* **可以通过用户自行的数值** 		*


**答案：BD**

题目解析 

文档创建时，Elasticsearch需要决定将这个文档存储在哪一个分片。实际上，这个过程是以下公式决定的： 

```
shard=hash(routing)%number_of_primary_shards 
```

* **routing是一个可变值，默认是文档的`_id`，也可以设置成一个自定义的值。**
* **routing通过hash函数生成一个数字，然后这个数字再除以**`number_of_primary_shards`（主分片的数量）后得到余数。
* 这个分布在0到`number_of_primary_shards-1`之间的余数，就是我们所寻求的文档所在分片的位置。 



9、当集群状态变为黄色，说明集群出现了什么问题？ 

* **数据有丢失的风险** 			*
* 部分文档已经丢失 
* 主分片无法分配 
* **副本分片无法分配** 			*


答案：AD 

> 题目解析 如果集群状态为黄色，说明： 

1. 分配了所有主分片，但至少缺少一个副本。 
2. 没有数据丢失，但是此时搜索结果仍将完整，没有丢失数据。 
3. 集群的高可用性在某种程度上会受到影响，我们应该将黄色视为应该提示调查的警告。如果更多分片消失，集群变红，就可能会丢失数据。 

10、当集群主分片不能分配时，Elasticsearch集群会： 

* **集群变红** 				*
* 集群变黄 
* **部分数据不再可用** 		*
* 索引无法使用 

答案：AC 

题目解析  **如果集群状态为红色，说明**： 

* 至少一个主分片分配失败，这将导致一些数据以及索引的某些部分不再可用。
* 尽管如此，ElasticSearch还是允许我们执行查询，至于是通知用户查询结果可能不完整还是挂起查询，则由应用构建者自行决定。 



11、以下那些查询属于Elasticsearch的Aggregation Query? 

* **中国2019年的动作片的总票房是多少**  *
* 附近50公里以内的电影院有哪些 
* 成龙一共演了哪几本电影 
* **成龙在过去10年出演电影的总票房收入是多少**   *

答案：AD 
题目解析 

* **Elasticsearch除了全文检索之外，还支持各种聚合算法。**
* Aggregation共分为三类：Metric Aggregations、Bucket Aggregations、Pipeline Aggregations0
* 而选项B和C则是通过Elasticsearch的地理位置和全文搜索实现。 




12、Elasticsearch查询通过哪几个参数实现对搜索结果的分页？ 

* **from**   *
* **to** 				*
* **scroll** 		*
* **search_after** 	*


答案：ABCD 

> 题目解析 

Elasticsearch 的查询。

提供多种分页方式。”浅“分页可以理解为简单意义上的分页。它的原理很简单，就是查询前20条数据，然后截断前10条，只返回10-20的数据。这样其实白白浪费了前10条数据。 

**from 定义了目标数据的偏移值，size定义当前返回的数目。默认from为0, size为10，即所有的查询默认仅仅返回前10** **通过from和size可以实现基础的浅分页查询**。

而Scroll API则支持我们检索大量数据（甚至全部数据）。Scroll API允许我们做一个初始阶段搜索并且持续批量从Elasticsearch里拉取结果直到没有结果剩下。但`Scroll API`上下文代价很高建议不要将其用于实时用户请求，而应该使用`search_after`参数通过提供实时游标来解决此问题。 


13、通过Elasticsearch提供的哪一种Aggregation，可以满足“本月有多少个不同的用户访问了网站”这个统计需求？ 

* Terms Aggregation 
* **Cardinality Aggregation** *
* Percentile Ranks Aggregation 
* Value Count Aggregation 


答案：B 

题目解析 

* A:Terms Aggregation:聚合分组，类似于sql中group by。例如可以将日志信息按照“error" , "warm", "info”分为不同的buckets
* B: Cardinality Aggregation:去重，类似于sql中distinct，结果为单位数量，如查询共有多少个班级，可以使用Cadinality
* C:Percentile Ranks Aggregation：一个`multi-value`指标聚合，它通过从聚合文档中提取数值来计算一个或多个百分比。 
* D:Value count aggregation:查看某个字段一共有多少个不一样的数值。 

14、如何可以提高Terms Aggregation的精确度？ 

* **增加`shard_size`** 		*
* 减少`the shard_size` 
* 将主分片数设置为1 
* 增加主分片数 


答案：A 

题目解析 丁erms聚合分析不准的原因，数据分散在多个分片上，而Coordinating Node无法获取数据全貌。 

* 解决方案1:当数据量不大时，设置`Primary Shard`为1，实现准确性能。 
* 解决方案2:在分布式数适当增加`shard_size`参数，提高精确度。原理：每次从Shard上额外多获取数据，提升准确率。`Shard_Size`设置过大，会耗费更多的计算资源。 


15、如何设置索引的Mapping，使得当写入的文档包含了未 定义的字段时，写入会最终失败？ 

* dynamic设置为true 
* dynamic设置为false 
* **dynamic设置为strict** *

答案：C 

题目解析 

* 动态映射(dynamic: true): 动态添加新的字段（或缺省）。 
* 静态映射(dynamic: false): 忽略新的字段，在原有的映射基础上，当有新的字段时，不会主动的添加新的映射关系，只作为查询结果出现在查询中。
* **严格模式（dynamic: strict)：如果遇到新的字段，就抛出异常**。 


16、“通配符匹配非常强大，所以应该被广泛使用”，请问这个说法是否正确？ 

* 正确 
* **错误** 		* 


答案：B 

支持的通配符是“”，它匹配任何字符序列(包括空字符）还有“？”，它匹配任何单价字符。 此查询可能很慢，因为它要迭代多个项。特别是通配符项不应以通配符`""`或“?”开头这样的通配查询的开销很大，会对生产环境造成性能的影响。所以一定要谨慎使用。 


17、Elasticsearch的Search template有哪些好处？ 

* **避免重复代码** 
* **减少出错的概率** 
* **更加容易测试和执行** 
* **只允许用户使用一些预定义的查询** 


> 答案：ABCD 

题目解析 Elasticsearch的Search Template允许用户使用可在执行时定义的参数定义查询。使用Search template能带来以下好处： 

1. 避免在多个地方重复代码 
2. 更容易测试和执行您的查询 
3. 在应用程序间共享查询 
4. 允许用户只执行一些预定义的查询 
5. 将搜索逻辑与应用程序逻辑分离 


18、关于Elasticsearch的父子文档对象，哪些陈述是正确的？ 

* 删除child对象会使parent对象和其他的 siblings都被reindex 
* **子文档必须被路由到和副文档相同的分片**   *
* 删除一个父文档会导致所有的子文档都被删除 
* **即使父文档并不存在，也可以创建子文档**   *



答案：BD 

题目解析 

Elasticsearch维护了一个父文档和子文档的映射关系，得益于这个映射，父、子文档关联查询操作非常快。但是这个映射也对父、子文档关系有个限制条件：**父文档和其所有子文档，都必须要存 储在同一个分片中**。而Elasticsearch的父子关联文档有以下优点

1. 父文档的更新不需要重新为子文档重建索引。 
2. 对子文档进行添加，更改，删除操作室不会影响父文档或者其他子文档。 
3. 子文档可以被当做查询请求的结果返回。 



19、找出以下关于Elasticsearch的**错误**陈述： 

* 在生产环境中，建议为所有的索引都创建Alias 
* 使用Bulk API写入数据，会比每次写入一条，多次写入更为高效 
* **将所有的副本都设置为2个或者更多，能为集群提供可靠的备份机制了** *
* 你可以通过一条REST请求，为集群创建一个快照


答案：C 
题目解析 

* A：正确。 在生产环境中，我们建议为所有的索引都创建Alias，这样，就可以很灵活的对索引进行reindex等变更操作，同时又不会对线上业务产生影响。 
* B：正确。相比与每次单条插入数据，使用Bulk API能够减少网络连接的开销, 因此能提升数据写入的性能。 
* **C：错误。 提高Replica的数量，可以提高一个集群的高可用性。但是Replica并不提供备份的机制。**  
* D：正确。  Snapshot and restore模块允许创建单个索引或者整个集群的快照到远程仓库。在早期版本里只支持共享文件系统的仓库，但是现在通过官方的仓库插件可以支持各种各样的后台仓库。 

 
20、找出以下关于Elasticsearch的**正确**陈述： 

* 你可以将索引的字段类型从“integer”转换到 "long"，因为这两种类型是兼容的 
* 如果你想检查哪些文档不包含一个具体的字段，可以通过`must_not` 
* 索引的主分片数在索索引写入数据后可以修改大小 
* **在很多情况下，为一个索引设置一个主分片，是一个好的设计** *

答案：D 

题目解析 

* A:错误。Intger和Long的字段类型并不兼容，索引有文档写入后，不支持修改字段的类型。 
* **B:错误。应该使用exist查询，而不是`must_not`查询。** 
* **C: Elasticsearch主分片数在索引创建时确定，索引创建后不能修改，除非通过`reindex API`重新创建。所以，而副本分片数可以在任意时间修改**。
*  D：很多场景下，为一个索引设置一个主分片是很好的选择。特别是索引数据量小于20G的情况。单个主分片可以避免数据量少，分散不均而导致的相关度算分不准等情况。 



