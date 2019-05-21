---
layout: post
title: 本地文件系统支持大数据组件情况
date: 2019-05-21
tags: 大数据
---  
### hbase standalone模式
hbase standalone运行在本地文件系统       
[官网详解](https://hbase.apache.org/book.html#quickstart)

### hive
hive必须使用hadoop

### zookeeper
[官网安装指导](https://zookeeper.apache.org/doc/current/zookeeperStarted.html)

### spark
spark run on standalone
[官网详解](http://spark.apache.org/docs/latest/spark-standalone.html)

### sqoop
sqoop依赖Hadoop命令      
You invoke Sqoop through the program launch capability provided by Hadoop. The sqoop command-line program is a wrapper which runs the bin/hadoop script shipped with Hadoop. If you have multiple installations of Hadoop present on your machine, you can select the Hadoop installation by setting the $HADOOP_COMMON_HOME and $HADOOP_MAPRED_HOME environment variables.      

### kafka
Kafka uses ZooKeeper     
[官网资料](http://kafka.apache.org/quickstart)

### NoSQL
- hadoop hbase       
对于普通的scan和基于行的get等基本查询，性能完全不是问题，只是只提供裸的api,易用性上是短板，可扩展性方面是最强的，其次坐上了Hadoop的快车，社区发展很快，各种基于其上的开源产品不少，来解决诸如join、聚集运算等复杂查询。     

- Mongodb        
分布式nosql，具备了区别mysql的最大亮点：可扩展性。mongodb 最新引人的莫过于提供了sql接口，是目前nosql里最像mysql的，只是没有ACID的特性，发展很快，支持了索引等特性，上手容易，对于数据量远超内存限制的场景来说，还需要慎重。          

- redis cluster       
in memory key-value store，同时提供了更加丰富的数据结构和运算的能力，成功用法是替代memcached，通过checkpoint和commit log提供了快速的宕机恢复，同时支持replication提供读可扩展和高可用。        

- leveldb         
真正基于磁盘的key-value storage, 模型单一简单，数据量不受限于内存大小，数据落盘高可靠，Google的几位大神出品的精品，LSM模型天然写优化，顺序写盘的方式对于新硬件ssd再适合不过了，不足是仅提供了一个库，需要自己封装server端。      

- bigtable开源实现 hypertable         

- Amazon DynamoDB        

- Google bigtable       

- Facebook Cassandra         

- 美团 Cellar

参考链接：       
https://www.cnblogs.com/vajoy/p/5471308.html        

### 数据集成工具
- kettle        
一款国外开源的ETL工具，纯java编写，可以在window、linux、Unix上运行，数据抽取高效稳定。      
[kettle使用教程](http://www.kettle.net.cn/1728.html)      

- sqoop         
依赖map-reduce计算框架     

- dataX       
DataX 是阿里巴巴集团内被广泛使用的离线数据同步工具/平台，实现包括 MySQL、Oracle、HDFS、Hive、OceanBase、HBase、OTS、ODPS 等各种异构数据源之间高效的数据同步功能。      
依赖python2.x      
[GitHub地址](https://github.com/alibaba/DataX)

- StreamSets       
[官网资料](https://streamsets.com/documentation/datacollector/latest/help/datacollector/UserGuide/Installation/InstallationAndConfig.html#concept_gbn_4lv_1r)      

[数据集成之 kettle、sqoop、datax、streamSets 比较](https://my.oschina.net/peakfang/blog/2056426)        
kettle速度较慢，但有web界面      
dataX速度较kettle快。     

### 集群管理工具
- Ambari
Ambari + had     
Hortonworks公司      

- Cloudera Manager
Cloudera Manager + CDH
Cloudera公司      

- Hue
CDH专门的一套web管理器。      
[官网](http://gethue.com/)

### 总结
综上分析，如果不安装Hadoop搭建本地大数据时，可以安装：      
zookeeper、kafka、hbase standalone模式、spark standalone模式；        
数据集成工具可以使用kettle、dataX、StreamSets；       
集群管理工具使用Ambari、Cloudera Manager；       
NoSQL数据库可选择hbase、redis、MongoDB、leveldb；     
