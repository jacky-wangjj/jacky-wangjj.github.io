---
layout: post
title: Spark Streaming详解
date: 2019-04-23
tags: 大数据
---  
### Spark Streaming简介
Spark Streaming是Spark为需要即时处理收到的数据的应用而设计的模型。     

Spark Streaming使用**离散化流**（discretized stream）作为抽象表示，即DStream。DStream是随时间推移而收到的数据的序列。在内部，每个时间区间收到的数据都作为RDD存在，而DStream是由这些RDD所组成的序列。    

DStream可以从各种输入源创建，比如Flume、Kafka或者HDFS。创建出来的DStream支持两种操作，一种是转化操作（transformation），会生成一个新的DStream，另一种是输出操作（output operation），可以把数据写入外部系统中。DStream提供了许多与RDD所支持的操作相类似的操作支持，还增加了与事件相关的新操作，比如滑动窗口。    

Spark Streaming应用需要进行额外配置保证24/7不间断工作。**检查点**机制，也就是把数据存储到可靠文件系统（比如HDFS）上的机制，这是Spark Streaming用来实现不间断工作的主要方式。     

### 简单实例
Spark Streaming的Maven索引
```
groupId = org.apache.spark
artifactId = spark-streaming_2.10
version = 1.2.0
```
Scala流计算import声明    
```scala
import org.apache.spark.streaming.StreamingContext
import org.apache.spark.streaming.StreamingContext._
import org.apache.spark.streaming.dstream.DStream
import org.apache.spark.streaming.Duration
import org.apache.spark.streaming.Seconds
```
Java流计算import声明    
```java
import org.apache.spark.streaming.api.java.JavaStreamingContext;
import org.apache.spark.streaming.api.java.JavaDStream;
import org.apache.spark.streaming.api.java.JavaPairDStream;
import org.apache.spark.streaming.Duration;
import org.apache.spark.streaming.Durations;
```

StreamingContext是流计算功能的主要入口，StreamingContext会在底层创建出SparkContext，用来处理数据。其构造函数接收用来指定多长时间处理一次新数据的**批次间隔**（batch interval）作为输入。    

用 Scala 进行流式筛选，打印出包含“error”的行
```scala
// 从SparkConf创建StreamingContext并指定1秒钟的批处理大小
val ssc = new StreamingContext(conf, Seconds(1))
// 连接到本地机器7777端口上后，使用收到的数据创建DStream
val lines = ssc.socketTextStream("localhost", 7777)
// 从DStream中筛选出包含字符串"error"的行
val errorLines = lines.filter(_.contains("error"))
// 打印出有"error"的行
errorLines.print()

// 启动流计算环境StreamingContext并等待它"完成"
ssc.start()
// 等待作业完成
ssc.awaitTermination()
```

用 Java 进行流式筛选，打印出包含“error”的行
```java
// 从SparkConf创建StreamingContext并指定1秒钟的批处理大小
JavaStreamingContext jssc = new JavaStreamingContext(conf, Durations.seconds(1));
// 以端口7777作为输入来源创建DStream
JavaDStream<String> lines = jssc.socketTextStream("localhost", 7777);
// 从DStream中筛选出包含字符串"error"的行
JavaDStream<String> errorLines = lines.filter(new Function<String, Boolean>() {
  public Boolean call(String line) {
    return line.contains("error");
  }});
// 打印出有"error"的行
errorLines.print();

// 启动流计算环境StreamingContext并等待它"完成"
jssc.start();
// 等待作业完成
jssc.awaitTermination();
```

这只是设定好了要进行的计算，系统收到数据时计算就会开始。要开始接收数据，必须显式调用StreamingContext的`start()`方法。这样，Spark Streaming就会开始把Spark作业不断交给下面的SparkContext去调度执行。执行会在另一个线程中进行，所以需要调用awaitTermination来等待流计算完成，来防止应用退出。    

一个StreamingContext只能启动一次，所以只有在配置好所有DStream以及所需要的输出操作之后才能启动。    

在Linux/Mac操作系统上运行流计算应用并提供数据
```shell
spark-submit --class com.oreilly.learningsparkexamples.scala.StreamingLogInput $ASSEMBLY_JAR local[4]

nc localhost 7777 # 使你可以键入输入的行来发送给服务器
<此处是你的输入>
```

### 架构与抽象
Spark Streaming使用“微批次”的架构，把流式计算当作一系列连续的小规模批处理来对待。Spark Streaming 从各种输入源中读取数据，并把数据分组为小的批次。新的批次按均匀的时间间隔创建出来。在每个时间区间开始的时候，一个新的批次就创建出来，在该区间内收到的数据都会被添加到这个批次中。在时间区间结束时，批次停止增长。时间区间的大小是由批次间隔这个参数决定的。**批次间隔一般设在 500 毫秒到几秒之间**，由应用开发者配置。每个输入批次都形成一个 RDD，以 Spark 作业的方式处理并生成其他的 RDD。处理的结果可以以批处理的方式传给外部系统。    

Spark Streaming的编程抽象是离散化流，也就是DStream。它是一个RDD序列，每个RDD代表数据流中一个时间片内的数据。    

DStream及其转化关系    
![](http://localhost:4000/images/blog/machine-learning/spark-streaming-DStream-eg-1.png#pic_center)    
![](https://jacky-wangjj.github.io/images/blog/machine-learning/spark-streaming-DStream-eg-1.png#pic_center)

Spark作业可以在 http://localhost:4040 查看。    

### Spark Streaming综合实例
- [整合Kafka和Spark Streaming](https://cloud.tencent.com/developer/article/1017077) 从kafka获取数据，经过spark streaming处理后存入mysql数据库。        

- [Spark Streaming完整实例](https://blog.csdn.net/awj321000/article/details/74223899) 获取网络数据或监控HDFS目录数据经Spark Streaming处理后打印，包括检查点机制。      

- [Spark Streaming从Kafka获取数据再写入HBase](https://forum.huawei.com/enterprise/zh/thread-451863.html)     

官方文档：[Spark Streaming Programming Guide](https://spark.apache.org/docs/0.9.1/streaming-programming-guide.html)    
