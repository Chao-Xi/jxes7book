# **第二节 利用 JDBC 插件导入数据到 Elasticsearch**

## **1、同步数据库数据到 Elasticsearch**

需求 – 将数据库中的数据同步到 ES，借助 ES 的全文搜索，提高搜索速度

* 需要把新增用户信息同步到 Elasticsearch 中
* 用户信息 Update 后，需要能被更新到 Elasticsearch
* 支持增量更新
* 用户注销后，不能被 ES 所搜索到

## **2、JDBC Input Plugin & 设计实现思路**

* 支持通过 JDBC Input Plugin 将数据从数据库从读到 Logstash
	* 需要自己提供所需的 JDBC Driver
* Scheduling
	* 语法来自 Rufus-scheduler
	* 扩展了 Cron，支持时区
* State
	*  `Tracking_column` / `sql_last_value`

[https://www.elastic.co/downloads/jdbc-client](https://www.elastic.co/downloads/jdbc-client)

```
cd /usr/share/logstash/logstash-core/lib/jars
wget https://cdn.mysql.com//Downloads/Connector-J/mysql-connector-java-5.1.49.zip
sudo yum install unzip
unzip mysql-connector-java-5.1.49.zip
cd mysql-connector-java-5.1.49
mv mysql-connector-java-5.1.49.jar ../

```

```
use dbtest;

CREATE TABLE user (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(320),
    last_updated INT(11) NOT NULL DEFAULT '0',
    is_deleted TINYINT(1) NOT NULL DEFAULT '0'
);


mysql> CREATE TABLE user (
    ->     id INT AUTO_INCREMENT PRIMARY KEY,
    ->     name VARCHAR(255) NOT NULL,
    ->     email VARCHAR(320),
    ->     last_updated INT(11) NOT NULL DEFAULT '0',
    ->     is_deleted TINYINT(1) NOT NULL DEFAULT '0'
    -> );
Query OK, 0 rows affected (0.32 sec)

mysql> insert into user values('','jacob','jacob@test.com','','');
Query OK, 1 row affected, 3 warnings (0.20 sec)

mysql> select * from user;
+----+-------+----------------+--------------+------------+
| id | name  | email          | last_updated | is_deleted |
+----+-------+----------------+--------------+------------+
|  1 | jacob | jacob@test.com |            0 |          0 |
+----+-------+----------------+--------------+------------+
1 row in set (0.19 sec)
```

**`mysql-demo.conf`**

```
input {
  jdbc {
  	
  	jdbc_driver_library => "/usr/share/logstash/logstash-core/lib/jars/mysql-connector-java-5.1.49.jar"
    jdbc_driver_class => "com.mysql.jdbc.Driver"
    jdbc_connection_string => "jdbc:mysql://db_hots:3306/dbtest?useSSL=false"
    # useSSL=false disable SSL the verify
    jdbc_user => admin
    jdbc_password => passoword
    #启用追踪，如果为true，则需要指定tracking_column
    use_column_value => true
    #指定追踪的字段，
    tracking_column => "last_updated"
    #追踪字段的类型，目前只有数字(numeric)和时间类型(timestamp)，默认是数字类型
    tracking_column_type => "numeric"
    #记录最后一次运行的结果
    record_last_run => true
    #上面运行结果的保存位置
    last_run_metadata_path => "jdbc-position.txt"
    statement => "SELECT * FROM user where last_updated >:sql_last_value;"
    schedule => " * * * * * *"
  }
}
output {
  elasticsearch {
    document_id => "%{id}"
    document_type => "_doc"
    index => "users"
    hosts => ["http://localhost:9200"]
  }
  stdout{
    codec => rubydebug
  }
}
```

```
$ logstash -f mysql-demo.conf
 FROM user where last_updated >0;
[2020-12-13T05:15:46,179][INFO ][logstash.inputs.jdbc     ][main][6b0ee2f856c602fc2c0158f436cb05489d0e96fe565728ffb597961d7dc64a68] (0.217996s) SELECT * FROM user where last_updated >0;
[2020-12-13T05:15:48,815][INFO ][logstash.inputs.jdbc     ][main][6b0ee2f856c602fc2c0158f436cb05489d0e96fe565728ffb597961d7dc64a68] (0.215261s) SELECT * FROM user where last_updated >0;
[2020-12-13T05:15:51,512][INFO ][logstash.inputs.jdbc     ][main][6b0ee2f856c602fc2c0158f436cb05489d0e96fe565728ffb597961d7dc64a68] (0.214389s) SELECT * FROM user where last_updated >0;
[2020-12-13T05:15:54,139][INFO ][logstash.inputs.jdbc     ][main][6b0ee2f856c602fc2c0158f436cb05489d0e96fe565728ffb597961d7dc64a68] (0.212874s) SELECT * FROM user where last_updated >0;
[2020-12-13T05:15:56,748][INFO ][logstash.inputs.jdbc     ][main][6b0ee2f856c602fc2c0158f436cb05489d0e96fe565728ffb597961d7dc64a68] (0.216092s) SELECT * FROM user where last_updated >0;
[2020-12-13T05:15:59,373][INFO ][logstash.inputs.jdbc     ][main][6b0ee2f856c602fc2c0158f436cb05489d0e96fe565728ffb597961d7dc64a68] (0.217051s) SELECT * FROM user where last_updated >0;
[2020-12-13T05:16:02,001][INFO ][logstash.inputs.jdbc     ][main][6b0ee2f856c602fc2c0158f436cb05489d0e96fe565728ffb597961d7dc64a68] (0.213808s) SELECT * FROM user where last_updated >0;
[2020-12-13T05:16:05,164][INFO ][logstash.inputs.jdbc     ][main][6b0ee2f856c602fc2c0158f436cb05489d0e96fe565728ffb597961d7dc64a68] (0.210873s) SELECT * FROM user where last_updated >0;
[2020-12-13T05:16:07,750][INFO ][logstash.inputs.jdbc     ][main][6b0ee2f856c602fc2c0158f436cb05489d0e96fe565728ffb597961d7dc64a68] (0.213390s) SELECT * FROM user where last_updated >0;
[2020-12-13T05:16:10,385][INFO ][logstash.inputs.jdbc     ][main][6b0ee2f856c602fc2c0158f436cb05489d0e96fe565728ffb597961d7dc64a68] (0.218651s) SELECT * FROM user where last_updated >0;
[2020-12-13T05:16:13,240][INFO ][logstash.inputs.jdbc     ][main][6b0ee2f856c602fc2c0158f436cb05489d0e96fe565728ffb597961d7dc64a68] (0.213114s) SELECT * FROM user where last_updated >0;
[2020-12-13T05:16:15,803][INFO ][logstash.inputs.jdbc     ][main][6b0ee2f856c602fc2c0158f436cb05489d0e96fe565728ffb597961d7dc64a68] (0.211320s) SELECT * FROM user where last_updated >0;
[2020-12-13T05:16:18,389][INFO ][logstash.inputs.jdbc     ][main][6b0ee2f856c602fc2c0158f436cb05489d0e96fe565728ffb597961d7dc64a68] (0.213701s) SELECT * FROM user where last_updated >0;
[2020-12-13T05:16:20,998][INFO ][logstash.inputs.jdbc     ][main][6b0ee2f856c602fc2c0158f436cb05489d0e96fe565728ffb597961d7dc64a68] (0.216809s) SELECT * FROM user where last_updated >0;
[2020-12-13T05:16:23,646][INFO ][logstash.inputs.jdbc     ][main][6b0ee2f856c602fc2c0158f436cb05489d0e96fe565728ffb597961d7dc64a68] (0.215923s) SELECT * FROM user where last_updated >0;
[2020-12-13T05:16:26,350][INFO ][logstash.inputs.jdbc     ][main][6b0ee2f856c602fc2c0158f436cb05489d0e96fe565728ffb597961d7dc64a68] (0.214398s) SELECT * FROM user where last_updated >0;
```

## **3、Demo**

### **3-1 插入新的数据**

```
mysql> insert into user values('','ham','ham@test.com',UNIX_TIMESTAMP() ,'');
Query OK, 1 row affected, 2 warnings (0.20 sec)

mysql> select * from user;
+----+-------+----------------+--------------+------------+
| id | name  | email          | last_updated | is_deleted |
+----+-------+----------------+--------------+------------+
|  1 | jacob | jacob@test.com |            0 |          0 |
|  2 | ham   | ham@test.com   |   1608174096 |          0 |
+----+-------+----------------+--------------+------------+
2 rows in set (0.20 sec)
```


```
$ logstash -f mysql-demo.conf

[2020-12-13T05:40:18,761][INFO ][logstash.inputs.jdbc     ][main][6b0ee2f856c602fc2c0158f436cb05489d0e96fe565728ffb597961d7dc64a68] (0.215617s) SELECT * FROM user where last_updated >1608174096;
{
           "email" => "ham@test.com",
            "name" => "ham",
      "@timestamp" => 2020-12-13T05:40:15.249Z,
              "id" => 2,
        "@version" => "1",
      "is_deleted" => false,
    "last_updated" => 1608174096
}
[2020-12-13T05:40:21,395][INFO ][logstash.inputs.jdbc     ][main][6b0ee2f856c602fc2c0158f436cb05489d0e96fe565728ffb597961d7dc64a68] (0.213273s) SELECT * FROM user where last_updated >1608174096;
[2020-12-13T05:40:25,689][INFO ][logstash.inputs.jdbc     ][main][6b0ee2f856c602fc2c0158f436cb05489d0e96fe565728ffb597961d7dc64a68] (0.217912s) SELECT * FROM user where last_updated >1608174096;
```


```
insert into user values('','mike','mike@test.com',UNIX_TIMESTAMP() ,'');
insert into user values('','kate','kate@test.com',UNIX_TIMESTAMP() ,'');
```


```
[2020-12-13T05:42:54,156][INFO ][logstash.inputs.jdbc     ][main][6b0ee2f856c602fc2c0158f436cb05489d0e96fe565728ffb597961d7dc64a68] (0.216099s) SELECT * FROM user where last_updated >1608174096;
{
           "email" => "mike@test.com",
            "name" => "mike",
      "@timestamp" => 2020-12-13T05:42:54.162Z,
              "id" => 3,
        "@version" => "1",
      "is_deleted" => false,
    "last_updated" => 1608174346
}

....
[2020-12-13T05:43:20,257][INFO ][logstash.inputs.jdbc     ][main][6b0ee2f856c602fc2c0158f436cb05489d0e96fe565728ffb597961d7dc64a68] (0.218078s) SELECT * FROM user where last_updated >1608174346;
{
           "email" => "kate@test.com",
            "name" => "kate",
      "@timestamp" => 2020-12-13T05:43:20.259Z,
              "id" => 4,
        "@version" => "1",
      "is_deleted" => false,
    "last_updated" => 1608174368
}
...
```

### **3-2 Check insterted user**


```
GET users/_search
```

```
{
  "took" : 28,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 3,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "users",
        "_type" : "_doc",
        "_id" : "2",
        "_score" : 1.0,
        "_source" : {
          "email" : "ham@test.com",
          "name" : "ham",
          "@timestamp" : "2020-12-13T05:40:15.249Z",
          "id" : 2,
          "@version" : "1",
          "is_deleted" : false,
          "last_updated" : 1608174096
        }
      },
      {
        "_index" : "users",
        "_type" : "_doc",
        "_id" : "3",
        "_score" : 1.0,
        "_source" : {
          "email" : "mike@test.com",
          "name" : "mike",
          "@timestamp" : "2020-12-13T05:42:54.162Z",
          "id" : 3,
          "@version" : "1",
          "is_deleted" : false,
          "last_updated" : 1608174346
        }
      },
      {
        "_index" : "users",
        "_type" : "_doc",
        "_id" : "4",
        "_score" : 1.0,
        "_source" : {
          "email" : "kate@test.com",
          "name" : "kate",
          "@timestamp" : "2020-12-13T05:43:20.259Z",
          "id" : 4,
          "@version" : "1",
          "is_deleted" : false,
          "last_updated" : 1608174368
        }
      }
    ]
  }
}
```

### **3-3 Update Data**

```
UPDATE user SET name="ham2", last=UNIX_TIMESTAMP() where id=2;
```

```
...
[2020-12-13T06:20:02,467][INFO ][logstash.inputs.jdbc     ][main][6b0ee2f856c602fc2c0158f436cb05489d0e96fe565728ffb597961d7dc64a68] (0.215766s) SELECT * FROM user where last_updated >1608174368;
{
           "email" => "ham@test.com",
            "name" => "ham2",
      "@timestamp" => 2020-12-13T06:20:02.542Z,
              "id" => 2,
        "@version" => "1",
      "is_deleted" => false,
    "last_updated" => 1608176401
}
...
```


### **3-3 Check Deleted User Data**



```
# 创建 alias，只显示没有被标记 deleted的用户
POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "users",
        "alias": "view_users",
         "filter" : { "term" : { "is_deleted" : false } }
      }
    }
  ]
}
```



**Set one user is_deleted as true**

```
UPDATE user SET is_deleted=1, last_updated=UNIX_TIMESTAMP() WHERE id=4;


select * from user;
+----+-------+----------------+--------------+------------+
| id | name  | email          | last_updated | is_deleted |
+----+-------+----------------+--------------+------------+
|  1 | jacob | jacob@test.com |            0 |          0 |
|  2 | ham2  | ham@test.com   |   1608176401 |          0 |
|  3 | mike  | mike@test.com  |   1608174346 |          0 |
|  4 | kate  | kate@test.com  |   1608213235 |          1 |
+----+-------+----------------+--------------+------------+
4 rows in set (0.20 sec)
```

```
...
{
      "@timestamp" => 2020-12-13T07:11:09.431Z,
            "name" => "kate",
              "id" => 4,
           "email" => "kate@test.com",
    "last_updated" => 1608213235,
        "@version" => "1",
      "is_deleted" => true
}
...
```




```
# 通过 Alias查询，查不到被标记成 deleted的用户
POST view_users/_search
{

}
```

**Output：kate disappear**

```
"hits" : [
      {
        "_index" : "users",
        "_type" : "_doc",
        "_id" : "3",
        "_score" : 1.0,
        "_source" : {
          "email" : "mike@test.com",
          "name" : "mike",
          "@timestamp" : "2020-12-13T05:42:54.162Z",
          "id" : 3,
          "@version" : "1",
          "is_deleted" : false,
          "last_updated" : 1608174346
        }
      },
      {
        "_index" : "users",
        "_type" : "_doc",
        "_id" : "2",
        "_score" : 1.0,
        "_source" : {
          "email" : "ham@test.com",
          "name" : "ham2",
          "@timestamp" : "2020-12-13T06:20:02.542Z",
          "id" : 2,
          "@version" : "1",
          "is_deleted" : false,
          "last_updated" : 1608176401
        }
      }
    ]
  }
```

**在只显示未被删除的用户的`view_users`查询**

```
POST view_users/_search
{
  "query": {
    "term": {
      "name.keyword": {
        "value": "kate"
      }
    }
  }
}
```

```
"hits" : [ ]
```


**在全部用户的`users`查询**

```
POST users/_search
{
  "query": {
    "term": {
      "name.keyword": {
        "value": "kate"
      }
    }
  }
}
```

```
"hits" : [
      {
        "_index" : "users",
        "_type" : "_doc",
        "_id" : "4",
        "_score" : 0.6931471,
        "_source" : {
          "@timestamp" : "2020-12-13T07:11:09.431Z",
          "name" : "kate",
          "id" : 4,
          "email" : "kate@test.com",
          "last_updated" : 1608213235,
          "@version" : "1",
          "is_deleted" : true
        }
      }
    ]
```

## **4、Q&A**


* 这种查表得方式应该不是很适用于生产环境的，监听binlog日志相对于生产环境来说更好点
	* **监听binlog确实有更多jdbc不具备的优点**


* 请问logstash这种轮询方式获取数据有没有啥性能问题？我们目前使用方式是kafka订阅binlog