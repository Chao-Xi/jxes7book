# **第三节 用机器学习实现异常检测**

## **1、异常检测所解决的问题**

* 解决一些基于规则或者 Dashboard 难以实时发现的问题
* IT运维
	* 如何知道系统正常运行
	*  如何调节阈值触发合适的报警
	*  如何进行归因分析
* 信息安全
	* 哪些用户构成了内部威胁
	* 系统是否感染了病毒
*  物联网 / 数据采集监控
	*  工厂和设备是否正常运营

## **2、什么是正常**

* 随着时间的推移，某个个体一直表现出一致的行为
* 某个个体和他的同类比较，一直表现出和其他个体一致的行为

## **3、什么是异常**

* **和自己比** - 个体的行为发生了急剧的变化
* **和他人比** - 个体明显区别于其他的个体

![Alt Image Text](../images/chap16_3_1.png "Body image")

## **4、相关术语**

*  Elastic 平台的机器学习功能
	*  Elastic 的ML，主要针对**时序数据**的**异常检测和预测**
* 非监督机器学习
	* 不需要使用人工标签的数据来学习，仅仅依靠历史数据自动学习
* 贝叶斯统计
	* 一种概率计算方法，使用先验结果来计算现值或者预测未来的数值
* 异常检测
	* 异常代表的是不同的，但未必代表的是坏的 
	* 定义异常需要一些指导，从哪个方面去看

## **5、如何学习“正常”**

* 察不同的人每天走路的步数，由此预测明天他会走多少步
* 需要观察不同的人，需要观察多久?
	* 一天/一周/一个月/一年/十年
* 直觉: 观察的数据多，你的预测越准确
* 使用这些观察来创建一个模型
	*  概率分布函数;使用这个模型找出什么事几乎不可能的事件

## **6、机器学习帮你自动挑选模型**

* 使用成熟的机器学习技术，挑选适合数据的正确的统计模型
* 更好的模型 = 更好的异常检测 = 更少的误报和漏报
*  出现在低概率区域，发现异常 


## **7、模型与需要考虑任何的周期**

**周期选择** 

* 需要一定周期的学习，才能使的置信区间的范围更小 
* 时间太长:影响因素太多，导致随机分布 
* 时间太短:完全是随机波动

![Alt Image Text](../images/chap16_3_2.png "Body image")


## **8、ES ML:单指标 / 多指标 / 种群分析**

![Alt Image Text](../images/chap16_3_3.png "Body image")

## **9、单指标任务**

*  Create New Job
*  选择 Index Pattern，选择 Fare quote
*  选择单指标任务
	* **Aggregation** 
	* **Field**
	* **Bucket Span** (取决于用户的数据业务 / 数据的采样时间 / 波动情况 / 需要预测的频率)
	* Use full data

## **10、异常检测上 Demo**

### **10-1 Upload  server_metrics data**

```
mkdir server_metrics 
cd server_metrics
wget https://download.elasticsearch.org/demos/machine_learning/gettingstarted/server_metrics.tar.gz
tar -xvf server_metrics.tar.gz

cd files

chmod +x upload_server_metrics_noauth.sh
./upload_server_metrics_noauth.sh
```

```
./upload_server_metrics_noauth.sh

== Script for creating index and uploading data == 
 

== Deleting old index == 

{"error":{"root_cause":[{"type":"index_not_found_exception","reason":"no such index [server-metrics]","index_uuid":"_na_","resource.type":"index_or_alias","resource.id":"server-metrics","index":"server-metrics"}],"type":"index_not_found_exception","reason":"no such index [server-metrics]","index_uuid":"_na_","resource.type":"index_or_alias","resource.id":"server-metrics","index":"server-metrics"},"status":404}
== Creating Index - server-metrics == 

{"error":{"root_cause":[{"type":"mapper_parsing_exception","reason":"Root mapping definition has unsupported parameters:  [metric : {properties={deny={type=long}, total={type=long}, @timestamp={type=date}, response={type=float}, service={type=keyword}, host={type=keyword}, accept={type=long}}}]"}],"type":"mapper_parsing_exception","reason":"Failed to parse mapping [_doc]: Root mapping definition has unsupported parameters:  [metric : {properties={deny={type=long}, total={type=long}, @timestamp={type=date}, response={type=float}, service={type=keyword}, host={type=keyword}, accept={type=long}}}]","caused_by":{"type":"mapper_parsing_exception","reason":"Root mapping definition has unsupported parameters:  [metric : {properties={deny={type=long}, total={type=long}, @timestamp={type=date}, response={type=float}, service={type=keyword}, host={type=keyword}, accept={type=long}}}]"}},"status":400}
== Bulk uploading data to index... 

Server-metrics_1 uploaded
Server-metrics_2 uploaded
Server-metrics_3 uploaded
Server-metrics_4 uploaded
Server-metrics_5 uploaded
Server-metrics_6 uploaded
Server-metrics_7 uploaded
Server-metrics_8 uploaded
Server-metrics_9 uploaded
Server-metrics_10 uploaded
Server-metrics_11 uploaded
Server-metrics_12 uploaded
Server-metrics_13 uploaded
Server-metrics_14 uploaded
Server-metrics_15 uploaded
Server-metrics_16 uploaded
Server-metrics_17 uploaded
Server-metrics_18 uploaded
Server-metrics_19 uploaded
Server-metrics_20 uploaded
 done - output to /dev/null

== Check upload 
health status index          uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   server-metrics EQcyn1KcRI-_ZLCAUAebxA   1   1     860643            0    145.5mb         82.9mb

 Server-metrics uploaded
```

### **10-2 Create Index Pattern**

**Stack Management -> Index Patterns**

```
http://192.168.33.12:9200/_cat/indices
green open server-metrics                      EQcyn1KcRI-_ZLCAUAebxA 1 1 905940      0 149.9mb  75.1mb
```

**server-metrics**

![Alt Image Text](../images/chap16_3_4.png "Body image")

![Alt Image Text](../images/chap16_3_5.png "Body image")

![Alt Image Text](../images/chap16_3_6.png "Body image")


**Stack Management -> License Management -> Start trial**

![Alt Image Text](../images/chap16_3_7.png "Body image")

![Alt Image Text](../images/chap16_3_8.png "Body image")


### **10-3 Create job Anomaly Detection**

**1、Ceate Single Metrics job**

![Alt Image Text](../images/chap16_3_9.png "Body image")

**2、Ceate job**

* Tim Range: Click **Use full server-metrics data**

![Alt Image Text](../images/chap16_3_10.png "Body image")

* Pick fields:
	* Sum total
	* **Bucket span: 15m**

![Alt Image Text](../images/chap16_3_11.png "Body image")
	
* Job details:
	* **single-metrics-demo**

![Alt Image Text](../images/chap16_3_12.png "Body image")

* Summary

![Alt Image Text](../images/chap16_3_13.png "Body image")

* View anomalies in different time span

![Alt Image Text](../images/chap16_3_14.png "Body image")

* Run the 2d forcasting

![Alt Image Text](../images/chap16_3_15.png "Body image")

* Created detection jobs: single-metrics-demo

![Alt Image Text](../images/chap16_3_16.png "Body image")

* Add Custom URLS for single-metrics-demo
	* Label:  single-anomly

![Alt Image Text](../images/chap16_3_17.png "Body image")

![Alt Image Text](../images/chap16_3_18.png "Body image")

* Check actions:  single-anomly

![Alt Image Text](../images/chap16_3_19.png "Body image")

![Alt Image Text](../images/chap16_3_20.png "Body image")

* **Sum total < 0**

![Alt Image Text](../images/chap16_3_21.png "Body image")

* Add total: **`total gte 0`** and saved as index 

![Alt Image Text](../images/chap16_3_22.png "Body image")

### **10-3 Create another job Anomaly Detection job with gte 0 index**

![Alt Image Text](../images/chap16_3_23.png "Body image")

* Job details:
	* **enhanced-demo-metrics**

![Alt Image Text](../images/chap16_3_24.png "Body image")

* Created detection jobs: enhanced-demo-metrics

![Alt Image Text](../images/chap16_3_25.png "Body image")

![Alt Image Text](../images/chap16_3_26.png "Body image")

## **11、Multi Metrics 多指标检测**

### **11-1 多指标检测** 

* 多个指标
* 按照某个字段进行分类
* 什么是影响因子
	* 影响因子是一个字段
	* 这个字段从逻辑分析上看，是可能造成异常的原因
	* 不一定必须是一个检测器，但是这个字段通常能够被用于区分数据
	* 对影响因子也可以打分，打分是基于其多大程度会影响到异常


### **11-2 Create job: multi-metric** 

* **Pick fields:**
	* Mean (accpet)
	* Mean (deny)
	* Mean (repsonse)
* **Split field: `service.keword`**
* Bukcet span: **15m**

![Alt Image Text](../images/chap16_3_27.png "Body image")

**job id: multi-metrics-job**

![Alt Image Text](../images/chap16_3_28.png "Body image")

![Alt Image Text](../images/chap16_3_29.png "Body image")

**Run the job**

![Alt Image Text](../images/chap16_3_31.png "Body image")

**Check the anomaly**

![Alt Image Text](../images/chap16_3_30.png "Body image")

![Alt Image Text](../images/chap16_3_32.png "Body image")


## **12、种群分析**

### **12-1 种群分析** 

* 如何分析
	* 种群之间比较
	* 不和自己的过去比较
* 适用情景
	* **个体具备高 Cardinality**
	* **少数个体从时间上是稀疏的**
	* **作为整体看，群体行为是均匀的**
*  不适用的场景
	*  个体的行为各自都不相同

### **12-2 Create pattern index: `user-activity`** 

```
DELETE user-activty
PUT _bulk
{"index": {"_index": "user-activity", "_type": "_doc", "_id": "AVvhFsqUD6JSRXpujonE"}}
{"username": "Frederica", "@timestamp": "2017-04-16T21:06:58.963073", "bytesSent": 313}
....
```
[user-activity.json](../files/user-activity.json)

**`user-activity`**

![Alt Image Text](../images/chap16_3_33.png "Body image")

![Alt Image Text](../images/chap16_3_34.png "Body image")

![Alt Image Text](../images/chap16_3_35.png "Body image")

* @timestamp
* `_id`
* `_index`
* `_score`
* `_type`
* `bytesSent`
* `username`

### **12-3 Create Job Population for `user-activiy`** 

![Alt Image Text](../images/chap16_3_36.png "Body image")

* **Population field: username.keyword**
* Add metrics: **Mean (bytesSent)**
* Bucket span: 15m
* `mean(bytesSent) over "username.keyword"`
* Job ID: population

![Alt Image Text](../images/chap16_3_37.png "Body image")

![Alt Image Text](../images/chap16_3_38.png "Body image")

**Run Job**

![Alt Image Text](../images/chap16_3_39.png "Body image")

**Check who(username.keyword) bytesSent casue most severity**

**Zoraida**

![Alt Image Text](../images/chap16_3_40.png "Body image")

**Check anomalies**

![Alt Image Text](../images/chap16_3_41.png "Body image")


## **13、Calenders**

Calendars contain a list of scheduled events for which you do not want to generate anomalies, such as planned system outages or public holidays.

**settings -> Calenders**

* national-day

![Alt Image Text](../images/chap16_3_42.png "Body image")


![Alt Image Text](../images/chap16_3_43.png "Body image")