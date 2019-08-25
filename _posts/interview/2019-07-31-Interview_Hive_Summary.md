---
layout: post
title: 大数据面试之Hive知识点
date: 2019-07-31
tags: interview
---
### HiveSQL练习题

### Hive复杂数据类型
- arrays: ARRAY<data_type>
- maps: MAP<primitive_type, data_type>
- structs: STRUCT<col_name : data_type [COMMENT col_comment], ...>
- union: UNIONTYPE<data_type, data_type, ...>

FIELDS TERMINATED BY表示字段与字段之间的分隔符
COLLECTION ITEMS TERMINATED BY表示一个字段中各个item之间的分隔符[可用于array和struct类型]
MAP KEYS TERMINATED BY表示map类型中的key/value的分隔符[可用于map类型]
```shell
# 创建表
create table union_testnew(
  foo uniontype<int, double, string, array<string>, map<string, string>>
)
row format delimited
collection items terminated by ','
map keys terminated by ':'
lines terminated by '\n'
stored as textfile;

# 数据准备
[root@master wadeyu]# vim union_test.log
  1 0,1
  2 1,3.0
  3 2,world
  4 3,wade:tom:polly
  5 4,k1^Dv1:k2^Dv2

# 导入数据
hive (badou)> load data local inpath './union_test.log' overwrite into table union_testnew;

# 查询数据
hive (badou)> select * from union_testnew;
OK
union_testnew.foo
{0:1}
{1:3.0}
{2:"world"}
{3:["wade","tom","polly"]}
{4:{"k1":"v1","k2":"v2"}}
Time taken: 0.225 seconds, Fetched: 5 row(s)
```

[array一行转多行](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+LateralView)
explode(array):列表中的每个元素生成一行
explode(map):map中每个key-value对生成一行，key为一列，value为一列。
```sql
select explode(arrays) as col from tbl;
select col from tbl lateral view explode(arrays) tableAlias as col;
```

使用concat_ws()和collect_set函数进行多行转一行
```shell
# 借用concat_ws()和collect_set()函数进行相同列的重复数据转换
# collect_set()函数可以将相关列合并成array<>类型；concat_ws()函数会将array<>类型根据指定的分隔符进行合并
## 示例数据
hive> select * from tmp_jiangzl_test;
tmp_jiangzl_test.col1   tmp_jiangzl_test.col2   tmp_jiangzl_test.col3
a       b       1
a       b       2
a       b       3
c       d       4
c       d       5
c       d       6
## 对于以上数据，我们可以将col3列根据列col1和col2进行合并
hive> select col1,col2,concat_ws(',',collect_set(col3)) from tmp_jiangzl_test group by col1,col2;
col1    col2    _c2
a       b       1,2,3
c       d       4,5,6
```

struct类型的使用
```shell
# 元数据格式
1,zhou:30
2,yan:30
3,chen:20
# 相关数据库结构
hive> create table test-struct(id INT, info struct<name:STRING, age:INT>)
    > ROW FORMAT DELIMITED FIELDS TERMINATED BY ','
    > COLLECTION ITEMS TERMINATED BY ':';
# 加载数据
hive> LOAD DATA LOCAL INPATH '/home/work/data/test5.txt' INTO TABLE test-struct;
# 查询相关数据
hive> select info.age from test-struct;
Total MapReduce jobs = 1
......
Total MapReduce CPU Time Spent: 490 msec
OK
30
30
```

map类型的使用
```shell
# 原始数据格式
1       job:80,team:60,person:70
2       job:60,team:80
3       job:90,team:70,person:100

# map结构的表结构创建
hive> create table employee(id string, perf map<string, int>)
    > ROW FORMAT DELIMITED
    > FIELDS TERMINATED BY '\t'
    > COLLECTION ITEMS TERMINATED BY ','
    > MAP KEYS TERMINATED BY ':';

# 数据导入
hive>  LOAD DATA LOCAL INPATH '/home/work/data/test7.txt' INTO TABLE employee;

# 数据查询
hive> select perf['person'] from employee;
Total MapReduce jobs = 1
......
Total MapReduce CPU Time Spent: 460 msec
OK
70
NULL

# 使用explode()函数查询
hive> select explode(perf) as (p_name,p_score) from employee limit 4;
OK
job 80
team 60
person 70

# 使用explode()和lateral view结合查询
hive> select id,p_name,p_score from employee lateral view explode(perf) perf as p_name,p_score limit 3;
OK
1 job 80
1 team 60
1 person 70

# 使用size()函数查看map结构中的键值对个数[也可查看array中的元素个数]
hive> select size(perf) from employee
3
2
3
```

|          函数名          |                         作用描述                          |
| ------------------------ | --------------------------------------------------------- |
|         array()          |              将函数内容转换成一个array<>类型              |
|   split(array, split)    |       将array<>类型安装split分隔符进行分割成字符串        |
|        explode()         | array数据类型作为输入，对数组中数据进行迭代，返回多行结果 |
|      collect_set()       |        将某字段的值进行去重汇总，产生array类型字段        |
|      collect_list()      |          同collect_set()，但是不会对字段进行去重          |
| concat_ws(split, struct) |        将struct类型的字段按照split进行分割成字符串        |
|   cast(column as type)   |           转换数据类型（column列转换为type类型            |

```shell
## array()函数可以将一列输入转换成一个数组输出
hive> select array(1,2,3) from xuxuebiao;
OK
[1,2,3]
[1,2,3]

## explode()函数以array数据类型作为输入，对数组中数据进行迭代，返回多行结果
hive> select explode(array(1,2,3)) from xuxuebiao;
OK
1
2
3
## 使用explode()函数查看array中的某个元素
hive> select * from appinfo LATERAL VIEW explode(ips) tmpappinfo  AS realid         where realid ='10.0.0.125' ;

## collect_set函数
### 该函数的作用是将某字段的值进行去重汇总，产生Array类型字段
hive> select * from test;
OK
1       A
1       C
1       B
hive>  select id,collect_set(name) from test group by id;
OK
1       ["A","C","B"]
```

### Hive内部表与外部表
##### 内部表
内部表的数据存储在数据仓库中：/usr/hive/warehouse。
```sql
# 创建表
create table if not exists db.tbl (
  col1 string,
  ...)
  row format delimited
  fields terminated by '\t';

# 数据库授权
grant create on database dbname to user hadoop;

# 导入数据
load data inpath 'hdfs://master/path/file.csv' overwrite into table db.tablename;
load data local inpath '/home/file.csv' overwrite into table db.tablename;

# 导出查询结果
insert overwrite local directory '/home/result'
row format delimited fields terminated by '\t'
select * from db.tablename;
```

##### 外部表
外部表的数据可以存储在其他目录，只保存数据的元数据信息。
```sql
create external table if not exists db.tbl (
  col1 string,
  ...
  )
  row format delimited
  fields terminated by '\t';
```

##### 视图表
视图表是一个虚表，不存储数据，用来简化复杂的查询。

可通过如下命令查看：
```sql
desc extended tablename;
desc formatted tablename;
```

常用hive sql
```shell
# 修改列名。change只修改hive元数据
## 这个命令可以修改表的列名，数据类型，列注释和列所在的位置顺序，FIRST将列放在第一列，AFTER col_name将列放在col_name后面一列
hive> ALTER TABLE db.tbl CHANGE oldcol newcol int comment 'some 注释' AFTER col3;

# 修改表结构。replace删除已经存在的列并添加新列
ALTER TABLE db.tbl replace columns (appname string,level string,leader string,appline string,dep string,ips array<string>);

## 增加表的列字段(默认增加到最后一列，可以使用change column 来调整位置)
hive> alter table appinfo add columns (appclass string comment 'app_perf_class');
```
详见https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-AlterTable/Partition/Column

### Hive分区与分桶
分区和分桶的主要作用是使得hive可以纵向扩展，即可以无限扩展行，另外一个作用是可以使得查询时间变短，当使用特定分区或分桶中的数据进行查询时，只需要扫描指定分区或分桶中的文件，从而优化查询速度。生产中常使用天作为分区，数据量大时可能使用小时作为分区。
```shell
# 手动添加分区
alter table tablename add partition(dt='20190810');

# 删除分区
alter table tablename drop partition(dt='20190810');

# 查询分区表时若没有加分区过滤，会禁止提交这个任务（strict方式每次查询必须制定分区）
set hive.mapred.mode = strict | nostrict;

# 查看分区
show partitions tablename;

desc extended tablename;
```

多重分区
```shell
# 创建多重分区表
create table tablename(
  col1 string,
  ...
  )
  partitioned by (year string, month string, day string)
  row format delimited
  fields terminated by '\t'
  lines terminated by '\n';

# 插入数据
insert into table tablename partition (year='2019',month='08',day='10') values ('col1',...);

# 查看分区数据
select * from tablename where year='2019';
```

### 常用函数

|     函数名      |             作用描述             |
| --------------- | -------------------------------- |
| round()/floor() | 可以将double类型转换为bigint类型 |
|      abs()      |         返回数值的绝对值         |
|     ucase()     |      将字符串转换成大写字母      |
|    reverse()    |         将字符串进行翻转         |
|    concat()     |          连接多个字符串          |
|      sum()      |               求和               |
|      avg()      |              平均值              |
|   min()/max()   |          最小值/最大值           |

### 常用的条件判断及数据清洗函数
hive处理数据过程中，常使用一些条件判断以及默认值处理函数。

|                   函数名                    |                   作用描述                   |
| ------------------------------------------- | -------------------------------------------- |
| if(test condition, true value, false value) | 条件判断，满足即为true值，不满足即为false值  |
|         case when ... then ... end          |                  多条件判断                  |
|                 parse_url()                 | 通常用于清洗url相关函数，提供常用url解析功能 |
|              parse_url_tuple()              |                     同上                     |
|              regexp_replace()               |                正则表达式替换                |
|              regexp_extract()               |                正则表达式解析                |
|            coalesce(column, '')             |               hive中的空值转换               |
|  from_unixtime(cast(createTime as bigint))  |  时间戳：1566739467转为"2019-08-25 21:24:27"   |

```shell
# if条件判断常用于不同规格数据的清洗操作
hive> select ip,if(assign != '分配状态未知',0,assign) as fenpei  from asset ;
OK
10.0.0.1 分配状态未知

# case多条件判断
hive> select ip,
    case
        when assign = '已分配' then 1
        when assign = '未分配' then 2
        else 0
    end
    as fenpei
from asset

hive (ods)> select name,salary,
          > case when salary < 800 then 'low'
          > when salary >= 800 and salary <=5000 then 'middle'
          > when salary >5000 and salary <10000 then 'high'
          > else 'very high'
          > end as bracket
          > from emp1;


# parser_url()函数
hive> select parse_url('https://www.baidu.com/s?cl=3&tn=baidutop10&fr=top1000&wd=%E8%BF%AA%E5%A3%AB%E5%B0%BC%E6%94%B6%E8%B4%AD%E7%A6%8F%E5%85%8B%E6%96%AF&rsv_idx=2','HOST') ;
www.baidu.com

# 正则表达式
hive> select regexp_replace('foobar', 'oo|ar', '');
select regexp_replace('foobar', 'oo|ar', '-');
## 输出第一个回溯引用(.*?)匹配到的内容即the
select regexp_extract('foothebar', 'foo(.*?)(bar)', 1);
## 输出第而个回溯引用(bar)匹配到的内容即bar
select regexp_extract('foothebar', 'foo(.*?)(bar)', 2);
## 输出全部内容
select regexp_extract('foothebar', 'foo(.*?)(bar)', 0);


# 清洗组合
select if(4>5,5000,1000),coalesce(null,1,3,5),coalesce(null,null,null,null), case 3 when 1 then 'lala' when 2 then 'chye' else 'abc' end;
```

### 开窗函数

|      开窗函数       |                描述                 |
| ------------------- | ----------------------------------- |
|    rank() over()    |  排序相同时重复，总数不变。即1 1 3  |
| dense_rank() over() | 排序相同时重复，总数会减少。即1 1 2 |
| row_number() over() |      不重复，总数不变。即1 2 3      |

```sql
# 开窗函数的典型应用：取出每个用户时间最早的一条数据
# 与group by的区别是，group by只能取出分组的字段（即group by后面的字段）
select t.* from (select *, row_number() over(partition by uid order by time) rank from tbl) t where t.rank=1;
select uid, min(time) from tbl group by uid;
```

### 常用环境变量

|                       环境变量                        |                                含义                                |
| ----------------------------------------------------- | ------------------------------------------------------------------ |
|         set hive.exec.dynamic.partition=true          |                            开启动态分区                            |
|    set hive.exec.dynamic.partition.mode=nonstrict     |         设置动态分区模式为非严格，即允许不使用分区限制条件         |
|   set hive.exec.max.dynamic.partitions.pernode=1000   |                  设置每个执行MR的节点上最大分区数                  |
|      set hive.exec.max.dynamic.partitions =1000       |                    设置所有MR节点上最大总分区数                    |
| set SERDEPROPERTIES('serialization.null.format'='\N') | 设置hive控制存储方式为'\N'(此时存储在hdfs中为'\N'，查询显示为NULL) |

### HiveSQL转换成MapReduce任务
- join转换
```sql
select u.name, o.orderid from order o join user u on o.uid=u.uid;
```
在map的输出value中为不同表的数据打上tag标记，在reduce阶段根据tag判断数据来源。

- group by转换
```sql
select rank, isonline, count(*) from city group by rank, isonline;
```
将group by的字段组合为map的输出key值，利用MapReduce的排序，在reduce阶段保存LastKey区分不同的key。

- distinct转换
```sql
select dealid, count(distinct uid) num from order group by dealid;
```
当只用一个distinct字段时，如果不考虑map阶段的hash groupby，只需要将groupby字段和distinct字段组合为map输出key，利用MapReduce的排序，同时将groupby字段作为reduce的key，在reduce阶段保存LastKey即可完成去重。

### HiveSQL优化
##### hive表优化
- 分区
- 分桶
- 相同的数据尽量聚集在一起

##### hive job优化
- 并行化执行
每个查询被转化成多个阶段，有些阶段关联性不大，可以并行化执行，减少执行时间。
set hive.exec.parallel=true;
set hive.exec.parallel.thread.number=8;
- 本地化执行
job的输入数据大小必须小于hive.exec.mode.local.auto.inputbytes.max(默认128M)
job的map数必须小于hive.exec.mode.local.auto.tasks.max(默认4)
job的reduce数必须为0或1
set hive.exec.mode.local.auto=true;
- 合并输入小文件
set hive.input.format=org.apache.hadoop.hive.ql.io.CombineHiveInputFormat
合并文件数有mapred.max.split.size限制的大小决定
- 合并输出小文件
set hive.merge.smallfiles.avgsize=256000000;当输出文件平均小于该值，启动新job合并文件
set hive.merge.size.per.task=64000000;合并之后的文件大小
- JVM重用
set mapred.job.reuse.jvm.num.tasks=20;
JVM重用可以使得job长时间保留slot，直到作业结束，这对于有较多任务和较多小文件的任务非常有意义，减少执行时间。当然这个值不能设置过大，因为有些作业会有reduce任务，如果reduce任务没有完成，则map任务占用的slot不释放，其他的作用可能就需要等待。
- 压缩数据
查询**结果压缩输出**。
set hive.exec.compress.output=true;
set mapred.output.compreession.codec=org.apache.hadoop.io.compress.GzipCodec;
set mapred.output.compression.type=BLOCK;
**中间数据压缩**处理的是hive多个job之间的数据，对于中间压缩最好选择节省cpu耗时的压缩方式。
set hive.exec.compress.intermediate=true;
set hive.intermediate.compression.codec=org.apache.hadoop.io.compress.SnappyCodec;
set hive.intermediate.compression.type=BLOCK;

##### Hive Map优化
set mapred.map.tasks =10; 无效
(1)默认map个数
default_num=total_size/block_size;
(2)期望大小
goal_num=mapred.map.tasks;
(3)设置处理的文件大小
split_size=max(mapred.min.split.size,block_size);
split_num=total_size/split_size;
(4)计算的map个数
compute_map_num=min(split_num,max(default_num,goal_num))
经过以上的分析，在设置map个数的时候，可以简答的总结为以下几点：
增大mapred.min.split.size的值
如果想增加map个数，则设置mapred.map.tasks为一个较大的值
如果想减小map个数，则设置mapred.min.split.size为一个较大的值
情况1：输入文件size巨大，但不是小文件
情况2：输入文件数量巨大，且都是小文件，就是单个文件的size小于blockSize。这种情况通过增大mapred.min.split.size不可行，需要使用combineFileInputFormat将多个input path合并成一个InputSplit送给mapper处理，从而减少mapper的数量。
map端聚合
set hive.map.aggr=true;
推测执行
mapred.map.tasks.apeculative.execution

##### Hive Shuffle优化
Map端
io.sort.mb
io.sort.spill.percent
min.num.spill.for.combine
io.sort.factor
io.sort.record.percent
Reduce端
mapred.reduce.parallel.copies
mapred.reduce.copy.backoff
io.sort.factor
mapred.job.shuffle.input.buffer.percent
mapred.job.shuffle.input.buffer.percent
mapred.job.shuffle.input.buffer.percent
##### Hive Reduce优化
需要reduce操作的查询
group by,join,distribute by,cluster by...
order by比较特殊,只需要一个reduce
sum,count,distinct...
聚合函数
高级查询
推测执行
mapred.reduce.tasks.speculative.execution
hive.mapred.reduce.tasks.speculative.execution
Reduce优化
numRTasks = min[maxReducers,input.size/perReducer]
maxReducers=hive.exec.reducers.max
perReducer = hive.exec.reducers.bytes.per.reducer
hive.exec.reducers.max 默认 ：999
hive.exec.reducers.bytes.per.reducer 默认:1G
set mapred.reduce.tasks=10;直接设置

##### Hive查询操作优化
- join优化
1）select /*+mapjoin(A)*/ f.a,f.b from A t join B f on (f.a=t.a)
2）Bucket join
两个表以相同方式划分桶
两个表的桶个数是倍数关系
create table order(cid int,price float) clustered by(cid) into 32 buckets;
create table customer(id int,first string) clustered by(id) into 32 buckets;
select price from order t join customer s on t.cid=s.id
3）hive.optimize.skewjoin=true;如果是Join过程出现倾斜，应该设置为true
set hive.skewjoin.key=100000; 这个是join的键对应的记录条数超过这个值则会进行优化
4）join小表放在左边，大表放在右边；这点与关系型数据库不同。
5）join的表中如果有多行数据重复时，join会有m*n个重复结果，浪费时间。

- group by优化
hive.groupby.skewindata=true;如果是group by 过程出现倾斜 应该设置为true
set hive.groupby.mapaggr.checkinterval=100000;--这个是group的键对应的记录条数超过这个值则会进行优化

- count distinct优化
select count(distinct id) from tbl;
select count(1) from (select id from tbl group by id) t;

##### Hive查询结果导出到文件
- insert overwrite directory
```shell
##写入本地文件时加local，写入hdfs不加。导出时指定列分隔符'\t'，行分隔符'\n'
INSERT overwrite [local] directory '/user/hive/wangjj/result'
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
COLLECTION ITEMS TERMINATED BY '\n'
select * from tbl;
```

- hive -e "select * from tbl" > result.txt
```shell
##目标sql
SQL="select * from tbl;"
shopt -s expand_aliases
alias hive="beeline -u \"jdbc:hive2://10.217.109.56:2181,10.217.109.57:2181,10.217.109.58:2181/;serviceDiscoveryMode=zooKeeper;zooKeeperNamespace=hiveserver2\" --outputformat=csv2 "
##配置hive执行engine，打印表头，保存成以','为分隔符的csv文件
hive -e "set hive.execution.engine=spark;
        set hive.cli.print.header=true;
        set hive.resultset.use.unique.column.names=false;
        ${SQL}" | sed 's/x01/,/g' > ./result.csv
```

- DataFrame.write.csv()
```shell
##先把hive表转化为DataFrame，再使用DataFrame.write.csv()导出到hdfs
df = spark.sql("select * from tbl")
df.write.csv()
```

参考资料：
https://zhuanlan.zhihu.com/p/67566718
https://zhuanlan.zhihu.com/p/65436503
