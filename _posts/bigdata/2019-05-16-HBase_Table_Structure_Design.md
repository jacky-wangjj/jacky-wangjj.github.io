---
layout: post
title: HBase表结构设计
date: 2019-05-16
tags: 大数据
---  
### HBase基本介绍
只创建一次HTable实例，一般在应用程序开始时创建；    
使用多个HTable实例时，考虑使用HTablePool类；    
所有的修改操作只保证行级别的原子性。    
数据库基本操作CRUD(Create, Read, Update, Delete)具体指增、查、改、删。    
HBase使用行键、列族、列限定符、时间戳指向一个单元格的值。    

**数据的版本化**      
hbase能为一个单元格（一个特定列的值）存储多个版本的数据，每个版本使用一个时间戳，时间戳是一个长整型值，以毫秒为单位。    
scan操作和get操作只会返回最后的（最新的）版本，如果将调用参数值设为Integer.MAX_VALUE就可以获得所有的版本。    

#### Put操作
单个Put插入        
```java
void put(Put put) throws IOException

Put(byte[] row)
Put(byte[] row, RowLock rowLock)
Put(byte[] row, long ts)
Put(byte[] row, long ts, RowLock rowLock)
```

创建Put实例，向实例添加数据，添加数据的方法如下：      
```java
Put add(byte[] family, byte[] qualifier, byte[] value)
Put add(byte[] family, byte[] qualifier, long ts, byte[] value)
Put add(KeyValue kv) throws IOException
```
KeyValue是HBase在存储架构中最底层的类，每个KeyValue实例包含其完整地址（行键、列族、列限定符及时间戳）和实际数据。      

检查是否存在特定单元格
```java
boolean has(byte[] family, byte[] qualifier)
boolean has(byte[] family, byte[] qualifier, long ts)
boolean has(byte[] family, byte[] qualifier, byte[] value)
boolean has(byte[] family, byte[] qualifier, long ts, byte[] value)
```

![](https://jacky-wangjj.github.io/images/blog/bigdata/hbase-put-funcs.png#pic_center)

向HBase插入数据的实例应用
```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.client.HTable;
import org.apache.hadoop.hbase.client.Put;
import org.apache.hadoop.hbase.util.Bytes;

import java.io.IOException;

public class PutExample {
  public static void main(String[] args) throws IOException {
    Configuration conf = HBaseConfiguration.create();
    HTable table = new HTable(conf, "testtable");
    Put put = new Put(Bytes.toBytes("row1"));
    put.add(Bytes.toBytes("conlfam1"), Bytes.toBytes("qual1"), Bytes.toBytes("val1"));
    put.add(Bytes.toBytes("conlfam1"), Bytes.toBytes("qual2"), Bytes.toBytes("val2"));
    table.put(put);
  }
}
```

Put列表插入
```java
void put(List<Put> puts) throws IOException
```

使用列表向HBase中添加数据
```java
List<Put> puts = new ArrayList<Put>();
Put put1 = new Put(Bytes.toBytes("row1"));
put1.add(Bytes.toBytes("colfam1"), Bytes.toBytes("qual1"), Bytes.toBytes("val1"));
puts.add(put1);

Put put2 = new Put(Bytes.toBytes("row2"));
put2.add(Bytes.toBytes("colfam1"), Bytes.toBytes("qual1"), Bytes.toBytes("val2"));
puts.add(put2);

Put put3 = new Put(Bytes.toBytes("row3"));
put3.add(Bytes.toBytes("colfam1"), Bytes.toBytes("qual2"), Bytes.toBytes("val3"));
puts.add(put3);

table.put(puts);
```
检查写：是一种特别的put调用，可以保证自身操作的原子性
```java
boolean checkAndPut(byte[] row, byte[] family, byte[] qualifier, byte[] value, Put put) throws IOException
```

**写缓冲区**    
HBase的API配备了一个客户端的写缓冲区，缓冲区负责收集put操作，然后调用RPC操作一次性将put送往服务器，默认情况下，客户端缓冲区是禁用的，可通过将自动刷写设置为false来激活缓冲区：
```java
table.setAutoFlush(false)
```
调用`flushCommits()`方法将所有的修改传送到远程服务器。    
通过以下调用来配置客户端写缓冲区的大小：
```java
long getWriteBufferSize()
void setWriteBufferSize(long writeBufferSize) throw IOException
```
或修改hbase-site.xml
```xml
<property>
  <name>hbase.client.write.buffer</name>
  <value>20971520</value>
</property>
```
使用客户端写缓冲区
```java
HTable table = new HTable(conf, "testtable");
System.out.println("Auto flush:"+table.isAutoFlush());

table.setAutoFlush(false);

Put put1 = new Put(Bytes.toBytes("row1"));
put1.add(Bytes.toBytes("colfam1"), Bytes.toBytes("qual1"), Bytes.toBytes("val1"));
table.put(put1);

Put put2 = new Put(Bytes.toBytes("row2"));
put2.add(Bytes.toBytes("colfam2"), Bytes.toBytes("qual2"), Bytes.toBytes("val2"));
table.put(put2);

table.flushCommits();
```

#### Get操作
单行get
```java
Result get(Get get) throws IOException

Get(byte[] row)
Get(byte[] row, RowLock rowLock)
```
可以同多种标准筛选目标数据    
```java
Get addFamily(byte[] family)
Get addColumn(byte[] family, byte[] qualifier)
Get setTimeRange(long minStamp, long maxStamp) throws IOException
Get setTimeStamp(long timestamp)
Get setMaxVersions()
Get setMaxVersions(int maxVersions) throws IOException
```

![](https://jacky-wangjj.github.io/images/blog/bigdata/hbase-get-funcs.png#pic_center)

从hbase中获取数据的应用
```java
Configuration conf = HBaseConfiguration.create();
HTable table = new HTable(conf, "testtable");
Get get = new Get(Bytes.toBytes("row1"));
get.addColumn(Bytes.toBytes("colfam1"), Bytes.toBytes("qual1"));
Result result = table.get(get);
byte[] val = result.getValue(Bytes.toBytes("colfam1"), Bytes.toBytes("qual1"));
System.out.println("value: "+ Bytes.toString(val));
```

**Result类**
Result类提供如下方法
```java
byte[] getValue(byte[] family, byte[] qualifier)
byte[] value()
byte[] getRow()
int size()
boolean isEmpty()
KeyValue[] raw()
List<KeyValue> list()
```

Get列表
```java
Result[] get(List<Get> gets) throws IOException
```
使用Get实例列表从HBase中获取数据
```java
byte[] cf1 = Bytes.toBytes("colfam1");
byte[] qf1 = Bytes.toBytes("qual1");
byte[] qf2 = Bytes.toBytes("qual2");
byte[] row1 = Bytes.toBytes("row1");
byte[] row2 = Bytes.toBytes("row2");

List<Get> gets = new ArrayList<Get>();

Get get1 = new Get(row1);
get1.addColumn(cf1, qf1);
gets.add(get1);

Get get2 = new Get(row2);
get2.addColumn(cf1, qf1);
gets.add(get2);

Get get3 = new Get(row2);
get3.addColumn(cf1, qf2);
gets.add(get3);

Result[] results = table.get(gets);
for (Result result: results) {
  String row = Bytes.toString(result.getRow());
  System.out.print("Row: "+row+" ");
  byte[] val = null;
  if (result.containsColumn(cf1, qf1)) {
    val = result.getValue(cf1, qf1);
    System.out.println("Value: "+Bytes.toString(val));
  }
  if (result.containsColumn(cf1, qf2)) {
    val = result.getValue(cf1, qf2);
    System.out.println("Value: "+Bytes.toString(val));
  }
}

for (Result result: results) {
  for (KeyValue kv: result.raw()) {
    System.out.println("Row: "+Bytes.toString(kv.getRow())+" Value: "+Bytes.toString(kv.getValue()));
  }
}
```

#### Delete操作
单行delete
```java
void delete(Delete delete) throws IOException

Delete(byte[] row)
Delete(byte[] row, long timestamp, RowLock rowLock)
```

缩小要删除的给定行中涉及的数据范围
```java
Delete deleteFamily(byte[] family)
Delete deleteFamily(byte[] family, long timestamp)
Delete deleteColumns(byte[] family, byte[] qualifier)
Delete deleteColumns(byte[] family, byte[] qualifier， long timestamp)
Delete deleteColumn(byte[] family, byte[] qualifier)
Delete deleteColumn(byte[] family, byte[] qualifier, long timestamp)
void setTimestamp(long timestamp)
```

从HBase中删除数据的应用实例
```java
Delete delete = new Delete(Bytes.toBytes("row1"));

delete.setTimestamp(1);

delete.deleteColumn(Bytes.toBytes("colfam1"), Bytes.toBytes("qual1"), 1);

delete.deleteColumns(Bytes.toBytes("colfam2"), Bytes.toBytes("qual1"));
delete.deleteColumns(Bytes.toBytes("colfam2"), Bytes.toBytes("qual3"), 15);

delete.deleteFamily(Bytes.toBytes("colfam3"));
delete.deleteFamily(Bytes.toBytes("colfam3")， 3);

table.delete(delete);
table.close();
```

Delete列表
```java
void delete(List<Delete> deletes) throws IOException
```

删除值列表的应用实例
```java
List<Delete> deletes = new ArrayList<Delete>();

Delete delete1 = new Delete(Bytes.toBytes("row1"));
delete1.setTimestamp(4);
deletes.add(delete1);

Delete delete2 = new Delete(Bytes.toBytes("row2"));
delete2.deleteColumn(Bytes.toBytes("colfam1"), Bytes.toBytes("qual1"));
delete2.deleteColumns(Bytes.toBytes("colfam2"), Bytes.toBytes("quual3"), 5);
deletes.add(delete2);

Delete delete3 = new Delete(Bytes.toBytes("row3"));
delete3.deleteFamily(Bytes.toBytes("colfam1"));
delete3.deleteFamily(Bytes.toBytes("colfam2"), 3);
deletes.add(delete3);

table.delete(deletes);
table.close();
```
原子删除操作
```java
boolean checkAndDelete(byte[] row, byte[] family, byte[] qualifier, byte[] value, Delete delete) throws IOException
```

#### 批量处理操作
```java
void batch(List<Row> actions, Object[] results) throws IOException,InterruptedException
Object[] batch(List<Row> actions) throws IOException,InterruptedException
```

使用批量处理操作的应用实例
```java
private final static byte[] ROW1 = Bytes.toBytes("row1");
private final static byte[] ROW2 = Bytes.toBytes("row2");
private final static byte[] COLFAM1 = Bytes.toBytes("colfam1");
private final static byte[] COLFAM2 = Bytes.toBytes("colfam2");
private final static byte[] QUAL1 = Bytes.toBytes("qual1");
private final static byte[] QUAL2 = Bytes.toBytes("qual2");

List<Row> batch = new ArrayList<Row>();

Put put = new Put(ROW2);
put.add(COLFAM2,QUAL1,Bytes.toBytes("val5"));
batch.add(put);

Get get1 = new Get(ROW1);
get1.addColumn(COLFAM1,QUAL1);
batch.add(get1);

Delete delete = new Delete(ROW1);
delete.deleteColumns(COLFAM1,QUAL2);
batch.add(delete);

Object[] results = new Object[batch.size()];
try {
  table.batch(batch, results);
} catch(Exception e) {
  System.err.println("Error: "+e);
}

for (int i=0; i<results.length; i++) {
  System.out.println("Result["+i+"]: "+results[i]);
}
```

#### Scan操作

![](https://jacky-wangjj.github.io/images/blog/bigdata/hbase-scan-funcs.png#pic_center)

使用扫描器获取表中的数据
```java
Scan scan1 = new Scan();
ResultScanner scanner1 = table.getScanner(scan1);
for (Result res: scanner1) {
  System.out.println(res);
}
scanner1.close();

Scan scan2 = new Scan();
scan2.addFamily(Byte.toBytes("colfam1"));
ResultScanner scanner2 = table.getScanner(scan2);
for (Result res: scanner2) {
  System.out.println(res);
}
scanner2.close();

Scan scan3 = new Scan();
scan3.addColumn(Bytes.toBytes("colfam1"), Bytes.toBytes("col-5")).
  addColumn(Bytes.toBytes("colfam2"), Bytes.toBytes("col-33")).
  setStartRow(Bytes.toBytes("row10")).
  setStopRow(Bytes.toBytes("row20"));
ResultScanner scanner3 = table.getScanner(scan3);
for (Result res: scanner3) {
  System.out.println(res);
}
scanner3.close();
```

默认每个next()调用都会生成一个单独的RPC请求，扫描器缓存使得一次RPC请求可以获取多行数据。     
默认情况缓存是关闭的，可以在表层面或扫描层面打开：      
调用HTable方法设置表级扫描器缓存
```java
void setScannerCaching(int scannerCaching)
int getScnnerCaching()
```
或在hbase-site.xml中配置
```xml中配置
<property>
  <name>hbase.client.scanner.caching</name>
  <value>10</value>
</property>
```
调用Scan类的方法设置扫描级的缓存
```java
void setCaching(int caching)
int getCaching()
```

缓存是面向行一级的操作，而批量则是面向列一级的操作。批量可以让用户选择每一次ResultScanner实例的next()操作要取回多少列。        

在扫描中使用缓存和批量参数
```java
caching = 5;
batch = 10;
Scan scan = new Scan();
scan.setCaching(caching);
scan.setBatch(batch);
ResultScanner scanner = table.getScanner(scan);
```
caching和batch影响RPC请求数。         
![](https://jacky-wangjj.github.io/images/blog/bigdata/hbase-scan-batch-caching.png#pic_center)

#### 过滤器
使用过滤器返回包含特定列中特定值的行
```java
SingleColumnValueFilter filter = new SingleColumnValueFilter(
                                      Bytes.toBytes("colfam1"),
                                      Bytes.toBytes("col-5"),
                                      CompareFilter.CompareOp.NOT_EQUAL,
                                      new SubStringComparator("val-5"));
filter.setFilterIfMissing(true);

Scan scan = new Scan();
scan.setFilter(filter);
ResultScanner scanner = table.getScanner(scan);
for (Result result : scanner) {
  for (KeyValue kv : result.raw()) {
    System.out.println("kv: "+kv+", value:"+Bytes.toString(kv.getValue));
  }
}
scanner.close();
```

#### hbase表结构JSON形式理解
![](https://jacky-wangjj.github.io/images/blog/bigdata/hbase-table-json.png#pic_center)

#### 使用HTablePool来共享HTable实例
```java
Configuration conf = HBaseConfiguration.create();
//创建表实例池，并保留5个实例
HTablePool pool = new HTablePool(conf, 5);

HTableInterface[] tables = new HTableInterface[10];
for (int n=0; n<10; n++) {
  //获取10个实例，超出容量5个。
  tables[n] = pool.getTable("testtable");
  System.out.println(Bytes.toString(tables[n].getTableName()));
}
//向表实例池返还HTable实例，其中的5个会被保留，多余的会被丢弃。
for (int n=0; n<5; n++) {
  pool.putTable(tables[n]);
}
//关闭整个表实例池，释放其中保留的表实例引用。
pool.closeTablePool("testtable");
```
### Spring Boot集成HBase
[spring-boot-starter-hbase集成HBase](https://zhuanlan.zhihu.com/p/60290536)

### HBase数据备份
[HBase数据备份方案](https://zhuanlan.zhihu.com/p/43442221)

### HBase行键的设计
查询数据时行键必需已知么？    
行键的设计需使得所有数据均匀的分布在集群的RegionServer上；     
一起查询的数据可以放入同一个列簇中；    
hbase不建议使用过多的列簇（3个以内）；    

rowkey设计：[bucket][timestamp][分类关键字]      
面向时间的扫描非常重要；    
```java
long bucket = timestamp % numRegionServer
```

**反转timestamp**    
如果最重要的是找寻最近的事件，那么反转时间戳（timestamp = Long.MAX_VALUE - timestamp）是有效的，只需scan即可获取最近记录。     

https://www.cnblogs.com/frankdeng/p/9310356.html     
https://www.cnblogs.com/JingJ/p/4521245.html     
https://www.cnblogs.com/cxzdy/p/5118456.html     

**用户应当尽量将需要查询的维度或信息存储在行键中，因为用它筛选数据的效率最高。**      

MD5之类的散列函数将行键分散到所有的region服务器上，随机化的方式很适合每次只读取一行数据的应用，如果用户的数据不需要连续扫描而只需随机读取，用户就可以使用这种策略。    

使用布隆过滤器的好处是：用户可以立即判断一个文件中是否包含特定的行键。这个过滤器的特性是：如果这个文件不包含这个行，它会给你一个明确的答复；但是如果文件包含这一行时，答复却可能有误，即它声称文件包含这个数据而实际却并非如此。通常有1%的块会被错误的加载和检查。     

如果用户定期修改所有行，那么大部分的存储文件都将包含用户查找的数据。这种场景不适合使用布隆过滤器。但是如果用户批量更新数据，使得一行数据每次只被写入到少数几个存储文件中，那么过滤器就能够为减少整个系统I/O操作的数量发挥很大作用。     

[RowKey设计指南](https://yq.aliyun.com/articles/685888)           
[Hbase最佳实践之列簇设计优化](https://zhuanlan.zhihu.com/p/38359555)        

### HBase表结构设计原则
好的行键设计可以避免数据倾斜，并能使常用查询得到优化，加快常用查询。        
列与列族，将属性相近的列放到同一个列族中，将常被一起查询的列放到一个列族中。     

### HBase表结构设计实例
https://www.ibm.com/developerworks/cn/analytics/library/ba-1604-hbase-develop-practice/index.html     
http://www.dataguru.cn/thread-144977-1-1.html     

[HBase在滴滴出行的应用场景和最佳实践](https://zhuanlan.zhihu.com/p/65435550)        

#### HBase预分区    
http://www.louisvv.com/archives/1989.html

http://ju.outofmemory.cn/entry/137374
