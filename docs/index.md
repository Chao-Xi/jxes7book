# **手摸手 Elasticsearch7 技术与实战教程**

> Started at Oct. 2020 By Jacob Xi 

![Alt Image Text](images/indx1_0.png "Body image")

## 内容简介

本书是本人的“手摸手战术教程”系列的第二本，历经四个月的时间，终于跟随着上海火箭般🚀上涨的楼市在2021春节之前顺利完工，感谢家人，朋友，同事们的支持与理解，也感谢所在team同事们的帮助与指导。

> [手摸手 Jenkins 战术教程 (大师版）](https://chao-xi.github.io/jxjenkinsbook/)

本书着重介绍了 ElasticSearch7, EFK & ElasticStack Monitoring Kubernetes, Logstash, Kibana 以及 X-PACK 在实际生产工作中的各种用法与流水线的开发，运维，以及与各个常用生产工具的整合。并且简单介绍了 Elastic 认证考试的考证介绍与流程。

本书共19章，主要内容包括：

* Elasticsearch, Kibana, logstash的安装以及Docker中运行ES, Kibana, Cerebo集群
* Elasticsearch 的查询语句 (Bulk，倒排，分析器，Mapping, Index Template&Dynamic Template，Aggregation)
* Elasticsearch 深入搜索 （词项，全文，结构化算分， Query&Filtering 多字符串，单字符串，中文，Search Template & Index Alias，Function Score Query， Term & Phrase Suggester，⾃动补全，跨集群搜索）
* 分布式特性及分布式搜索的机制 (集群分布式模型，分⽚与集群的故障转移，⽂档分布式存储，分⽚生命周期，排序及Doc Values&Fielddata，分页与遍历，处理并发)
* 深入聚合分析（Bucket & Metric 聚合分析， Pipeline 聚合分析，聚合作⽤范围及排序精准度问题）
* 数据建模 (对象及 Nested 对象，文档的父子关系，Update By Query & Reindex API，ngest Pipeline 与 Painless Script）
* 数据保护，水平扩展Elasticsearch集群，生产环境中的集群运维，索引生命周期管理
* Kubernets 安装 EFK 集群， Elastic Stack 构建 Kubernetes 全栈监控
* 生产环境中的集群运维，索引生命周期管理
* 用Logstash和Beats构建数据管道，用Kibana进行数据可视化分析， 探索X-Pack套件



## Salut! C'est Moi

> The man is not old as long as he is seeking something, A man is not old until regrets take the place of dreams.

Hello, this is me, Jacob. Currently, I'm working as DevOps and Cloud Engineer in SAP, and I'm the certified AWS Solution Architect and Certified Azure Administrator, Kubernetes Specialist, Jenkins CI/CD and ElasticStack enthusiast. 

I was working as Backend Engineer in New York City and achieved my CS master degree in SIT, America. Believe it or not, I'll keep writing, more and more books will come out at such dramatic and unprecedented 2021. 

If you have anything want to talk to me directly, you can reach out for via email xichao2015@outlook.com。


Salute, c'est moi, Jacob. Actuellement, je travaille en tant qu'ingénieur DevOps et Cloud dans SAP, et je suis architecte de solution AWS certifié et administrateur Azure certifié, spécialiste Kubernetes et passionné de CI/CD.

Je travaillais en tant qu'ingénieur backend à New York et j'ai obtenu mon master CS à SIT, en Amérique. Croyez-le ou non, je continuerai à écrire, de plus en plus de livres sortiront cette année.

## 目录大纲

* **第一章 Elasticsearch 概述**
	* [第一节 ElasticSearch 概述](https://chao-xi.github.io/jxes7book/chap1/1Es_intro/)
* **第二章 Elasticsearch 安装手册**
	* [第一节 Elasticsearch的安装与简单配置](https://chao-xi.github.io/jxes7book/chap2/1ES_vm_install/)
	* [第二节 Kibana的安装与界面快速预览](https://chao-xi.github.io/jxes7book/chap2/2KB_vm_install/)
	* [第三节 在Docker容器中运行Elasticsearch Kibana和Cerebro](https://chao-xi.github.io/jxes7book/chap2/3ES_KB_docker/)
	* [第四节 Logstash安装与导入数据](https://chao-xi.github.io/jxes7book/chap2/4LogStash_vm_install/)
* **第三章 Elasticsearch 入门查询**
	* [第一节 ElasticSearch 基本概念](https://chao-xi.github.io/jxes7book/chap3/1ES_intro/)
	* [第二节 文档的基本CRUD与批量操作 & 读取＆Bulk API](https://chao-xi.github.io/jxes7book/chap3/2ES_CRUD/)
	* [第三节 倒排索引](https://chao-xi.github.io/jxes7book/chap3/3ES_ReverseIndex/)
	* [第四节 使用分析器进行分词](https://chao-xi.github.io/jxes7book/chap3/4ES_Analyzer/)
	* [第五节 ElasticSearch Search语句](https://chao-xi.github.io/jxes7book/chap3/5ES_Search/)
	* [第六节 ElasticSearch Mapping 介绍](https://chao-xi.github.io/jxes7book/chap3/6ES_Mapping/)
	* [第七节 Index Template和Dynamic Template](https://chao-xi.github.io/jxes7book/chap3/7ES_Index/)
	* [第八节 Elasticsearch Aggregations聚合分析简介](https://chao-xi.github.io/jxes7book/chap3/8ES_Aggr/)
	* [第九节 Elasticsearch 入门总结 & 测试](https://chao-xi.github.io/jxes7book/chap3/9ES_SUM/)
* **第四章 Elasticsearch 深入搜索**
	* [第一节 基于词项和基于全文的搜索](https://chao-xi.github.io/jxes7book/chap4/1ES_search/)
	* [第二节 结构化搜索以及搜索的相关性算分](https://chao-xi.github.io/jxes7book/chap4/2ES_search_str_score/)
	* [第三节 Query & Filtering与多字符串多字段查询](https://chao-xi.github.io/jxes7book/chap4/3ES_query_filter/)
	* [第四节 单字符串多字段查询](https://chao-xi.github.io/jxes7book/chap4/4ES_sin_mul_search/)
	* [第五节 多语言及中文分词与检索](https://chao-xi.github.io/jxes7book/chap4/5ES_multi_lag_ana/)
	* [第六节 Space Jam，一次全文搜索的实例](https://chao-xi.github.io/jxes7book/chap4/6ES_pra_tmdb/)
	* [第七节 使用 Search Template 和 Index Alias](https://chao-xi.github.io/jxes7book/chap4/7ES_ST_IA/)
	* [第八节 综合排序:Function Score Query 优化算分](https://chao-xi.github.io/jxes7book/chap4/8ES_FUNC_SCORE/)
	* [第九节 Term & Phrase Suggester](https://chao-xi.github.io/jxes7book/chap4/9ES_suggester/)
	* [第十节 ⾃动补全与基于上下文的提示](https://chao-xi.github.io/jxes7book/chap4/10ES_Autocomplete/)
	* [第十一节 跨集群搜索](https://chao-xi.github.io/jxes7book/chap4/11ES_CrossSearch/)
* **第五章 分布式特性及分布式搜索的机制**
	* [第一节 集群分布式模型及选主与脑裂问题](https://chao-xi.github.io/jxes7book/chap5/1ES_cluster/)
	* [第二节 分⽚与集群的故障转移](https://chao-xi.github.io/jxes7book/chap5/2ES_shard/)
	* [第三节 ⽂档分布式存储](https://chao-xi.github.io/jxes7book/chap5/3ES_doc_shard/)
	* [第四节 分⽚及其生命周期](https://chao-xi.github.io/jxes7book/chap5/4ES_shared_lifecycle/)
	* [第五节 剖析分布式查询及相关性算分](https://chao-xi.github.io/jxes7book/chap5/5ES_SE_Score/)
	* [第六节 排序及Doc Values&Fielddata](https://chao-xi.github.io/jxes7book/chap5/6ES_Sort_query/)
	* [第七节 分页与遍历 – From，Size，Search After & Scroll API](https://chao-xi.github.io/jxes7book/chap5/7ES_Pages_Traverse/)
	* [第八节 处理并发读写操作](https://chao-xi.github.io/jxes7book/chap5/8ES_Concurrent_process/)
* **第六章 深入聚合分析**
	* [第一节 Bucket & Metric 聚合分析及嵌套聚合](https://chao-xi.github.io/jxes7book/chap6/1chap6_bucket_metrics/) 
	* [第二节 Pipeline 聚合分析](https://chao-xi.github.io/jxes7book/chap6/2chap6_pipeline_metrics/)
	* [第三节 聚合的作⽤用范围及排序](https://chao-xi.github.io/jxes7book/chap6/3chap6_aggragate_scope_order/)
	* [第四节 聚合的精准度问题](https://chao-xi.github.io/jxes7book/chap6/4chap6_accurate_aggragate/)
* **第七章 数据建模**
	* [第一节 对象及 Nested 对象](https://chao-xi.github.io/jxes7book/chap7/1chap7_nested_obj/)
	* [第二节 文档的父子关系](https://chao-xi.github.io/jxes7book/chap7/2chap7_parent_child/)
	* [第三节 Update By Query & Reindex API](https://chao-xi.github.io/jxes7book/chap7/3chap7_updateindex_reindex/)
	* [第四节 Ingest Pipeline 与 Painless Script](https://chao-xi.github.io/jxes7book/chap7/4chap7_ingest_painless/)
	* [第五节 Elasticsearch 数据建模实例](https://chao-xi.github.io/jxes7book/chap7/5chap7_es_model/)
	* [第六节 Elasticsearch 数据建模最佳实践](https://chao-xi.github.io/jxes7book/chap7/6chap7_es_model_pra/)
	* [第七节 第二部分总结回顾](https://chao-xi.github.io/jxes7book/chap7/7chap7_es_sum2/)
* **第八章 数据保护**
	* [第一节 集群身份认证与用户鉴权](https://chao-xi.github.io/jxes7book/chap8/1ES_cluster_IAM/)
	* [第二节 集群内部间的安全通信](https://chao-xi.github.io/jxes7book/chap8/2ES_cluster_enc_transit/)
	* [第三节 集群与外部间的安全通信](https://chao-xi.github.io/jxes7book/chap8/3ES_cluster_out_sec/)
* **第九章 水平扩展Elasticsearch集群**
	* [第一节 常见的集群部署方式](https://chao-xi.github.io/jxes7book/chap9/1ES_cluster_deploy/)
	* [第二节 Hot & Warm 架构与 Shard Filtering](https://chao-xi.github.io/jxes7book/chap9/2ES_hot_warm/)
	* [第三节 分片设定及管理](https://chao-xi.github.io/jxes7book/chap9/3ES_shard_settings/)
	* [第四节 如何对集群进行容量规划](https://chao-xi.github.io/jxes7book/chap9/4ES_cluster_plan/)
	* [第五节、云上管理 Elasticsearch 的一些方法](https://chao-xi.github.io/jxes7book/chap9/5ES_Cloud/)
* **第十章 使用 Kubernets 安装 EFK 集群**
	* [第一节 kubernetes 日志架构](https://chao-xi.github.io/jxes7book/chap10/1log_architecture/) 
	* [第二节 在 K8S 上搭建 EFK 日志收集系统](https://chao-xi.github.io/jxes7book/chap10/2EFK_log/)
	* [第三节 在 Kubernetes 安装 EFK 日志收集系统[更新]](https://chao-xi.github.io/jxes7book/chap10/16EFK_log_adv/)
	* [第四节 kibana 日志分析](https://chao-xi.github.io/jxes7book/chap10/17EFK_log_Ana/)
	* [第五节 基于EKF日志的报警](https://chao-xi.github.io/jxes7book/chap10/18EFK_log_alert/)
	* [第六节 使用Operator快速部署Elasticsearch集群](https://chao-xi.github.io/jxes7book/chap10/3ECK_operator/)
* **第十一章 使用 Elastic Stack 构建 Kubernetes 全栈监控**
	* [第一节 ElasticSearch 的集群安装](https://chao-xi.github.io/jxes7book/chap11/20Elastic_Stack_Monitoring1/) 
	* [第二节 使用 Elastic Stack — Metricbeat 监控安装](https://chao-xi.github.io/jxes7book/chap11/21Elastic_Stack_Monitoring2/)
	* [第三节 使用 Elastic Stack - Filebeat 日志数据收集安装](https://chao-xi.github.io/jxes7book/chap11/22Elastic_Stack_Monitoring3/)
	* [第四节 使用 Elastic Stack — Elastic APM 应用性能监控的工具安装](https://chao-xi.github.io/jxes7book/chap11/23Elastic_Stack_Monitoring4/)
* **第十二章 生产环境中的集群运维**
	* [第一节 生产环境常用配置和上线清单](https://chao-xi.github.io/jxes7book/chap12/1chap12_ES_prod/)
	* [第二节 监控Elasticsearch集群](https://chao-xi.github.io/jxes7book/chap12/2chap12_mon_es/)
	* [第三节 诊断集群的潜在问题](https://chao-xi.github.io/jxes7book/chap12/3chap12_es_diag/)
	* [第四节 解决集群 Yellow 与 Red 的问题](https://chao-xi.github.io/jxes7book/chap12/4chap12_es_red_yellow/)
	* [第五节 集群写性能优化](https://chao-xi.github.io/jxes7book/chap12/5chap12_es_cluster_perf/)
	* [第六节 集群压力测试](https://chao-xi.github.io/jxes7book/chap12/6chap12_es_pre_test/)
	* [第七节 段合并优化及注意事项](https://chao-xi.github.io/jxes7book/chap12/7chap12_es_segment_merge/)
	* [第八节 缓存及使用 Circuit Breaker 限制内存使用](https://chao-xi.github.io/jxes7book/chap12/8chap12_Circuit_Breaker/)
	* [第九节 一些运维相关的建议](https://chao-xi.github.io/jxes7book/chap12/9chap12_es_ops/)
* **第十三章 索引生命周期管理**
	* [第一节 使用 Shrink 与 Rollover API 管理索引](https://chao-xi.github.io/jxes7book/chap13/1chap13_es_shrink_rolloverapi/) 
	* [第二节 索引全生命周期管理及工具介绍](https://chao-xi.github.io/jxes7book/chap13/2chap13_index_lms/)
* **第十四章 用Logstash和Beats构建数据管道**
	* [第一节 Logstash 入门及架构介绍](https://chao-xi.github.io/jxes7book/chap14/1logstash_introduce/)
	* [第二节 利用 JDBC 插件导入数据到 Elasticsearch](https://chao-xi.github.io/jxes7book/chap14/2JDBC_logstash_ES/)
	* [第三节 Beats 介绍](https://chao-xi.github.io/jxes7book/chap14/3Beats/)
* **第十五章 用Kibana进行数据可视化分析**
	* [第一节 使用 Index Pattern 配置数据](https://chao-xi.github.io/jxes7book/chap15/1chap15_index_pattern/)
	* [第二节 Kibana 使用的技巧](https://chao-xi.github.io/jxes7book/chap15/2chap15_kibana/)
* **第十六章 探索X-Pack套件** 
	* [第一节 用Monitoring和Alerting监控Elasticsearch集群](https://chao-xi.github.io/jxes7book/chap16/1ES_monitor_alert_xpack/)
	* [第二节 用 APM 进行程序性能监控](https://chao-xi.github.io/jxes7book/chap16/2Xpack_APM/)
	* [第三节 用机器学习实现异常检测](https://chao-xi.github.io/jxes7book/chap16/3Xpack_ML/)
	* [第四节 用 Elastic Stack 进行日志管理](https://chao-xi.github.io/jxes7book/chap16/4ELK_Filebeat/)
	* [第五节 用 Canvas 进行数据的实时展示](https://chao-xi.github.io/jxes7book/chap16/5xpack_canvas/)
* **第十七章 ELK 实战 Workshop**
	* [第一节 电影搜索服务](https://chao-xi.github.io/jxes7book/chap17/1ES_WS_movie/)
	* [第二节 Stackoverflow用户调查问卷分析](https://chao-xi.github.io/jxes7book/chap17/2ES_WS_stackoverflow/)
* **第十八章 Elastic认证, 开发与备份** 
	* [第一节 Elastic 认证考试](https://chao-xi.github.io/jxes7book/chap18/1ES_Certified_test/)
	* [第二节 集群 Backup & Restore](https://chao-xi.github.io/jxes7book/chap18/2ES_backup_restore/) 
	* [第三节 基于 Java 和 Elasticsearch 开发应用](https://chao-xi.github.io/jxes7book/chap18/3ES_Java/)

* [ElasticSearch URI 常用语句总结篇](https://chao-xi.github.io/jxes7book/chap19/sum1_search_api/)
* [Elasticsearch 深入搜索总结](https://chao-xi.github.io/jxes7book/chap19/sum2_search_dep/)
* [ElasticSearch 分布式搜索](https://chao-xi.github.io/jxes7book/chap19/sum3_cross_search/)
* [ElasticSearch 深入聚合分析](https://chao-xi.github.io/jxes7book/chap19/sum4_agg_search/)
* [ElasticSearch  数据建模](https://chao-xi.github.io/jxes7book/chap19/sum5_datamodel/)
* [ElasticSearch Operation and Management](https://chao-xi.github.io/jxes7book/chap19/sum6_Opt_Mgt/)


## To be continue

本人将带来手摸手战术教程更多的内容和文章， 接下来的将在Datatase, Redis, Golang, Chef, Azure900, Azure103, AWS Solution Arcitect, AWS Big Data Speciality, Istio, Python带来更多更全面的电子书，敬请期待。


![Alt Image Text](images/indx1_1.png "Body image")


